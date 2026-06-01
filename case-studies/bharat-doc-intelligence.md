# Case study — Bharat Doc Intelligence

*A multimodal GST-document agent built on the OpenAI Agents SDK, designed so that
the model performs perception and deterministic code performs judgment.*

**Repository:** https://github.com/sciencenerd-des/openai-ai-deployment-portfolio
**Stack:** Python · FastAPI · OpenAI Agents SDK · Responses API (vision) · Pydantic · pytest · Docker · GitHub Actions

---

## Abstract

Bharat Doc Intelligence reads Indian GST invoices and receipts — including
regional-language scans — into a strictly typed schema, then applies a
deterministic audit to decide whether the document is internally consistent and
compliant. The project's contribution is not the extraction itself but the
*architecture around it*: a clean boundary between probabilistic perception and
deterministic judgment, packaged so that any developer can run the entire system,
its tests, and its evaluations without an API key.

## 1. Problem and motivation

Document understanding is an attractive use of multimodal models: real invoices
are photographed at angles, thermally printed, and multilingual. But the
operational decision a tax tool must make — *is this input tax credit claimable?*
— rests on arithmetic and statutory rules where a hallucinated answer is
unacceptable. The design problem was therefore to capture the model's perceptual
reach while removing it from the decision path entirely.

## 2. System design

The pipeline is three stages. **Perception:** a vision-capable model, invoked
through the Agents SDK with a Pydantic `output_type`, transcribes the document
into an `ExtractedInvoice`. **Judgment:** pure Python functions validate the
result — a GSTIN modulo-36 checksum, line-item arithmetic, CGST/SGST symmetry,
IGST exclusivity, and totals reconciliation — emitting structured anomalies.
**Reporting:** a confidence score and verdict (*validated* / *flagged*) are
derived deterministically from those anomalies. A later layer extends judgment to
GSTR-2B reconciliation, producing per-invoice IMS actions and ITC verdicts.

## 3. Engineering decisions

- **Schema before prompt.** Defining the target type first turned an open-ended
  prompting problem into a contract the SDK enforces, eliminating ad-hoc JSON
  parsing.
- **The model may inspect but not certify.** A `validate_gstin` function tool lets
  the agent check a number mid-read, but the authoritative check runs later, in
  code the model cannot influence.
- **Transcribe, not fix.** The system instruction forbids "correcting" printed
  numbers — preserving the flaw is what makes the flaw detectable.

## 4. Working with the agentic tooling

Mastering the Agents SDK meant learning where to *cede* control to the model and
where to *withhold* it. The SDK's `output_type`, function tools, and Responses-API
multimodal input items did the heavy lifting of structured, image-grounded
extraction; my job was to constrain them. The most useful mental model was to
treat the agent as a narrow, well-typed function with an unreliable
implementation — and to wrap it accordingly.

## 5. Verification and evaluation

Anomaly detection is scored as a labelled multi-label task (precision, recall, F1,
and exact-match) with a regression threshold that fails CI on any drop. Because
the judgment layer is deterministic and the system has a zero-key mock mode, the
full test suite and eval harness run offline on every push.

## 6. Learnings

1. **Reliability is an architecture, not a prompt.** The boundary between model
   and code is the product.
2. **Developer experience compounds trust.** "Clone and run in one command, no
   key" turned a demo into something reviewers actually executed.
3. **Domain modelling is leverage.** Encoding GST rules as code made the model's
   output falsifiable.

## 7. Limitations and future work

Extraction accuracy on genuinely adversarial scans is not yet measured against a
labelled image set; the perception eval is the natural next step. The GSTN/IRP
integrations are designed behind a mock-first interface but not yet wired to live
deployments.
