# Model reads, code judges: a multimodal agent you can actually trust

*How I built a GST-invoice agent on the OpenAI Agents SDK where the model never gets to decide whether the maths is right.*

---

Large language models are extraordinary at *reading* messy real-world documents —
a crumpled thermal-printed invoice, a photo taken at an angle, a bill half in
Hindi. They are also, stubbornly, not something you want certifying that
`2 × 299 = 598`. For a compliance tool that decides whether a business can claim
a tax credit, "the model said the totals add up" is not an acceptable answer.

So the design principle for [Bharat Doc Intelligence](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio)
is one line:

> **The model reads the document into a typed schema. Deterministic Python judges
> whether it's correct.**

This post walks through how that split works in practice, and why it's the right
shape for most "AI + structured data" features — not just tax documents.

## 1. Describe the answer before you call the model

The first thing is a Pydantic model of *what a read document is*. The Agents SDK
will hold the model to this shape via `output_type`, which removes an entire class
of brittle JSON-parsing glue.

```python
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
    # …
```

## 2. The agent reads — and can self-check, but can't self-certify

```python
from agents import Agent, Runner, function_tool

@function_tool
def validate_gstin(gstin: str) -> str:
    ok, reason = _validate_gstin(gstin)         # pure-Python checksum
    return f"{'VALID' if ok else 'INVALID'}: {reason}"

agent = Agent(
    name="Bharat Doc Intelligence",
    instructions=INSTRUCTIONS,
    model="gpt-4o-mini",
    tools=[validate_gstin],
    output_type=ExtractedInvoice,
)
```

Two deliberate choices:

- `output_type=ExtractedInvoice` constrains the model to the schema.
- `validate_gstin` is a **tool**, so the model can check a GST number mid-read —
  but notice it returns a *string for the model to read*, not a verdict the model
  gets to overwrite. The authoritative check happens later, in code.

The instructions tell the model to **transcribe, not fix**: "Write numbers exactly
as printed; do not 'correct' arithmetic." You want the model to faithfully report
that the invoice says `698` even when `2 × 299 = 598` — because catching that
discrepancy is the whole point.

## 3. Pass the image through the Responses API

The Agents SDK accepts Responses-API input items, so a multimodal read is just a
`user` message mixing text and an image:

```python
content = [
    {"type": "input_text", "text": "Extract all fields from this document."},
    {"type": "input_image", "image_url": f"data:image/png;base64,{b64}", "detail": "high"},
]
result = await Runner.run(agent, [{"role": "user", "content": content}])
invoice = result.final_output_as(ExtractedInvoice)
```

## 4. Now code judges — and it's the part that can't lie

Everything that constitutes a *claim about correctness* lives in pure functions
with no model in the loop. They're fast, free, and exhaustively unit-tested.

```python
def validate_document(doc: ExtractedInvoice) -> list[Anomaly]:
    anomalies = []
    # GSTIN checksum (the GSTN modulo-36 algorithm) — a check the model can't fake
    if doc.supplier_gstin:
        ok, reason = validate_gstin(doc.supplier_gstin)
        if not ok:
            anomalies.append(Anomaly("gstin_invalid", "error", reason, "supplier_gstin"))
    # line-item arithmetic
    for i, it in enumerate(doc.line_items):
        if it.amount and abs(it.quantity * it.unit_price - it.amount) > 1:
            anomalies.append(Anomaly("line_amount_mismatch", "warning", ..., f"line_items[{i}]"))
    # subtotal + tax == grand total, CGST == SGST, IGST not mixed with CGST/SGST …
    return anomalies
```

The GSTIN checksum is the clearest example of why this split matters: it's a
deterministic modulo-36 calculation. A model can *guess* whether a GSTIN looks
valid; code can *prove* it. So code does.

## 5. Make it runnable with no key

The single highest-leverage thing for a demo: wrap the model call so the whole
pipeline falls back to deterministic fixtures when there's no `OPENAI_API_KEY`.

```python
def _extract(image_bytes, text, mime):
    if settings.use_mock:
        return mock_extract(text)
    from .agent import run_extraction        # lazy import: SDK only needed live
    return asyncio.run(run_extraction(image_bytes=image_bytes, text=text, mime=mime))
```

Now a reviewer clones the repo and runs the entire app, UI, tests, and evals in
one command, for free. CI does too.

## Why this generalises

This pattern — **perception by the model, judgement by code** — applies far beyond
invoices:

- Extracting fields from contracts? Let code enforce that dates are ordered and
  amounts reconcile.
- Parsing logs into incidents? Let code validate severity thresholds.
- Any time an LLM produces structured data that downstream systems *act on*, the
  acting decision should be a deterministic function of that data, not a second
  opinion from the model.

The model gives you reach into messy, unstructured, multilingual reality. Code
gives you a verdict you can defend. Keep them in their lanes.

---

*Full source, evals, and a one-command mock mode:
[github.com/sciencenerd-des/openai-ai-deployment-portfolio](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio)*
