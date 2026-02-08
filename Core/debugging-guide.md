# Debugging / Problem-Resolution Guide (Core)

Goal: prevent “guessing fixes” loops by using evidence, clear tactics, and explicit stopping conditions.

## Required mindset

- **Facts over speculation**: conclusions must be backed by observable evidence.
- **Hypotheses guide experiments**: use hypotheses to decide what to measure, not as conclusions.
- **One change per experiment**: avoid bundling multiple changes that confuse causality.

## Human-in-the-loop contract

In most projects the agent cannot run the app or observe runtime behavior directly.

- The **agent** proposes the smallest next experiment and explains what it will prove/disprove.
- The **human** runs the experiment and provides outputs (logs, test output, screenshots, traces, crash reports).

## Pick a tactic, justify it, reassess

For each iteration:
1. State the current best explanation space (2–4 possibilities).
2. Pick a tactic and justify why it’s the best next step.
3. Design the smallest discriminating experiment.
4. Run (human) → report evidence → update the possibility space.
5. If the tactic isn’t producing new discriminating evidence quickly, switch tactics.

## Debugging toolbox (core)

- **Facts-first logging/instrumentation**
  - Purpose: produce evidence that eliminates possibilities (not “log everything”).
  - Log at:
    - decision points (branches, retries, state changes)
    - function/module boundaries (inputs + outputs)
    - sequencing points (start/end markers, ordering)
    - identity/lifecycle points (object instance identity when relevant)
  - Log format:
    - use `AIDEV:` as the debug log prefix unless explicitly overridden
    - include key values needed to differentiate paths
  - Workflow:
    1. write a hypothesis
    2. add the smallest log(s) that will confirm/refute it
    3. have the human run and paste the output
    4. state *facts* from the output (no “probably/maybe”)

- **Partition the possibility space**
  - Example structure: A/B/C → B1/B2/B3.
  - Prefer partitions with a clear discriminating observation.

- **Minimal working implementation → move toward real until it breaks**
  - Start with a small implementation that works.
  - Change one dimension at a time toward the real implementation.
  - Identify the first change that introduces the issue.

- **Bisection**
  - Runtime bisection using feature flags/config toggles.
  - `git bisect` when the bug is a regression with a known commit range.

- **Minimal harness**
  - Create a deterministic reproducer (unit test, small harness, or minimal sample).
  - Goal: “repro on demand” so experiments are cheap.

- **Invariants & assertions**
  - Define what must be true; add temporary asserts/guards to fail at the first invalid state.

## Investigation loop (logging-heavy default)

1. **Observe the symptom**: actual vs expected.
2. **Form a hypothesis**: keep it tentative.
3. **Collect evidence**: often via the Facts-first logging tactic.
4. **Wait for evidence**: the human runs and provides output.
5. **State facts**: report what the evidence shows.
6. **Repeat** until the root cause statement is isolated.

## Log hygiene

- Remove or disable temporary logs once the investigation is complete.
- Keep logs focused on the current hypothesis.
- Avoid log spam: prefer logging state changes, not every read.

## Stop conditions (consult human)

Stop and consult the human when:
- Expected behavior is ambiguous (product intent required).
- Required evidence is missing (human hasn’t provided output needed to proceed).
- The next change is large/destructive/wide-reaching.
- You’ve iterated without gaining discriminating evidence.

## Adapter notes (minimal)

Adapters should only cover:
- Logging APIs and how to capture/share evidence for a stack
- Known tooling limitations that affect evidence collection

