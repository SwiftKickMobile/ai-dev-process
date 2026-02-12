# Debugging Process Rule (Agent Policy)

Goal: prevent "guessing fixes" loops by enforcing the **Debugging / Problem-Resolution Guide**.

## Required behavior when debugging

When you are diagnosing or fixing a bug, flaky behavior, failing test, or unexpected runtime outcome:

- **You must follow** the investigation loop and techniques in `@debugging-guide.md`.
- **Do not** propose a fix unless you can cite evidence for why it addresses the current best hypothesis.
- **State your technique selection** up front or take direction from the human before starting.

## Prohibited behaviors

- Speculative fixes without evidence ("it's probably X").
- Bundling multiple unrelated changes where the outcome would be ambiguous.
- Rewriting architecture or refactoring broadly as a debugging tactic unless explicitly approved.
- Deleting or modifying production code while debugging. Comment out instead and make a copy when testing modifications. Build separate debug views/harnesses rather than altering the code under investigation. This keeps the original code in place as a reference, preventing important details from being forgotten or lost during the investigation.

## Stop conditions (consult human)

Stop and consult the human when:
- Expected behavior is ambiguous (product intent required).
- Required evidence is missing (human hasn't provided output needed to proceed).
- The next change is large, destructive, or wide-reaching.
- You've iterated without gaining discriminating evidence.
