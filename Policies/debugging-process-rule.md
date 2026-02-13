# Debugging Process Rule (Agent Policy)

Goal: prevent “guessing fixes” loops by enforcing the **Debugging / Problem-Resolution Guide**.

## Required behavior when debugging

When you are diagnosing or fixing a bug, flaky behavior, failing test, or unexpected runtime outcome:

- **You must follow** `Guides/Core/debugging-guide.md`.
- **Do not** propose a fix unless you can cite evidence for why it addresses the current best hypothesis.
- **Do not** ship “shotgun” changes (multiple speculative changes bundled together).

## Minimal loop (enforced)

For each iteration:

1. **State facts** (what is observed in code/output). Label unknowns explicitly.
2. **State the possibility space** (2–4 *logically valid* root-cause partitions that cover the space without “holes”).
3. **Pick a tactic** from the guide and justify why it’s the best next step.
4. **Design the smallest discriminating experiment** (what it will prove/disprove).
5. **Stop conditions**:
   - If you need output you cannot access, STOP and ask the human to run the command and paste results.
   - If multiple plausible behaviors depend on product intent, STOP and ask the human which is correct.
6. Apply **one** change, re-run the smallest verification, and update the possibility space.

## Evidence requirements

- Prefer evidence that eliminates possibilities:
  - targeted logging/instrumentation
  - partitioning / bisection
  - minimal working implementation + iterative convergence
  - isolating a failing test case
- When using logs, prefer a consistent prefix:
  - default `AIDEV:` unless the project specifies a different prefix

## Prohibited behaviors

- “It’s probably X” fixes without evidence.
- Changing multiple files/areas at once “just in case”.
- Rewriting architecture or refactoring broadly as a debugging tactic unless explicitly approved.

