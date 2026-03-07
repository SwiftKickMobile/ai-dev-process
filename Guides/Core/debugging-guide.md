Managed-By: skai
Managed-Id: guide.debugging-core
Managed-Source: Guides/Core/debugging-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Debugging / Problem-Resolution Guide (Core)

Goal: prevent "guessing fixes" loops by using evidence, clear tactics, and explicit stopping conditions.

## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line. The only normal exception is full workflow completion, which uses `🏁 Complete. Let me know if anything needs adjustment.`

Use these standard gate lines:
- Planned gate: `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- Blocked gate: `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

Planned gates are the expected review points of this workflow. At each planned gate:
1. Summarize what you did and what should happen next.
2. End with the planned gate line.
3. STOP and wait for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent. If the gate is non-standard, make it describe the exact human response or handoff needed to resume the workflow.

If an unexpected blocker prevents continued work, use the blocked gate line and STOP until the human resolves it.

Workflow-specific gate notes:
- The pre-experiment gate is often a non-standard planned gate. When the human must run the experiment, the normal response is to provide the experiment output or ask for changes to the experiment. `Next` there only means "proceed with this proposed experiment" when the agent can run the experiment itself.
- The root-cause, fix, and verify/close gates are hard planned gates. `auto` does not bypass them.
- The pre-experiment and post-experiment gates are the normal iteration gates. `auto` may bypass them when the workflow is still gaining discriminating evidence.

Planned gates for this workflow:
- Before running an experiment (pre-experiment): after stating the current facts, possibility-space partition, chosen tactic, and proposed discriminating experiment.
- After an experiment (post-experiment): after stating the evidence, conclusions, updated possibility space, and recommended next step.
- Before declaring a root cause (root cause, hard gate).
- Before applying a fix (fix, hard gate).
- Before concluding (verify/close, hard gate).

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the workflow's active-phase rules below. Do not use them to skip hard approval gates or clear unrelated 🟡 markers.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- This workflow currently uses inline discussion rather than a required debugging work document.
- The active debugging state lives in the conversation itself: current facts, possibility space, selected tactic, proposed experiment, and latest evidence.
- At the pre-experiment gate, STOP after proposing the smallest discriminating experiment and wait for the human to run it or redirect it.
- At the pre-experiment gate, the normal resume signal is experiment evidence from the human, not advance intent.
- At the post-experiment gate, STOP after interpreting the evidence and proposing the next step.
- At the root-cause, fix, and verify/close gates, STOP and wait for explicit human approval before advancing.

Workflow-specific advance behavior:
- `auto` may bypass the post-experiment iteration gate while the workflow is still in evidence-gathering mode.
- `auto` does not bypass a pre-experiment gate when the human must run the experiment and provide evidence.
- `auto` does not bypass the root-cause, fix, or verify/close hard gates.
- Use `root cause`, `fix`, and `verify` as the stable bounded-auto targets for this guide.
- At the pre-experiment gate, advance intent is only relevant when the agent can run the proposed experiment itself. When the human must run it, the workflow resumes on evidence rather than on `next`.

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
1. State the current best explanation space (2-4 possibilities).
2. Pick a tactic and justify why it's the best next step.
3. Design the smallest discriminating experiment.
4. Run (human) → report evidence → update the possibility space.
5. If the tactic isn't producing new discriminating evidence quickly, switch tactics.

Gate: before each experiment, state the current possibility space, tactic choice, and proposed discriminating experiment, then STOP at the pre-experiment gate. When the human is the one running the experiment, end with a gate line that asks them to run it and share the output, or tell you what to change first.

## Debugging toolbox (core)

- **Facts-first logging/instrumentation**
  - Purpose: produce evidence that eliminates possibilities (not "log everything").
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
    4. state *facts* from the output (no "probably/maybe")

- **Partition the possibility space**
  - Example structure: A/B/C → B1/B2/B3.
  - Prefer partitions with a clear discriminating observation.
  - Use *logically valid* partitions:
    - cover the space well enough that the true cause must be in one bucket (no "holes")
    - allow some overlap in practice (both can be true), but still be useful for elimination
    - if overlap is likely, add an explicit "both / interaction" or "other / unknown" bucket
  - Examples:
    - **Location**: server-side vs client-side vs integration/contract mismatch
    - **Call boundary**: bug is in this function vs in a function it calls (or in the contract between them)
    - **Timing**: ordering/race vs stale state/caching vs deterministic logic bug
    - **Data**: bad input vs transformation bug vs persistence/serialization bug

Partition quality bar:
- A possibility-space partition must list 2-4 plausible causes. "X vs not X" is not acceptable unless the "not X" bucket is further partitioned into concrete alternatives.
- Partitions must be phrased so evidence can actually eliminate buckets (no hand-wavy buckets).

- **Minimal working implementation → move toward real until it breaks**
  - Start with a small implementation that works.
  - Change one dimension at a time toward the real implementation.
  - Identify the first change that introduces the issue.

- **Bisection**
  - Use when you can define an approximate notion of "distance" and systematically move in ~half-steps rather than making tiny one-at-a-time changes.
  - Preferred: a deterministic reproducer (unit test / small harness) so each iteration is cheap.
  - When using a git-based bisection workflow (temporary branch), read `Guides/Core/debugging-tactic-code-bisection.md` before starting.

- **Minimal harness**
  - Create a deterministic reproducer (unit test, small harness, or minimal sample) so experiments are cheap.
  - Recipe (high-level):
    1. Pick the smallest surface area that still exhibits the symptom.
    2. Freeze inputs and eliminate external dependencies.
    3. Make the run fast and deterministic.
    4. Use it as the experiment runner for subsequent tactics (logging, bisection, invariants).

- **Prior-art research**
  - Use only after the possibility space is narrowed to a specific library/framework/component or a clear error signature.
  - Output contract: summarize 2-3 candidate explanations and map each to a discriminating experiment in this codebase (do not jump to fixes).

- **Invariants & assertions**
  - Define what must be true; add temporary asserts/guards to fail at the first invalid state.

## Testing a hypothesis is not applying a fix

Testing a hypothesis means running a discriminating experiment (logging, bisection, unit-test repro, minimal harness, etc.).

It does NOT mean "implement the fix and see if it helps."

Inspection-only exception (narrow):
- Only treat something as "obvious by inspection" when you can point to a specific, local code fact that directly implies incorrect behavior (e.g., typo, wrong variable used, off-by-one, missing guard).
- If the proposed change is non-local, architectural, or depends on runtime behavior, it is not "obvious by inspection" -- require evidence first.

## Investigation loop (logging-heavy default)

1. **Observe the symptom**: actual vs expected.
2. **Form a hypothesis**: keep it tentative.
3. **Collect evidence**: often via the Facts-first logging tactic.
4. **Wait for evidence**: the human runs and provides output.
5. **State facts**: report what the evidence shows.
6. **Repeat** until the root cause statement is isolated.

Gate: after each experiment, report the evidence, state only the facts it supports, update the possibility space, and recommend the next experiment or the move to root cause. Then STOP at the post-experiment gate unless `auto` is carrying the workflow forward.

### Root cause (definition + hard gate)

"Root cause" means the most specific, testable statement that explains the observed symptom and is supported by discriminating evidence (or direct code proof for truly local inspection-only bugs).

Do not declare a root cause prematurely:
- A top-level partition bucket (or a chosen path through the possibility space) is not a root cause if it can be further partitioned into multiple plausible sub-causes.
- Before declaring "this bucket/path is the root cause" or "this bucket/path cannot be partitioned further", inspect the relevant code within the current best-explanation path and attempt at least one further partition based on concrete code structure/branches/contracts.
- If the current best explanation is still a category ("it's in module X", "it's caching", "it's concurrency"), STOP and propose the next partition or the smallest experiment that will partition it further.

Gate: STOP and get explicit approval at the root-cause gate before declaring a root cause, and again at the fix gate before applying a fix.

## Fix gate

Before applying a fix:
- summarize the proposed fix (or the viable fix options)
- explain why it follows from the approved root cause
- state the expected verification evidence if the fix is correct

Gate: STOP at the fix gate and wait for explicit approval before changing code.

## Verify / close gate

After applying an approved fix:
- run or request the planned verification
- report the verification results
- note any cleanup of temporary logs, assertions, or experiment scaffolding

Gate: STOP at the verify/close gate and wait for explicit approval before concluding the debugging workflow.

## Log hygiene

- Remove or disable temporary logs once an experiment is complete and the hypothesis is eliminated.
- If a temporary log remains useful for the next iteration, keep it, but keep the set small and hypothesis-focused.
- Avoid log spam: prefer logging state changes, not every read.

## Stop conditions (workflow-specific)

Use the blocked gate and STOP when:
- **Expected behavior is ambiguous**: product intent is required before you can design a valid discriminating experiment.
- **Required evidence is missing**: the human has not provided the output needed to proceed.
- **Stalled progress**: you've iterated without gaining discriminating evidence. Ask the human for a direction change (different tactic, broader/narrower possibility space, additional evidence, or a workaround). This applies even in `auto`.

## Adapter notes (minimal)

Adapters should only cover:
- Logging APIs and how to capture/share evidence for a stack
- Known tooling limitations that affect evidence collection

