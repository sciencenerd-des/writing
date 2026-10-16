# Use Codex to investigate why a failed command looks green

An AI coding workflow depends on its feedback. If a failing command reports
success, an agent can make a reasonable next decision from incorrect evidence.
Before asking for a better prompt or a larger model, inspect the command that
decides whether work passed.

Forge, my coding-agent reliability harness, contains a small example you can
reproduce without a model, database, Docker, or API key. The exercise uses its
actual auditor helper rather than an imitation of the mechanism.

## Reproduce the false pass

```bash
git clone https://github.com/sciencenerd-des/forge.git
cd forge
git checkout 1ec40d8ffbff0ffe89654bf33d3b0c26a558a052
python3 examples/verification_demo.py
```

The checkout pins the version used for this walkthrough. The demo requires
Python 3.11+, Bash, and `tail`. Inside a temporary directory, it runs a fixed
command whose producer prints a message and exits with an error:

```bash
(echo BROKEN; exit 1) | tail -1
```

By default, Bash uses the exit status of the final pipeline stage. Here, `tail`
succeeds. A caller checking only that pipeline's exit status sees zero, even
though the producer failed. This problem is easy to miss when output formatting
and correctness checks are combined in the same command.

The demo calls `harden_pipefail` from `engine/src/auditor.py` and reruns the
command with `set -o pipefail`. The failure now reaches the caller. It also
runs a successful producer, which must still pass, and applies hardening twice,
which must leave the already hardened command unchanged.

## Use Codex as a review partner

Open the checkout in Codex and try:

> Read AGENTS.md, examples/verification_demo.py, and harden_pipefail in
> engine/src/auditor.py. Trace the real helper called by the demonstration.
> Explain the two different exit statuses. Find an example where pipefail
> alone would not protect the verification result. Do not change the checks.

The useful output is a testable explanation. Run the command yourself, compare
the result, and inspect the source before accepting the explanation.

One counterexample is a broad fallback:

```bash
failing_test_runner || successful_fallback
```

If the fallback succeeds, the overall command can still report success.
`pipefail` governs pipelines; it does not make this fallback appropriate.
Choose a runner based on availability separately from evaluating its result.
Forge's existing `test_pipefail_hardening.py` includes a regression where a real
pytest failure must not be hidden by a different runner finding zero tests.

## What the evidence supports

The broken producer is a negative control. The working producer is a positive
control. Together, they show that this helper preserves this particular failure
while accepting a genuine success. They do not prove that a generated program
meets its full requirements or that every shell command is safe.

The exercise also does not test model inference, sandbox isolation, database
durability, or restart recovery. Those require their own evidence. A useful
tutorial states this boundary so readers can reuse the lesson without
overestimating what was verified.

For a deeper exercise, read the
[Forge walkthrough](https://github.com/sciencenerd-des/forge/blob/1ec40d8ffbff0ffe89654bf33d3b0c26a558a052/docs/CODEX_WALKTHROUGH.md)
and [regression source](https://github.com/sciencenerd-des/forge/blob/main/tests/regression/test_pipefail_hardening.py).
Keep the commit SHA with any reproduction or recording. This is a small place
to practice a larger habit: ask Codex for an explanation, then give that
explanation an independent way to fail.
