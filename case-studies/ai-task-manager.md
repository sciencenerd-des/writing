# Case study — AI Task Manager

*Embedding a small, safe AI feature inside a real full-stack product: natural
language becomes structured task fields, a human reviews, and the feature degrades
gracefully to a deterministic parser when no API key is present.*

**Repository:** https://github.com/sciencenerd-des/task-manager
**Stack:** TypeScript · Next.js · Convex · WorkOS AuthKit · OpenAI · Vitest · GitHub Actions

---

## Abstract

Most discussion of AI features imagines greenfield apps. The more common reality
is adding a narrow AI capability to an existing product without compromising its
reliability. This project takes a working Kanban task manager — real-time backend,
authentication, drag-and-drop — and adds one feature: typing *"urgent: submit the
GST return by friday"* populates a task's title, priority, and due date for the
user to review. The interesting engineering is in the constraints around that
feature, not the feature itself.

## 1. Problem and motivation

A natural-language affordance is only an asset if it never makes the product feel
unreliable. Three risks had to be designed out: surprising the user by creating
something they did not confirm; breaking local development for contributors
without an API key; and shipping a date/priority parser that is subtly wrong.

## 2. System design

A Convex **action** receives the text and returns *suggested* fields — it never
writes to the database. When `OPENAI_API_KEY` is configured it calls the OpenAI
API; otherwise it falls back to a pure, dependency-free local parser that extracts
priority keywords and relative/weekday due dates. The UI populates the existing
create-task form with the suggestion, and the user confirms before anything is
saved.

## 3. Engineering decisions

- **Suggest, then confirm.** The AI proposes; the human commits. The action
  returning data rather than performing the write makes this structural.
- **Graceful degradation by default.** The deterministic fallback means the
  feature — and the whole local dev loop — works with no key, mirroring the
  zero-key philosophy from my other projects.
- **Put the testable logic in a pure function.** The fallback parser has no Convex
  or network dependencies, so it is unit-testable in isolation.

## 4. Working with the agentic tooling

The lesson here was about *boundaries in a serverless runtime*: an OpenAI call
belongs in a Convex action (which may use the network), not in a query or
mutation, and the call's output must be validated and constrained to the
application's enum types before it touches state. The model is treated as an
optional accelerator over a deterministic baseline, never as a dependency.

## 5. Verification and evaluation

The parser is covered by a Vitest suite (nine cases: priority detection, relative
dates, weekday resolution, combined inputs, and edge cases). Writing those tests
immediately surfaced a real **timezone bug** — formatting a local-midnight
timestamp with `toISOString()` shifted the day backwards in positive-offset
timezones such as IST. The fix was to format using local date parts. CI runs lint
and the unit tests on every push with no secrets.

## 6. Learnings

1. **The best place to add AI to a product is behind a human-review boundary.**
2. **A deterministic fallback is not a downgrade — it is what makes the feature
   safe to depend on** and trivial to test.
3. **Tests earn their keep at the boundaries**: the timezone bug would have been
   invisible in a manual demo and embarrassing in production.

## 7. Limitations and future work

The OpenAI path is verified by construction and unit-tested via its fallback, but
an end-to-end test against a live deployment requires provider and auth
credentials. A natural extension is multi-task parsing ("plan my week …") with the
same review-before-commit guarantee.
