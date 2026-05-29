# Eval-Gating an LLM Feature: From "Seems to Work" to a Hard CI Check

*LLM demos rot silently. Here is how I turned a fuzzy "it usually catches the bad invoices" feeling into a strict pass/fail gate that runs in CI without an API key.*

---

The dirtiest open secret in AI engineering is that LLM features degrade
invisibly.

You change one adjective in a prompt, upgrade a model, or refactor a parser. The
endpoint still returns `200 OK`. Nothing crashes. But somewhere inside the
system, it may now be slightly worse at catching important anomalies.

That is how production AI systems fail: quietly, and then all at once.

To ship with confidence, you have to move beyond manual vibe-checks. Treat the
system's behavior as a labeled evaluation dataset, measure it, and make the
build fail when the metric slips.

Here is how I set that up for the audit layer of
[Bharat Doc Intelligence](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio).

## 1. Separate perception from judgment

A common mistake is testing an AI feature as one opaque blob. This mixes two
different problems:

1. **Perception:** did the model read the document correctly?
2. **Judgment:** given extracted data, did the business logic flag the right
   violations?

The perception layer is probabilistic. It needs real document images, model
calls, field-level scoring, and tolerance for small formatting differences.

The judgment layer is deterministic. Given the same extracted data, it should
always return the same anomaly codes. That makes it cheap and valuable to gate on
every commit.

For a compliance tool, business-logic bugs are both dangerous and easy to test.
So the repo gates them hard.

## 2. Build a labeled multi-label dataset

Each eval case contains a structured invoice payload and the exact anomaly codes
the validator should emit.

```json
[
  {
    "name": "clean_intrastate_invoice",
    "expected_codes": [],
    "doc": {
      "document_type": "tax_invoice",
      "supplier_gstin": "27AAPFU0939F1ZV"
    }
  },
  {
    "name": "bad_gstin_checksum",
    "expected_codes": ["gstin_invalid"],
    "doc": {
      "document_type": "tax_invoice",
      "supplier_gstin": "29AAAAA0000A1Z9"
    }
  }
]
```

The eval harness loads each case into the same Pydantic model used by the app,
runs the validator, and compares the detected codes to the label.

```python
for case in cases:
    doc = ExtractedInvoice.model_validate(case["doc"])
    detected = {a.code for a in validate_document(doc)}
    expected = set(case["expected_codes"])

    tp += len(detected & expected)
    fp += len(detected - expected)
    fn += len(expected - detected)
    exact += int(detected == expected)
```

The harness reports precision, recall, F1, and exact-match accuracy.

## 3. Make exact-match unforgiving

Exact-match is intentionally strict. It punishes both missed errors and false
alarms.

That matters in compliance software. A false negative lets bad data through. A
false positive makes users stop trusting the system. The evaluator should care
about both.

For the curated regression set, the threshold is set to perfection:

```python
THRESHOLD = 1.0

if result["exact_match"] < THRESHOLD:
    print("FAIL: exact-match dropped below threshold")
    raise SystemExit(1)
```

If one historical regression case breaks, CI fails.

## 4. Run the gate with no secrets

The project has a zero-key mock mode, so GitHub Actions can run the tests and
evals without OpenAI or GSTN credentials.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      USE_MOCK: "1"
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements-dev.txt
      - run: pytest -q
      - run: python -m evals.run_evals --write
```

This is the important practical detail: if an evaluation suite is expensive or
annoying to run, it will not be run often enough. The deterministic audit eval is
fast, free, and runs on every push.

## 5. Evaluate model perception separately

The model-reading layer still needs evaluation. It is just a different kind of
evaluation.

For perception evals, use the same general shape — labeled examples, metrics,
thresholds — but change the operating model:

- run them on a schedule or release-candidate branch;
- use fuzzy scoring for text fields;
- compare model versions against the same labeled image set;
- use the results to choose upgrades based on data rather than vibes.

Do not mix that noisy, token-consuming evaluation with the deterministic logic
gate that should run on every commit.

## The engineering takeaway

Moving past prototype demos means treating AI systems like software:

- isolate probabilistic perception from deterministic business logic;
- express expected behavior as labeled examples;
- choose a metric and make regressions fail loudly;
- keep the critical gate cheap enough to run in CI.

An AI feature without an automated evaluation gate is not finished. It is just a
demo that happens to work today.

---

*Eval harness, dataset, and CI config:
[github.com/sciencenerd-des/openai-ai-deployment-portfolio/tree/main/evals](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio/tree/main/evals)*

