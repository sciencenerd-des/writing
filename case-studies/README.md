# Engineering case studies

A set of structured retrospectives on projects I built or studied while learning
to engineer reliable applications on top of agentic AI tooling. Each follows the
same academic format — problem, design, engineering decisions, how I worked with
the AI tools, verification, learnings, and limitations — so the *method*, not just
the result, is visible.

The through-line across all of them is a single conviction:

> **Treat agentic AI as a powerful but unreliable component, and engineer
> determinism, verification, and developer experience around it.**

## Case studies

1. **[Bharat Doc Intelligence](bharat-doc-intelligence.md)** — a multimodal GST
   document agent where the model reads and deterministic code judges.
2. **[Financial Agents](financial-agents.md)** — a five-agent mesh with
   approval-gated, human-in-the-loop orchestration.
3. **[AI Task Manager](ai-task-manager.md)** — a small, safe AI feature embedded
   in a real full-stack product, with a zero-key fallback.
4. **[MiniMythos](minimythos.md)** — a research harness around a recurrent-depth
   reasoning architecture.
5. **[Studying primary sources](studying-agentic-primary-sources.md)** — what I
   learned by reading and extending two reference implementations
   (Karpathy's `llm-council` and `autoresearch`).

— [Biswajit Mondal](https://github.com/sciencenerd-des)
