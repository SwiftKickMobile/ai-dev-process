Managed-By: ai-dev-process
Managed-Id: guide.process-improvement-tickets
Managed-Source: Guides/Process/process-improvement-tickets.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-28

# Process improvement tickets

Purpose: convert process improvement suggestions into GitHub issues on the `ai-dev-process` repo. Typically triggered after a dev retro (step 6: Process reflection), but can be used any time a process gap is identified.

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- If GitHub MCP access is missing/broken: STOP and ask the human to set it up (or to fix permissions).
- After converting suggestions into drafts and updating the working file: present the updated working file for human review and STOP.
- After the human marks drafts approved: STOP before creating issues unless the human provides advance intent to proceed with filing.
- After creating issues: report results and STOP (complete).

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

## Confidentiality rule

This repo is public. Issues must **never** contain project names, client names, domain-specific details, or any information that identifies the source project. Every suggestion must be generalized into a process-level observation.

- Bad: "In Project Acme, the retro missed security edge cases in the auth module"
- Good: "The retro template does not prompt for security considerations, which can lead to gaps in auth-related work"

The human reviews all issue content before creation (step 3). This is the final gate, but the agent should make a good-faith effort to generalize first.

## Prerequisites

- One or more process improvement suggestions (from a retro or ad-hoc observation).
- A GitHub MCP server configured with issue creation permissions.
- The `ai-dev-process` GitHub repo identifier (e.g., `owner/ai-dev-process`). Ask the human if you don't know it.

## Procedure

### 0. Check GitHub MCP access

Before drafting any issues, verify that a GitHub MCP server is configured and accessible.

If it is not available, tell the human that a GitHub MCP server is required for ticket creation and offer to help set one up (e.g., adding a GitHub MCP server to the project's MCP configuration with a personal access token that has repo issue permissions).

Do not proceed until GitHub MCP access is confirmed.

### 1. Locate the working file

Look for the `process-tickets.md` working file (path per `Guides/Core/working-doc-conventions.md`). This file is created by the retro's process reflection step with initial suggestions.

If the file exists, read it and proceed to step 2.

If it does not exist (e.g., this was triggered outside a retro), create it and ask the human what process improvements they'd like to file. Write each suggestion using the format below and proceed to step 2.

### 2. Convert suggestions to issue drafts

For each suggestion in the working file, convert it to an issue draft by adding a `**Status**: draft` field. Apply the confidentiality rule -- generalize any project-specific language. The result for each entry should look like:

```
## Ticket: <concise summary of the process improvement>

**Status**: draft
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
<Paths within the ai-dev-process repo, if known. "Unknown" is acceptable.>

**Verification** (optional)
<How to confirm the change worked next time>
```

For the `**Labels**` field: choose from this explicit list (source of truth for this workflow):
- `enhancement`
- `bug`
- `documentation`
- `help wanted`
- `question`
- `invalid`

Update the working file and present it to the human for review. Do not create any issues yet.

### 3. Human review and iteration

The human reviews the working file and may:

- Approve all drafts as-is.
- Edit drafts directly in the working file or ask the agent to revise them.
- Remove drafts they don't want filed.
- Add new drafts.

Iterate on the working file until the human explicitly approves. Mark approved drafts with `**Status**: approved`.

### 4. Create issues

For each approved draft:

1. Use the GitHub MCP server to create the issue on the `ai-dev-process` repo with the labels from the draft.
2. Update the draft's status in the working file to `**Status**: filed (#<number>)`.

Report the created issue numbers and URLs back to the human.
