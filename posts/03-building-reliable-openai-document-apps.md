# Building Reliable OpenAI Document Apps with Validation Layers

LLMs are useful for perception-heavy work such as document understanding, but they should not be the only source of business correctness.

## Pattern

Use the model for extraction. Use deterministic code for validation.

## Example: GST invoices

A document workflow can split responsibilities:

- OpenAI vision extracts vendor details, line items, tax fields, and totals.
- Pydantic schemas make the response structured.
- GSTIN validators check identifiers.
- Tax rules check CGST/SGST symmetry and invoice math.
- The API returns audit flags instead of hiding uncertainty.

## Developer experience lessons

- mock mode lets contributors run the app without secrets
- evals prevent regressions when prompts or models change
- structured outputs are easier to test and debug
- troubleshooting docs reduce integration support burden

## Related project

Bharat Doc Intelligence: https://github.com/sciencenerd-des/openai-ai-deployment-portfolio
