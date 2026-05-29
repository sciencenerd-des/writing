# Model Reads, Code Judges: A Multimodal Agent You Can Actually Trust

*How I built a GST-invoice agent using the OpenAI Agents SDK where the model handles messy reality, but code keeps the final veto.*

---

Large language models are extraordinary at *reading* chaotic real-world
documents. They do not blink at a crumpled thermal receipt, a smartphone photo
taken at a bad angle, or a bill written partly in Hindi and partly in English.

But as good as they are at perception, they are not the right authority for basic
arithmetic or compliance judgment. You do not want a frontier model certifying
that `2 * 299 = 598`. It might get it right most of the time, but for a tool
deciding whether a business can legally claim a tax credit, "the model
hallucinated the math" is not an acceptable audit explanation.

When building [Bharat Doc Intelligence](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio),
I used one architectural boundary everywhere:

> **The model's job is to read unstructured reality into a strictly typed schema.
> Deterministic Python code's job is to judge whether that data is actually
> correct.**

That split gives you the best of both worlds: the flexibility of an LLM at the
messy input boundary and the reliability of unit-tested code at the decision
boundary.

## 1. Establish the ground rules with typed schemas

Before calling a model, define exactly what a successful extraction looks like.
The app uses Pydantic models to make the invoice structure explicit.

```python
from pydantic import BaseModel
from typing import Literal

class LineItem(BaseModel):
    description: str
    hsn_sac: str | None = None
    quantity: float = 1.0
    unit_price: float = 0.0
    amount: float = 0.0

class ExtractedInvoice(BaseModel):
    document_type: Literal["tax_invoice", "receipt", "bill_of_supply", "unknown"]
    supplier_gstin: str | None = None
    line_items: list[LineItem] = []
    subtotal: float | None = None
    total_tax: float | None = None
    grand_total: float | None = None
    detected_language: str | None = None
```

Passing this schema into the OpenAI Agents SDK as `output_type` forces the model
to return the application contract. It removes a whole class of brittle regexes,
ad hoc JSON parsing, and "hope the model formatted this correctly" code.

## 2. Let the agent inspect, but not certify

The extraction agent is allowed to use helper tools while reading the document,
but it does not own the final verdict.

```python
from agents import Agent, Runner, function_tool

@function_tool
def validate_gstin(gstin: str) -> str:
    ok, reason = _validate_gstin(gstin)
    return f"{'VALID' if ok else 'INVALID'}: {reason}"

agent = Agent(
    name="Bharat Doc Intelligence",
    instructions=INSTRUCTIONS,
    model="gpt-4o-mini",
    tools=[validate_gstin],
    output_type=ExtractedInvoice,
)
```

The distinction matters. `validate_gstin` is exposed as a tool so the model can
notice a GST number problem during extraction. But the tool returns a string into
the model context. It does not let the model overwrite the downstream audit
state.

The system prompt also tells the model to **transcribe, not fix**. If an invoice
physically prints the wrong line amount, the correct extraction is the wrong
printed value. Correcting the merchant's bad math would defeat the purpose of an
auditing tool. We need to preserve the flaw to catch the flaw.

## 3. Feed messy pixels through the Responses API

The Agents SDK can accept multimodal input items. A document read is just a user
message containing text plus a base64 image:

```python
content = [
    {"type": "input_text", "text": "Extract all fields from this document exactly as printed."},
    {"type": "input_image", "image_url": f"data:image/png;base64,{b64}", "detail": "high"},
]
result = await Runner.run(agent, [{"role": "user", "content": content}])
invoice = result.final_output_as(ExtractedInvoice)
```

At that moment, the AI's job is finished. It has turned messy input into a
structured Python object. Now deterministic code takes over.

## 4. Put judgment where code cannot lie

Every claim about mathematical correctness, GSTIN validity, or compliance risk
lives in ordinary Python functions. No model call. No prompt dependency. No
random variation.

```python
def validate_document(doc: ExtractedInvoice) -> list[Anomaly]:
    anomalies = []

    if doc.supplier_gstin:
        ok, reason = validate_gstin(doc.supplier_gstin)
        if not ok:
            anomalies.append(Anomaly(
                code="gstin_invalid",
                severity="error",
                message=reason,
                field="supplier_gstin",
            ))

    for i, item in enumerate(doc.line_items):
        expected = round(item.quantity * item.unit_price, 2)
        if item.amount and abs(expected - item.amount) > 1:
            anomalies.append(Anomaly(
                code="line_amount_mismatch",
                severity="warning",
                message="quantity * unit_price does not equal printed amount",
                field=f"line_items[{i}]",
            ))

    return anomalies
```

GSTIN validation is the cleanest example. A GSTIN contains a modulo-36 checksum.
An LLM can only guess whether a GSTIN looks plausible. Python can prove whether
the checksum is valid in under a millisecond.

So code does the proving.

## 5. Make local runs boring

If you want reviewers to trust a project, make it easy to run. This repo falls
back to deterministic mock fixtures when no `OPENAI_API_KEY` is present.

```python
def _extract(image_bytes, text, mime):
    if settings.use_mock:
        return mock_extract(text)

    from .agent import run_extraction
    return asyncio.run(run_extraction(image_bytes=image_bytes, text=text, mime=mime))
```

That small boundary makes the developer experience dramatically better. A
reviewer can clone the repo, run the UI, execute tests, and run evals without
configuring secrets or spending tokens. CI can do the same thing.

## Why this pattern generalizes

This perception-vs-judgment pattern applies to many enterprise AI features:

- **Contract analysis:** let the model extract clauses and dates; let code enforce
  policy rules and date windows.
- **System log triage:** let the model summarize noisy logs; let code decide
  whether thresholds page an engineer.
- **E-commerce cataloging:** let the model read a manufacturer spec sheet; let
  code validate dimensions, weights, and shipping constraints.

Whenever an LLM output drives an operational action, the action should be a
deterministic function of validated data.

The model gives you reach into messy, multilingual, unstructured reality. Code
gives you a verdict you can defend. Keep them in their lanes.

---

*Full source, evals, and mock mode:
[github.com/sciencenerd-des/openai-ai-deployment-portfolio](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio)*

