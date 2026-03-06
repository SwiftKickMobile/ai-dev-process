Managed-By: skai
Managed-Id: guide.debugging-tactic-code-bisection
Managed-Source: Guides/Core/debugging-tactic-code-bisection.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-27

# Debugging tactic: code bisection

Purpose: isolate the smallest delta that flips a result (fail -> pass or pass -> fail) by moving in approximate half-steps, systematically, with a clean and reversible state trail.

This is not `git bisect` (history search). This is code-level bisection within an implementation or reproducer.

## Checkpoints

This tactic is used inside the debugging workflow and follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this tactic must STOP and wait at these checkpoints):
- Before starting git-based bisection: get explicit approval to create commits for this tactic (commits are still gated by universal safety rules).
- When the smallest failing delta is isolated: STOP and present the minimal diff + evidence before doing any real fix work.

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

## Preconditions (must be true)

- The symptom is reproducible (ideally via a deterministic reproducer such as a unit test or small harness).
- You can define an approximate notion of "distance" (surface area, branches/variables, config combinations, scope of code paths) so you can move in ~half-steps.
- You have explicit authorization to use git commits for bisection (commits are still gated by universal safety rules).

## Setup (non-polluting)

Goal: do not pollute the branch under work.

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
5. Record the iteration in the working document.

### Commit discipline (required)

- Working tree must be clean before switching between iterations (commit first).
- Commit messages must be descriptive enough to recover from mistakes.
  - Format recommendation: `BISect: step N - <what changed> - <expected> -> <observed>`
- Record: step N, commit SHA, delta summary, pass/fail.

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

