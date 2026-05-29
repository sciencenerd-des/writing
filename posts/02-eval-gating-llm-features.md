# Eval-gating an LLM feature: from "seems to work" to a CI check

*LLM demos rot silently. Here's how I turned a fuzzy "it usually catches the bad
invoices" into a hard pass/fail gate that runs in CI with no API key.*

---

The dirty secret of LLM features is that they degrade invisibly. You tweak a
prompt, bump a model version, refactor a parser — and nothing throws. The thing
still *runs*. It's just quietly worse, and you find out from a user.

Traditional tests don't catch this well, because the interesting behaviour is
statistical ("does it catch the anomalies?") rather than a single assertion. The
fix is to treat the behaviour as a **labelled evaluation** and gate on a metric.

Here's how [Bharat Doc Intelligence](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio)
does it for its GST audit layer.

## 1. Separate the two things you're evaluating

There are two distinct questions, and conflating them is a common mistake:

1. **Did the model extract the document correctly?** (perception — needs real
   documents, ground-truth fields, fuzzy scoring)
2. **Given correct extraction, did the logic flag the right problems?**
   (judgement — deterministic, cheap, exhaustively checkable)

The judgement layer is where bugs are most dangerous *and* most testable, so
that's what I gate hardest. It's pure Python, so its eval needs no model at all.

## 2. Make it a labelled multi-label task

Each case lists the anomaly *codes* a correct validator should raise:

```json
[
  { "name": "clean_intrastate_invoice", "expected_codes": [], "doc": { … } },
  { "name": "bad_gstin_checksum", "expected_codes": ["gstin_invalid"], "doc": { … } },
  { "name": "line_amount_mismatch",
    "expected_codes": ["line_amount_mismatch", "subtotal_mismatch"], "doc": { … } },
  …
]
```

Then score detection as a multi-label problem — precision, recall, F1 across
codes, plus exact-match per case:

```python
for case in cases:
    detected = {a.code for a in validate_document(ExtractedInvoice(**case["doc"]))}
    expected = set(case["expected_codes"])
    tp += len(detected & expected)
    fp += len(detected - expected)
    fn += len(expected - detected)
    exact += (detected == expected)
```

Exact-match is the strict one: it punishes *both* misses and false alarms. For a
compliance tool, a false alarm (flagging a clean invoice) erodes trust as much as
a miss, so I want it at 100% on the curated set.

## 3. Turn the metric into a gate

The harness exits non-zero if exact-match accuracy drops below a threshold — so
it behaves like a test:

```python
THRESHOLD = 1.0   # require perfection on the curated set
if exact_match < THRESHOLD:
    print(f"FAIL: exact-match {exact_match:.0%} < {THRESHOLD:.0%}")
    raise SystemExit(1)
```

## 4. Wire it into CI — with no secrets

Because the judgement layer is deterministic and the whole app has a mock mode,
CI runs the full suite and the evals with **no API key**:

```yaml
# .github/workflows/ci.yml
env:
  USE_MOCK: "1"
steps:
  - run: pip install -r requirements-dev.txt
  - run: pytest -q
  - run: python -m evals.run_evals    # exits non-zero if accuracy regresses
```

Now a prompt change that accidentally breaks the contract, or a refactor that
drops a check, fails the build instead of shipping.

## 5. What about evaluating the *model*?

The model-perception eval is the harder, more expensive half — it needs real
labelled scans and fuzzy field-level scoring, and it costs tokens to run. The
pattern there is the same shape (labelled set, metric, threshold) but you run it
on a schedule or pre-release rather than on every push, and you compare models
(`gpt-4o-mini` vs a frontier model) on the same set to make upgrade decisions
with data instead of vibes.

## The takeaway

- Split *perception* evals from *logic* evals; gate the logic hard and cheap.
- Express behaviour as a labelled set + a metric + a threshold.
- Exact-match punishes false positives, which matters for trust.
- Mock mode makes the gate free, so it can live on every push.

An LLM feature without an eval gate isn't done — it's just untested code that
happens to demo well today.

---

*Eval harness, dataset, and CI config:
[github.com/sciencenerd-des/openai-ai-deployment-portfolio/tree/main/evals](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio/tree/main/evals)*
