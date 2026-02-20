Managed-By: ai-dev-process
Managed-Id: guide.dev-retro
Managed-Source: Guides/Process/dev-retro.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-20

# Dev-session retro (LLM + human)

Purpose: a completeness backstop for an LLM-driven development session in a host repo. This retro covers **everything since the previous dev retro** (if any); if none, it covers the current session.

Do not do a git/diff report unless asked. Prefer evidence-backed review and consistency checks.

## Inputs (read what exists)

Read the documents and artifacts that were produced or used during this session, as applicable:

- The **plan** for the session:
  - work spec doc(s) (if used)
  - planning docs / analysis docs (including unit-testing planning + infrastructure + writing work docs)
- The project's Integration doc:
  - `docs/ai-dev-process/integration.md`
- Evidence artifacts produced during the session:
  - build/test outputs, logs, `.xcresult`, screenshots/screen recordings, crash reports, etc.
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
  - integration doc values/commands (inside managed blocks only, if using `ai-dev-process` Integration format)
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

If there are suggestions, write them to a working file following `Guides/Core/working-doc-conventions.md` (subpath: none, filename: `process-tickets.md`). Use this format for each suggestion:

```
## Suggestion: <concise summary>

**Friction**
<What was painful or suboptimal>

**Suggestion**
<What to change in ai-dev-process>

**Affected files**
<Paths within the ai-dev-process repo, if known. "Unknown" is acceptable.>
```

This file will be used for iteration and, if the human chooses to file tickets, as the basis for issue drafts (see `Guides/Process/process-improvement-tickets.md`).

If nothing stands out, say **"No process suggestions."**

## Retro output (keep it short)

Always start your message with a one-line declaration that the retro was performed:
- `Dev retro: DONE`

Then output only:
- what you fixed immediately (if any)
- remaining follow-ups (1-8 bullets)
- or: "Dev retro complete; no misses found."
- Include process suggestions from step 6 (if any) as a separate **"Process suggestions"** section at the end.
- If there are process suggestions, tell the human: these can be filed as GitHub issues on `ai-dev-process`; we can discuss and revise them first, and when you're happy with them, say the word and I'll draft the tickets.
