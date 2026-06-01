# Case study — Financial Agents

*An event-driven, five-agent system used to study safe multi-agent orchestration:
agents propose, an independent agent verifies, orchestration gates, and a human
approves.*

**Repository:** https://github.com/sciencenerd-des/Financial_Agents
**Stack:** Python · FastAPI · Docker Compose · typed contracts · pytest

---

## Abstract

Financial Agents is a deliberately structured multi-agent system whose purpose is
to explore *control*, not autonomy. Five specialised services — perception,
drafting, verification, orchestration, and translation — communicate through a
unified gateway under an approval-gated workflow. The finance domain is used as a
demanding testbed: a setting where an unchecked autonomous action is expensive,
which forces the architecture to make verification and human approval
first-class.

## 1. Problem and motivation

Naive multi-agent systems chain a model's output directly into another model's
input, compounding error and removing any point at which a human or a rule can
intervene. The question this project investigates is structural: *what does an
agent system look like when proposal and verification are separated, and when no
consequential action occurs without an explicit gate?*

## 2. System design

The mesh is event-driven and role-separated:

- **Perception** turns raw signals into structured observations.
- **Drafting** proposes an action or plan from those observations.
- **Verification** independently audits the draft against rules and risk
  constraints — it does not author, it checks.
- **Orchestration** sequences the workflow and enforces approval gates.
- **Translation** renders outcomes into stakeholder-readable narrative.

A FastAPI gateway exposes the workflow behind typed contracts, and the whole stack
is reproducible via Docker Compose.

## 3. Engineering decisions

- **Separate the author from the auditor.** Verification is a distinct agent with
  a distinct objective, so it has no incentive to rationalise a bad draft.
- **Typed contracts between agents.** Each hand-off is a validated schema, which
  localises failures and makes the system testable.
- **Approval as a gate, not a suggestion.** Orchestration blocks consequential
  steps until a human (or a rule) approves.

## 4. Working with the agentic tooling

The central skill this project built was *orchestration discipline*: resisting the
temptation to let agents free-run, and instead modelling the workflow as a state
machine with explicit transitions. The agents are powerful at their narrow tasks;
the engineering is in the contracts, the gates, and the sequencing between them.

## 5. Verification and evaluation

The system is covered by contract tests (schema conformance at each boundary), API
tests (gateway behaviour), and workflow tests (end-to-end sequencing including the
approval gate), alongside security- and risk-oriented checks. A mock mode keeps
these runnable without live providers.

## 6. Learnings

1. **Separation of proposal and verification is the single highest-value pattern**
   in agent design — it is where safety and reliability come from.
2. **Typed contracts turn a fragile chain into a testable graph.**
3. **Human-in-the-loop is an architectural commitment**, expressed as a gate, not
   a disclaimer.

## 7. Limitations and future work

The verification agent's audits are rule-driven and would benefit from a labelled
evaluation of its catch-rate. The approval gate is currently coarse; a graduated,
risk-scored gate (auto-approve low risk, escalate high risk) is the natural
extension.
