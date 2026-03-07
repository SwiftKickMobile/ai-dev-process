Managed-By: skai
Managed-Id: guide.process-improvement
Managed-Source: Guides/Process/process-improvement.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Process improvement

Purpose: capture process improvement suggestions and file them as GitHub issues on the `skai` repo. Handles both ad-hoc suggestions (human describes a problem or idea) and retro handoffs (drafts already written by the retro).

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
- The Phase 1 understanding gate is a non-standard planned gate. `Next` there means: the current understanding is good enough to draft ticket(s) now.
- The Phase 1 draft-review gate is the main review/file gate for this workflow. `Next` there means: file the remaining approved `## 🟡 Ticket: ...` entries now. The human may instead ask to revise drafts further or leave them for later.

Planned gates for this workflow:
- After Phase 1 understanding is complete, but before drafting ticket(s).
- After Phase 1 drafts are written or updated in `working-docs/<branch-path>/<session-name>/process-tickets.md` and presented for review, but before deciding whether to file them now or leave them for later.

Workflow-specific blocked gates:
- GitHub MCP access is unavailable or misconfigured, so filing cannot proceed.
- The required `process-tickets.md` draft file cannot be located and the human has not provided the session/path needed to continue.

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
- The workflow-owned artifact is `working-docs/<branch-path>/<session-name>/process-tickets.md`.
- In that artifact, each `## 🟡 Ticket: ...` heading is a pending draft / pending filing marker.
- The Phase 1 understanding gate does not require its own workflow artifact or marker; it is a human approval stop before drafting begins.
- At the Phase 1 draft-review gate, STOP with the `## 🟡 Ticket: ...` markers still present.
- If the human gives advance intent at the Phase 1 draft-review gate, create GitHub issues for the remaining approved `## 🟡 Ticket: ...` entries and then update each filed entry by removing `🟡` from the heading and adding `**Filed**: #<number>`.
- If the human decides to skip an entry during draft review, remove the `🟡` marker and strike through the heading instead of filing it.

Workflow-specific advance behavior:
- `auto` may move past the understanding gate and may draft suggestions, but it must STOP at the draft-review gate before creating issues.
- Use `ready to draft` and `draft review` as the stable bounded-auto targets for this guide.

## Confidentiality rule

This repo is public. Issues must **never** contain project names, client names, domain-specific details, or any information that identifies the source project. Every suggestion must be generalized into a process-level observation.

- Bad: "In Project Acme, the retro missed security edge cases in the auth module"
- Good: "The retro template does not prompt for security considerations, which can lead to gaps in auth-related work"

The human reviews all issue content before creation at the draft-review gate. This is the final approval point before filing, but the agent should make a good-faith effort to generalize first.

## Entry points

- **Ad-hoc**: the human describes a process problem, improvement idea, or feature request. Start at Phase 1.
- **From retro**: the dev retro's process reflection step develops one or more process suggestions and hands them off here. Skip the ad-hoc discussion step and start at Phase 1, step 2 so this guide owns drafting, review, and filing.

## Phase 1: Draft

Goal: produce one or more ticket drafts in `working-docs/<branch-path>/<session-name>/process-tickets.md`.

Before writing the working file, choose a `session-name` for this suggestion-filing effort so multiple draft sessions on the same branch stay separate.

### 1. Understand the suggestion

This step is for the **ad-hoc** entry path only. If you arrived here from `Guides/Process/dev-retro.md`, skip this step and use the handed-off suggestions as the draft inputs for step 2.

Ask the developer to describe what they have in mind. Use follow-up questions to clarify:

- What were you trying to do?
- What went wrong or felt harder than it should?
- Did this happen once, or is it a recurring pattern?
- Do you have a sense of what a fix would look like?

Keep the conversation natural -- don't interrogate. The goal is to get enough detail to write an actionable draft.

If the developer provides a clear description up front, skip the questions and proceed to drafting.

Gate: once you believe you understand the suggestion well enough to draft actionable ticket(s), STOP and end with `⏳ GATE: Next: Draft the process ticket(s). Say "next" or what to change.`

At this gate:
- `next` / advance intent = draft the ticket(s)
- requested changes or clarifications = continue discussion, then stop again at the same gate when ready to draft

### 2. Write drafts to the working file

Write a ticket draft entry to `process-tickets.md` at `working-docs/<branch-path>/<session-name>/process-tickets.md`. If the file already exists for this session, append to it. If it doesn't exist, create it.

Draft inputs:
- **Ad-hoc path**: use the understanding developed in step 1.
- **Retro handoff path**: use the concrete process suggestions handed off from `Guides/Process/dev-retro.md`.

If the retro handoff is not specific enough to draft actionable tickets, STOP and ask the human to clarify before writing the drafts.

Draft format:

```
## 🟡 Ticket: <concise summary>

**Labels**: <labels>

**Friction**
<What was painful or suboptimal, described generically -- no project references>

**Evidence/example**
<What happened that shows the problem (generic; no project references)>

**Failure mode**
<What goes wrong if this repeats>

**Candidate approach** (optional)
<A possible fix direction, if confident>

**Affected files**
<Paths within the skai repo, if known. "Unknown" is acceptable.>

**Verification** (optional)
<How to confirm the change worked next time>
```

For `**Labels**`: choose from this list (source of truth for this workflow):
- `enhancement`
- `bug`
- `documentation`
- `help wanted`
- `question`
- `invalid`

Apply the confidentiality rule to all drafts.

Present the drafts to the developer for review.

Gate: STOP after drafts are written or updated and presented. End with `⏳ GATE: Next: File the drafted process tickets. Say "next", "leave for later", or what to change.`

At this gate:
- `next` / advance intent = file the remaining approved `## 🟡 Ticket: ...` entries now
- `leave for later` = end the workflow without filing; keep the `## 🟡 Ticket: ...` markers in place in `process-tickets.md`
- requested changes = revise drafts, optionally skip specific entries, re-present them, and stop again at the same gate

## Phase 2: File

Goal: file approved drafts as GitHub issues.

### 1. Check GitHub MCP access

Verify that a GitHub MCP server is configured and accessible.

If it is not available, tell the human that a GitHub MCP server is required for ticket creation and offer to help set one up (e.g., adding a GitHub MCP server to the project's MCP configuration with a personal access token that has repo issue permissions).

Do not proceed until GitHub MCP access is confirmed.

### 2. Read the approved drafts

Locate the `process-tickets.md` working file at `working-docs/<branch-path>/<session-name>/process-tickets.md`. Read the remaining `## 🟡 Ticket: ...` entries that the draft-review gate left in place.

To skip an entry: remove the 🟡 marker and strike through the heading (e.g., `## ~~Ticket: <summary>~~`).

### 3. Create issues

For each 🟡 entry remaining after review:

1. Use the GitHub MCP server to create the issue on the `skai` repo with the labels from the draft.
2. Update the working file: remove the 🟡 marker from the heading and add `**Filed**: #<number>` below the heading.

Report the created issue numbers and URLs back to the human.

Then complete the workflow.
