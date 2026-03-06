Managed-By: skai
Managed-Id: guide.process-improvement
Managed-Source: Guides/Process/process-improvement.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-06

# Process improvement

Purpose: capture process improvement suggestions and file them as GitHub issues on the `skai` repo. Handles both ad-hoc suggestions (human describes a problem or idea) and retro handoffs (drafts already written by the retro).

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After drafts are written to the working file: present for review and wait for advance intent before proceeding to filing.
- After filing issues: report results and STOP (complete).

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

## Confidentiality rule

This repo is public. Issues must **never** contain project names, client names, domain-specific details, or any information that identifies the source project. Every suggestion must be generalized into a process-level observation.

- Bad: "In Project Acme, the retro missed security edge cases in the auth module"
- Good: "The retro template does not prompt for security considerations, which can lead to gaps in auth-related work"

The human reviews all issue content before creation (Phase 2). This is the final gate, but the agent should make a good-faith effort to generalize first.

## Entry points

- **Ad-hoc**: the human describes a process problem, improvement idea, or feature request. Start at Phase 1.
- **From retro**: the dev retro's process reflection step writes drafts to the working file and hands off here. Skip Phase 1 and start at Phase 2.

## Phase 1: Draft

Goal: produce one or more ticket drafts in the `process-tickets.md` working file (path per `Guides/Core/working-doc-conventions.md`, subpath: none, filename: `process-tickets.md`).

### 1. Understand the suggestion

Ask the developer to describe what they have in mind. Use follow-up questions to clarify:

- What were you trying to do?
- What went wrong or felt harder than it should?
- Did this happen once, or is it a recurring pattern?
- Do you have a sense of what a fix would look like?

Keep the conversation natural -- don't interrogate. The goal is to get enough detail to write an actionable draft.

If the developer provides a clear description up front, skip the questions and proceed to drafting.

### 2. Write drafts to the working file

Write a ticket draft entry to `process-tickets.md`. If the file already exists (e.g., from a previous session), append to it. If it doesn't exist, create it.

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

Present the drafts to the developer for review and iterate until they're satisfied.

Checkpoint: STOP after drafts are written and reviewed. Ask the developer if they'd like to file now or leave drafts in the working file for later. If they want to file, advance to Phase 2.

## Phase 2: File

Goal: file approved drafts as GitHub issues.

### 1. Check GitHub MCP access

Verify that a GitHub MCP server is configured and accessible.

If it is not available, tell the human that a GitHub MCP server is required for ticket creation and offer to help set one up (e.g., adding a GitHub MCP server to the project's MCP configuration with a personal access token that has repo issue permissions).

Do not proceed until GitHub MCP access is confirmed.

### 2. Present drafts for review

Locate the `process-tickets.md` working file (path per `Guides/Core/working-doc-conventions.md`). Read it and present all 🟡 entries to the human.

Checkpoint: present the entries and STOP with the standard gate line. On advance intent, proceed to step 3 for all 🟡 entries. If the human requests changes (revisions, skipping specific entries), apply them and re-present.

To skip an entry: remove the 🟡 marker and strike through the heading (e.g., `## ~~Ticket: <summary>~~`).

### 3. Create issues

For each 🟡 entry remaining after review:

1. Use the GitHub MCP server to create the issue on the `skai` repo with the labels from the draft.
2. Update the working file: remove the 🟡 marker from the heading and add `**Filed**: #<number>` below the heading.

Report the created issue numbers and URLs back to the human.
