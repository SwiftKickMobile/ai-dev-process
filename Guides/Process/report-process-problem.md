Managed-By: ai-dev-process
Managed-Id: guide.report-process-problem
Managed-Source: Guides/Process/report-process-problem.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-20

# Report a process problem

Purpose: structured intake for ad-hoc process improvement suggestions outside of a dev retro. Helps the developer articulate friction and produces entries in the same working file format used by the retro and ticket creation flows.

## When to use

- The developer hits a process pain point mid-session and wants to capture it while the context is fresh.
- The developer has a process idea that doesn't come from a retro.

This guide feeds into `Guides/Process/process-improvement-tickets.md` when the developer is ready to file tickets.

## Procedure

### 1. Understand the problem

Ask the developer to describe the friction. Use follow-up questions to clarify:

- What were you trying to do?
- What went wrong or felt harder than it should?
- Did this happen once, or is it a recurring pattern?
- Do you have a sense of what a fix would look like?

Keep the conversation natural -- don't interrogate. The goal is to get enough detail to write an actionable suggestion.

After discussing the problem, let the developer know: when you're ready, I can draft this into the working file for ticket creation. Let them drive the timing.

### 2. Write to the working file

When the developer indicates they're ready, write a suggestion entry to the `process-tickets.md` working file (path per `Guides/Core/working-doc-conventions.md`, subpath: none, filename: `process-tickets.md`).

If the file already exists (e.g., from a retro or a previous report), append to it. If it doesn't exist, create it.

Use this format:

```
## Suggestion: <concise summary>

**Friction**
<What was painful or suboptimal>

**Suggestion**
<What to change in ai-dev-process>

**Affected files**
<Paths within the ai-dev-process repo, if known. "Unknown" is acceptable.>
```

Present the entry to the developer for review and iterate until they're satisfied.

### 3. Next steps

Ask the developer if they'd like to file the suggestion as a GitHub issue on `ai-dev-process` now, or leave it in the working file for later.

If they want to file now, proceed to `Guides/Process/process-improvement-tickets.md`.
