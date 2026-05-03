Managed-By: skai
Managed-Id: guide.debugging-tactic-code-bisection
Managed-Source: Guides/Core/debugging-tactic-code-bisection.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-05-03

# Debugging tactic: code bisection

Purpose: isolate the smallest delta that flips a result (fail -> pass or pass -> fail) by moving in approximate half-steps, systematically, with a clean and reversible state trail.

This is not `git bisect` (history search). This is code-level bisection within an implementation or reproducer.

## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line. The only normal exception is full workflow completion, which uses `🏁 Complete. Let me know if anything needs adjustment.`

Use these standard gate lines:
- Planned gate: `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- Blocked gate: `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

The gate line is exclusively about advance intent. Place questions, proposals, options, drafts, and any other solicitation of human input *above* the gate line as discussion content. Never phrase a gate as "answer X, then I'll do Y" or "pick option A/B/C, then I'll proceed" -- that collapses discussion into advance intent and pre-commits the agent to execute on the next message regardless of how the human responds.

If the human's response contains discussion (answers, refinements, follow-up questions) but not explicit advance intent, treat the response as discussion: incorporate it (revise drafts, refine proposals, ask follow-ups) and re-gate. Do not auto-advance.

Planned gates are the expected review points of this tactic. At each planned gate:
1. Summarize what you did and what should happen next.
2. End with the planned gate line.
3. STOP and wait for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent. If the gate is non-standard, make it describe the exact human response or handoff needed to resume the workflow.

If an unexpected blocker prevents continued work, use the blocked gate line and STOP until the human resolves it.

Workflow-specific gate notes:
- Starting git-based bisection is a hard planned gate because it creates temporary commits and branches.
- Isolating the smallest failing delta is also a hard planned gate: this tactic stops there and hands control back to the main debugging workflow or the human's next decision.

Planned gates for this workflow:
- Before starting git-based bisection: after explaining why bisection is the right tactic, what branch/worktree setup will be created, and why temporary commits are required.
- When the smallest failing delta is isolated: after presenting the minimal diff + evidence and before doing any real fix work.

Workflow-specific blocked gates:
- The symptom is not reproducible enough for bisection to produce meaningful pass/fail results.
- You do not have explicit authorization to create the temporary bisection commits/branches required by this tactic.

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the tactic's active-step rules below. Do not use them to skip required approval for temporary commits or to jump straight from isolation into applying a fix.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- This tactic currently uses inline discussion rather than a required bisection work document.
- The active state lives in the conversation plus the temporary branch/worktree and the sequence of bisection commits.
- At the start gate, STOP before creating the temporary branch/worktree or making the first bisection commit.
- At the isolated-delta gate, STOP after presenting the minimal diff + evidence and before switching into real fix work.

Workflow-specific advance behavior:
- `auto` does not bypass the start gate for git-based bisection.
- `auto` does not bypass the isolated-delta gate.
- Use `start bisection` and `isolated delta` as the stable bounded-auto targets for this tactic.

## Preconditions (must be true)

- The symptom is reproducible (ideally via a deterministic reproducer such as a unit test or small harness).
- You can define an approximate notion of "distance" (surface area, branches/variables, config combinations, scope of code paths) so you can move in ~half-steps.
- You have explicit authorization to use git commits for bisection (commits are still gated by universal safety rules).

## Setup (non-polluting)

Goal: do not pollute the branch under work.

Gate: before doing any setup that creates the temporary branch/worktree or the first bisection commit, STOP at the start gate and get explicit approval.

- Record the current branch name (the branch under work).
- Create a temporary local branch off the current HEAD named `<current branch name>/bisection`.
- If you want the main working tree to remain untouched, use a separate worktree for the bisection branch; otherwise do the work on the bisection branch in the same working tree.

Universal invariant: do not push bisection commits.

## Starting points

You need:

- A failing implementation (production code or a minimal reproducer that still fails), and
- Optionally a simplified working implementation (as small as needed to stop failing).

You can travel in either direction:

- **Trim** a failing implementation toward a working one until it starts working.
- **Add** from a working implementation toward a failing one until it starts failing.

## Iteration loop (one half-step per iteration)

Each iteration is one discriminating experiment:

1. Choose the next half-step change (trim or add) based on the last result.
2. Apply the change.
3. Run the reproducer/test and classify the result (pass/fail).
4. Commit the iteration.
5. Keep enough inline trace in the conversation to explain what changed, what commit was created, and whether the result moved you toward or away from the failing delta.

### Commit discipline (required)

- Working tree must be clean before switching between iterations (commit first).
- Commit messages must be descriptive enough to recover from mistakes.
  - Format recommendation: `BISect: step N - <what changed> - <expected> -> <observed>`
- Record: step N, commit SHA, delta summary, pass/fail.

At minimum, state that information in the conversation as you go so the trail stays recoverable even without a separate work document.

### Switching backward (when you overshoot)

If a half-step overshoots (e.g., it becomes passing when you expected failing), go back to the prior step commit SHA and try the opposite direction with a new half-step.

Preferred safe behavior:
- Check out the prior step commit SHA (detached state) to re-run or to branch a new alternate step from it.

Only if explicitly approved:
- Use destructive branch-pointer moves (e.g., hard resets) to move the bisection branch HEAD backward.

## Exit

When you have isolated the smallest failing delta:

- STOP and present the minimal diff + evidence.
- Switch back to the previous branch (the branch under work) before doing any real fix work.
- Human decides whether to apply a real fix (separate from the bisection branch) or continue bisecting.

Optional cleanup (permission-gated): deleting the temporary bisection branch is a separate action and should be done only if the human explicitly asks.

