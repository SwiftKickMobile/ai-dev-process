Managed-By: ai-dev-process
Managed-Id: guide.debugging-core
Managed-Source: Core/debugging-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-08

# Debugging / Problem-Resolution Guide (Core)

Goal: prevent "guessing fixes" loops by using evidence, clear tactics, and explicit stopping conditions.

## Strategy: fact-based analysis

All debugging follows one overarching strategy: **conclusions must be backed by observable evidence.**

- **No speculation**: never treat a hypothesis as a conclusion. Hypotheses guide what to measure; evidence determines what is true.
- **Unambiguous experiments**: design each experiment so the result clearly narrows the possibility space. Avoid combining unrelated changes where the outcome would be ambiguous.
- **State facts explicitly**: after gathering evidence, report what it shows — no "probably" or "maybe".
- **Preserve production code**: never delete or modify production code while debugging. Comment out instead and make a copy when testing modifications. Build separate debug views/harnesses rather than altering the code under investigation. This keeps the original code in place as a reference, preventing important details from being forgotten or lost during the investigation.

### `AIDEV:` tagging convention

Use `AIDEV:` as a unified prefix for all temporary debug artifacts — logs, commented-out code, and debug modifications. This makes cleanup a single search. The human can override with a custom prefix when needed.

Tag commented-out code and modifications with a brief explanation:

```
// AIDEV: original implementation
// let result = computeValue(input)
// AIDEV: test with hardcoded value to isolate calculation
let result = 42
```

Tag debug logs the same way:

```
print("AIDEV: contentOffset=\(offset) delta=\(delta)")
```

When debugging is complete, search for `AIDEV:` to find and remove all temporary artifacts.

## Human-in-the-loop contract

In most projects the agent cannot run the app or observe runtime behavior directly.

- The **agent** proposes the smallest next experiment and explains what it will prove/disprove.
- The **human** runs the experiment and provides outputs (logs, test output, screenshots, traces, crash reports).

## Investigation loop

1. **Observe the symptom**: actual vs expected.
2. **Form a hypothesis**: keep it tentative.
3. **Select techniques**: state which technique(s) from the toolbox you plan to use and why, or take direction from the human. Techniques may be combined as needed (e.g., logging + code bisection).
4. **Design the smallest discriminating experiment**: the experiment should eliminate roughly half the possibility space or confirm/refute the hypothesis.
5. **Gather evidence**: the human runs the experiment and provides output.
6. **State facts**: report what the evidence shows.
7. **Update the possibility space** and repeat until the root cause is isolated.
8. If a technique isn't producing discriminating evidence quickly, reassess and switch.

## Techniques

Techniques are fact-gathering methods that serve the strategy. Select based on the situation and combine as needed.

### Logging / instrumentation

- Purpose: produce evidence that eliminates possibilities (not "log everything").
- Log at:
  - decision points (branches, retries, state changes)
  - function/module boundaries (inputs + outputs)
  - sequencing points (start/end markers, ordering)
  - identity/lifecycle points (object instance identity when relevant)
- Use the `AIDEV:` prefix (see tagging convention above). Include key values needed to differentiate paths.
- Keep logs focused on the current hypothesis. Prefer logging state changes, not every read.

### Partition the possibility space

- Enumerate 2–4 possible explanations; narrow with targeted experiments.
- Example structure: A/B/C → B1/B2/B3.
- Prefer partitions with a clear discriminating observation.

### Code bisection

- Binary search across code differences to isolate the root cause in logarithmic steps.
- At each step, add or remove approximately half of the remaining differences. The result tells you which half contains the bug; discard the other half and repeat.
- Two starting directions (choose whichever is more practical):
  - **Minimal → production**: start with a minimal working version and progressively add complexity from the broken code, jumping roughly halfway at each step.
  - **Production → minimal**: start with the broken production code and strip away roughly half the suspected differences at each step.
- Prefer building separate debug views/harnesses over modifying production code.

### Git bisect

- `git bisect` when the bug is a regression with a known commit range.

### Minimal harness

- Create a deterministic reproducer (unit test, small harness, or minimal sample).
- Goal: "repro on demand" so experiments are cheap.

### Invariants & assertions

- Define what must be true; add temporary asserts/guards to fail at the first invalid state.

## Stop conditions (consult human)

Stop and consult the human when:
- Expected behavior is ambiguous (product intent required).
- Required evidence is missing (human hasn't provided output needed to proceed).
- The next change is large/destructive/wide-reaching.
- You've iterated without gaining discriminating evidence.

## Adapter notes (minimal)

Adapters should only cover:
- Logging APIs and how to capture/share evidence for a stack
- Known tooling limitations that affect evidence collection
