# Study notes — learning from primary sources

*What I learned by reading and extending two reference implementations rather than
building from scratch: Andrej Karpathy's `llm-council` and `autoresearch`. These
are studies of existing open-source projects, not original work of mine — and that
distinction is the point of this note.*

**Studied:** [karpathy/llm-council](https://github.com/karpathy/llm-council) ·
[karpathy/autoresearch](https://github.com/karpathy/autoresearch)

---

## Abstract

A large part of becoming fluent with agentic AI is reading code written by people
who are further ahead, and extending it just enough to understand its decisions
from the inside. This note records what I took from two such reference
implementations and, more importantly, how those readings shaped the patterns I
later used in my own projects. I am explicit that these are forks I studied; the
intellectual credit for the designs is the original authors'.

## 1. Why study rather than build

Reimplementing a known pattern from a blank file teaches syntax; extending a
well-made reference teaches *judgment* — why a boundary was drawn here and not
there. Choosing two compact, opinionated projects let me see complete designs end
to end without the noise of a large codebase.

## 2. `llm-council` — ensembles and peer review as evaluation

`llm-council` sends one prompt to several models, has them review each other's
anonymised answers, and synthesises a final response. Studying it sharpened two
ideas for me:

- **Ensembling as a qualitative eval.** Comparing several models on the same
  prompt is a cheap, legible way to inspect differences in behaviour — a habit I
  carried into how I think about model selection.
- **Anonymised peer review reduces self-preference.** Removing authorship before
  review is a small structural choice with an outsized effect on fairness — the
  same instinct that later made me separate the *author* agent from the *verifier*
  agent in my own multi-agent work.

## 3. `autoresearch` — bounded autonomy with metric gates

`autoresearch` runs a loop: propose a change, run a short experiment, evaluate a
metric, keep or revert, and log the result. The lesson I internalised was the
*shape of safe autonomy*:

- **Autonomy is made safe by bounds and gates**, not by trusting the loop. Short
  runs, an explicit metric, and a keep/revert decision turn an open-ended agent
  into something controllable.
- **Logging every decision is what makes a loop auditable** after the fact.

These two principles — explicit metric gates and an auditable decision trail —
became defaults in my own projects' evaluation harnesses.

## 4. How these readings shaped my own work

- The **perception/judgment split** in Bharat Doc Intelligence echoes the
  "propose then check" instinct.
- The **author/verifier separation** in Financial Agents is a direct descendant of
  the anonymised-review idea.
- The **eval-gated CI** across my projects is the metric-gate principle from
  `autoresearch`, made cheap enough to run on every commit.

## 5. Learnings

1. **Read primary sources to acquire judgment, not just patterns.**
2. **Attribute honestly.** Studying and extending someone's reference is valuable
   precisely because you are clear about whose design it is.
3. **The best ideas transfer as principles** — separation of roles, bounded
   autonomy, metric gates — independent of the original code.

## 6. A note on attribution

These projects are forks of Andrej Karpathy's repositories, undertaken as learning
exercises. They are presented here as studies, not as original projects, and the
design credit belongs to the original author.
