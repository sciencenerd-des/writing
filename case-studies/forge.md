# Forge: making verification evidence visible

**Repository:** https://github.com/sciencenerd-des/forge
**Audience:** practical AI builders learning to inspect coding-agent behavior.

## Problem

A coding agent can receive misleading success signals when a shell pipeline
hides a failed command, a fallback masks a real test failure, or acceptance
checks are too weak. These are properties of the harness around the model.

## Design visible in the source

Forge separates planning, contract creation, execution, and evaluation. It
includes durable state, sandbox interfaces, review gates, and a regression
suite. The useful teaching artifact is the connection from a concrete failure
to its guard and regression, not an assertion that the architecture solves
every reliability problem.

## A reproducible example

`examples/verification_demo.py` exercises the production auditor's pipeline
hardening with a failing producer, a working producer, and an idempotence check.
Run it from the checkout with `python3 examples/verification_demo.py` once the
walkthrough branch is merged, or check out that branch first. It needs no model
or service. The linked tutorial asks Codex to explain the evidence and identify
a failure that this mechanism cannot fix.

## Role of Codex

Forge is my primary example of Codex-assisted development. The tutorial gives a concrete
workflow for using Codex to inspect the code and reason about negative controls.
That statement is not a line-by-line authorship attribution or a claim that
Codex is the inference backend for every Forge run. Historical prompts and
outcomes should be attached only where retained and shareable.

## Verification and limits

The offline demonstration covers pipeline exit-status preservation. The
regression suite covers additional named cases; its result must be quoted with
the tested revision, environment, and skips. Docker isolation, durable Postgres
recovery, and live-model task completion require separate integration runs.
No adoption, success-rate, or benchmark claim is made here.

## Lesson for builders

Make the failure observable, define an independent check, reproduce a negative
control, implement a narrow repair, and preserve the regression. Use Codex to
speed up investigation while keeping the pass/fail evidence outside a model's
self-report.
