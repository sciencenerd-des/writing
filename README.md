# Writing

Technical notes on building reliable LLM applications — structured extraction,
multimodal agents, and eval-driven development. Code-first, no hype.

— [Biswajit Mondal](https://github.com/sciencenerd-des)

## Posts

1. **[Model reads, code judges: a multimodal agent you can actually trust](posts/01-model-reads-code-judges.md)**
   How to split an AI document feature so the model handles perception and
   deterministic code makes every correctness call. OpenAI Agents SDK · Responses
   API (vision) · Pydantic.

2. **[Eval-gating an LLM feature: from "seems to work" to a CI check](posts/02-eval-gating-llm-features.md)**
   Turning fuzzy "it usually works" into a labelled eval with a pass/fail
   threshold that runs in CI with no API key.

3. **[Building Reliable OpenAI Document Apps with Validation Layers](posts/03-building-reliable-openai-document-apps.md)**
   A practical pattern for using OpenAI for extraction while deterministic code handles business correctness.

4. **[Mock-first AI Development](posts/04-mock-first-ai-development.md)**
   Why AI apps should run locally and in CI without API keys.

The projects these draw from:
[Bharat Doc Intelligence](https://github.com/sciencenerd-des/openai-ai-deployment-portfolio)
· [Financial Agents](https://github.com/sciencenerd-des/Financial_Agents).
