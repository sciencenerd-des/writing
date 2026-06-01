# Case study — MiniMythos

*A local-first research harness built around a recurrent-depth reasoning
architecture, undertaken to understand how to engineer reproducible
infrastructure around an experimental model.*

**Repository:** https://github.com/sciencenerd-des/minimythos
**Stack:** Python · PyTorch · config-driven experiment tooling

---

## Abstract

MiniMythos is an experimental platform that wraps a recurrent-depth transformer
(a Prelude → looped recurrent block → Coda architecture) as the backbone for an
iterative coding agent, a world model, and an evaluator. It is the most
research-oriented of my projects, and its value to me was less about the model and
more about the *scaffolding*: building configurable, reproducible experiment
infrastructure around a moving target.

## 1. Problem and motivation

Recurrent-depth reasoning — spending variable compute by looping a latent block —
is a compelling idea, but experimenting with it productively requires
infrastructure: configurable environments, an evaluation loop, and a disciplined
way to vary one thing at a time. The project was an exercise in building that
scaffolding well enough that the research question, not the plumbing, became the
bottleneck.

## 2. System design

The system separates concerns into a trainer, an evaluator, a dashboard, and a CLI,
with model and environment configurations expressed as declarative files. The
coding-agent loop follows an inspect → patch → test → diagnose → repair cycle; a
world-model component predicts next states for planning; and the evaluator scores
behaviour across configured environments.

## 3. Engineering decisions

- **Configuration over code.** Environments and model sizes are declarative, so
  experiments are reproducible and comparable rather than ad hoc.
- **Separate the loop from the model.** The agent/evaluator scaffolding is
  independent of the specific backbone, so the architecture can change without
  rewriting the harness.
- **Bounded experiments.** Short, metric-gated runs over open-ended training.

## 4. Working with the tooling

This project was where I internalised that *research velocity is an engineering
property*. The discipline of declarative configs, a clean trainer/evaluator split,
and a CLI that makes runs repeatable is what lets you trust a comparison between
two ideas. AI-assisted coding tools were most useful here for scaffolding the
repetitive infrastructure quickly so attention could stay on the experiment design.

## 5. Verification and evaluation

Evaluation is built into the harness as a first-class component, with
configuration for interactive-generalisation settings and per-environment scoring.
The emphasis is on reproducibility — same config, same result — as the precondition
for any meaningful comparison.

## 6. Learnings

1. **Infrastructure determines research throughput.** Good scaffolding turns ideas
   into experiments cheaply.
2. **Reproducibility is the unit of trust** in any evaluation.
3. **Decoupling the harness from the model** is what lets the interesting part
   change freely.

## 7. Limitations and future work

MiniMythos is exploratory; its results are not benchmarked against established
baselines, and the recurrent-depth backbone is a research artifact rather than a
production model. The harness, however, generalises — its evaluator-first,
config-driven structure is the part worth carrying forward.
