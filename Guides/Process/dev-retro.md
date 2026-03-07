Managed-By: skai
Managed-Id: guide.dev-retro
Managed-Source: Guides/Process/dev-retro.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Dev-session retro (LLM + human)

Purpose: a completeness backstop for an LLM-driven development session in a host repo. This retro covers **everything since the previous dev retro** (if any); if none, it covers the current session.

Do not do a git/diff report unless asked. Prefer evidence-backed review and consistency checks.

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
- The process-improvement handoff is a non-standard planned gate. `Next` there means: hand the current process suggestions to `Guides/Process/process-improvement.md`, where that workflow will own ticket drafting, review, and filing.
- At that handoff gate, the retro output's `Process suggestions` section is the handoff artifact. The `🟡` suggestion markers there remain conceptually pending while waiting for the human to approve drafting.

Planned gates for this workflow:
- After the retro output is prepared and process suggestions have been identified, but before handing them off to the process-improvement workflow for drafting.

Workflow-specific blocked gates:
- Required session artifacts are missing or cannot be located, so the retro cannot be completed with evidence.

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the workflow's active-phase rules below. Do not use them to skip required review or filing decisions.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <milestone>` = auto-advance but STOP before the named planned gate. Use stable, workflow-specific milestone names.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- This guide does not require a separate phase marker for ordinary retro work.
- If process suggestions are generated, the workflow-owned handoff artifact is the retro output's `Process suggestions` section.
- In that section, mark each suggestion with `🟡` to show it is pending handoff into the drafting workflow.
- At the process-improvement handoff gate, STOP with those `🟡` suggestion markers still present.
- If the human gives advance intent at that handoff gate, transfer control to `Guides/Process/process-improvement.md` starting at Phase 1, step 2 using the remaining `🟡` suggestions as the draft inputs.
- Do not draft `process-tickets.md` in this guide. `Guides/Process/process-improvement.md` owns ticket drafting, review, and filing.

Workflow-specific advance behavior:
- `auto` may complete the retro itself and may develop process suggestions, but if any suggestions were identified it must still STOP at the process-improvement handoff gate.
- Use `draft process tickets` as the stable bounded-auto target for this guide's planned handoff gate.

## Inputs (read what exists)

Read the documents and artifacts that were produced or used during this session, as applicable:

- The **plan** for the session:
  - work spec doc(s) (if used)
  - planning docs / analysis docs (including unit-testing planning + infrastructure + writing work docs)
- The project's Integration doc:
  - `docs/skai/integration.md`
- Evidence artifacts produced during the session:
  - build/test outputs, logs, result bundles/reports, screenshots/screen recordings, crash reports, etc.
- The canonical requirements library:
  - `/requirements/**` (or your org's equivalent)

If any of these inputs are missing but required to perform the retro, STOP and ask the human where they are.

## Retro checklist

### 1) Self-review (evidence-backed)

- Summarize what was attempted and what was achieved.
- For each key outcome, cite the best available evidence:
  - tests passing/failing + artifacts
  - logs / screenshots / crash output
  - observable behavior changes

### 2) Gaps and flaws

Identify:
- what might still be wrong (known unknowns)
- what assumptions were made
- what risks remain

For each gap, propose the smallest next verification step (or STOP and ask the human for required evidence).

### 3) Plan drift / consistency

If the session deviated from the documented plan(s):
- Update the plan docs so they match reality (or explicitly record why the plan changed).
- Ensure the "next steps" reflect the new reality.

### 4) Documentation updates (only what changed)

If the session changed behavior, conventions, or integration details:
- Update the most relevant docs (choose the minimum set):
  - code comments where correctness depends on subtle behavior
  - README / developer docs
  - integration doc values/commands (inside managed blocks only, if using `skai` Integration format)
  - process docs/runbooks (if a repeatable workflow changed)

If you are not confident what should be documented, STOP and ask the human what level of documentation is expected.

### 5) Product requirements backfill (retro requirements)

Goal: if the session discovered or clarified externally observable behavior, ensure `/requirements/**` reflects it.

Rules (migrated from the former `retro-prd` process):
- Infer externally observable or cross-component behavioral contracts from the code + evidence from this session.
- Compare them to the existing `/requirements/**` library.
- Add missing requirements and update incorrect/outdated ones.
- Do NOT introduce implementation details (types, functions, files, initializers).
- Place each requirement using the project's scope rules (platform/domains/features/apps, etc.).
- Do NOT add progress markers.

Only update `/requirements/**`.

### 6) Process reflection

Reflect on the session since the last retro (or since session start). Consider:

- **Pattern violations**: Did you break an established convention or project pattern? What cue did you miss, and what check would have caught it earlier?
- **Recurring friction**: Were there repeated failures (e.g., build issues, test flakiness, tooling problems, unclear requirements) that a process or infrastructure change could prevent?
- **Missing knowledge**: Did you lack context that a rule, skill, or documentation improvement would provide?
- **Documentation gaps**: Are there undocumented invariants, conventions, or patterns that you had to learn the hard way or that the human had to explain?
- **Human corrections**: Did the human have to point out something you should have caught yourself? What was the root cause -- a missing check, a missing convention, or a gap in your understanding of the project?

Output: 1-4 concrete suggestions (not vague observations). Each suggestion should name the specific file, doc, or artifact to create/update and what it should say. The human will decide which to act on.

Suggestion quality bar (problem-first; solution optional):
- Required for a ticket-worthy suggestion:
  - **Friction/problem** (concrete)
  - **Evidence/example** from the session (what happened)
  - **Failure mode** it prevents (what goes wrong if this repeats)
- Optional (only if confident):
  - **Candidate approach** (direction, not a mandate)
  - **Likely file(s)** to change (or "Unknown")
  - **Verification plan** (how to know next time)
- Hard rule: do not output suggestions that are purely abstract ("be more careful", "improve quality") without evidence and a verifiable check.

If there are suggestions:
- list them in the retro output under a **"Process suggestions"** section
- format each one as a `🟡` suggestion with enough detail for `Guides/Process/process-improvement.md` to draft a ticket:
  - concise summary
  - friction/problem
  - evidence/example
  - failure mode
  - optional candidate approach
  - likely affected files (or `Unknown`)
  - optional verification
- do not draft `process-tickets.md` here

If nothing stands out, say **"No process suggestions."**

## Retro output (keep it short)

Always start your message with a one-line declaration that the retro was performed:
- `Dev retro: DONE`

Then output only:
- what you fixed immediately (if any)
- remaining follow-ups (1-8 bullets)
- or: "Dev retro complete; no misses found."
- Include process suggestions from step 6 (if any) as a separate **"Process suggestions"** section at the end.
- If there are process suggestions: end with `⏳ GATE: Next: Draft process tickets from these retro suggestions. Say "next" or what to change.`
- On advance intent from that gate, follow `Guides/Process/process-improvement.md` starting at Phase 1, step 2.
