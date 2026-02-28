# `ai-dev-process`

Reusable, mostly IDE-agnostic **agentic coding guides** and **policies** you can vendor into any repo.

This repo is designed to be installed as a **git submodule** and activated by an **LLM-runbook-driven installer** that:
- inspects the host repo (including legacy installs),
- proposes a migration plan,
- writes **managed files** only (safe updates),
- installs agent-facing assets into the host repo's expected directories (IDE/agent-specific),
- generates IDE-specific artifacts (e.g., Cursor `.mdc`) into the host repo.
- updates Cursor/Claude ignore files using managed blocks so multi-agent installs can coexist cleanly (permission-gated if the ignore files already exist).

Recommended host locations for agent-facing docs:
- Cursor: `.cursor/skills/ai-dev-process-*/`
- Claude Code: `.claude/skills/ai-dev-process-*/`

## Quick start (recommended)

### 1) Add the submodule (one-time)

From your host repo root:

```bash
git submodule add <REPO_URL> Submodules/ai-dev-process
git submodule update --init --recursive
```

If you already have the submodule installed, just run:

```bash
git submodule update --init --recursive
```

### 2) Ask your agent to install/update (copy/paste)

Paste ONE of these prompts into your agent chat (from the host repo root).

#### Cursor prompt

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/Cursor/install-update-cursor.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` ([`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

#### JetBrains (IntelliJ IDEA) + Claude Code prompt

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/JetBrains-ClaudeCode/install-update-jetbrains-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` ([`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

#### Xcode (MCP-hosted) + Claude Code prompt (EXPERIMENTAL)

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/Xcode-ClaudeCode/install-update-xcode-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` ([`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.
> - This runbook is experimental: if any Xcode+MCP assumption doesn't match this repo's setup, STOP and ask me what convention to use.

#### Android Studio + Claude Code prompt (Android stack)

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/AndroidStudio-ClaudeCode/install-update-androidstudio-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` ([`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

### 3) Set up a GitHub MCP server (optional)

Some workflows (process improvement ticket filing, ticket implementation) require a GitHub MCP server to create and manage GitHub issues. If you don't need these workflows, skip this step.

Add a GitHub MCP server to your IDE's MCP configuration with a personal access token that has repo issue permissions. The JSON format is the same across IDEs:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    }
  }
}
```

**Config file locations:**
- Cursor: `.cursor/mcp.json` (project-level) or `~/.cursor/mcp.json` (global)
- Claude Code: `~/.claude.json`

## How installs stay safe

- **Integration doc (project-owned)**: [`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md) is the single source of truth for project-specific commands/paths (build/test/lint/etc). Templates live in [`Templates/`](Templates/).
- **Managed files**: host-project files written by the installer have a required header (see [`Install/managed-header.md`](Install/managed-header.md)). The installer overwrites only files that already contain this header.
- **Legacy installs**: lookalike files without the header are treated as **legacy candidates** and are not overwritten by default (see [`Install/conflict-precedence-policy.md`](Install/conflict-precedence-policy.md)).

## Integration document (how to use it)

The Integration doc ([`docs/ai-dev-process/integration.md`](docs/ai-dev-process/integration.md)) is the **project-owned** place where `ai-dev-process` workflows get the concrete, copy/pasteable details they need to run deterministically (build/test commands, destinations, artifact paths, evidence expectations).

Why it matters:
- It prevents agents from guessing project-specific constants (like `xcodebuild -destination` strings, scheme/test plan conventions, or where `.xcresult` / logs are stored).
- It makes install/update migrations safe: the installer can preserve your filled values while updating the managed template structure around them.

How humans should fill it:
- **🟡 means "required project-specific value is missing."**
- Under a 🟡 item you may see one or more `INSTRUCTION:` lines. Those are **not part of the long-term document**; they exist only to explain what to fill in.
- When you fill a value:
  - remove the 🟡 marker
  - delete the `INSTRUCTION:` line(s) under it
- If a future install/update can't infer a required value with high confidence, the installer may **restore** 🟡 + `INSTRUCTION:` prompts so the doc remains a complete, reliable source of truth.

## IDE clutter / autocomplete (recommended)

To reduce duplicate autocomplete/search results (submodule sources + installed assets), hide the submodule in your editor UI while keeping the submodule [`README.md`](README.md) visible.

Example for Cursor/VS Code workspace settings (`.vscode/settings.json`):

```json
{
  "files.exclude": {
    "**/Submodules/ai-dev-process/**": true,
    "**/Submodules/ai-dev-process/README.md": false
  },
  "search.exclude": {
    "**/Submodules/ai-dev-process/**": true,
    "**/Submodules/ai-dev-process/README.md": false
  }
}
```

Android Studio (JetBrains):
- In the Project tool window, right-click `Submodules/ai-dev-process` → **Mark Directory as** → **Excluded**.
- Optionally also exclude `.claude/skills/ai-dev-process-*` if you don't want the skill install artifacts in search results.
- Prefer local IDE excludes over committing `.idea` changes unless your repo explicitly versions IDE config.

## Usage

After installation, workflows are available as **skills** that your agent activates automatically based on conversational context.

### How all workflows work

**Minimal command structure.** You don't need to memorize strict commands:

1. Start with context that activates a skill/workflow (e.g. "Write a work spec", "Start unit testing", "Start debugging").
2. Progress through steps using **advance intent** (e.g. "begin", "next", "continue", "go ahead").
3. At certain steps the agent will **stop and wait** at a checkpoint (gate). At checkpoints, the agent ends its output with exactly one of these lines:
   - `⏳ GATE: Next: <thing>. Say "next" or what to change.`
   - `⏳ GATE: Complete. Say "next" or what to change.`
   - `⏳ GATE: Blocked. Say "next" or what to change.`
4. Saying "next" (or similar) at a checkpoint counts as approval to proceed.
5. Adding **"auto"** to advance intent tells the agent to proceed without stopping at checkpoints, unless a universal STOP condition applies (e.g. "next auto", "begin auto").
6. You can bound auto: **"auto to <milestone>"** means "proceed until you are about to begin the milestone, then stop" (e.g. "next auto to task 7").

**Agent note-taking and progress tracking.** The agent keeps structured notes so you can pick up where you left off:

- The agent uses **🟡** markers in working documents to flag TODOs, open questions, and items pending your approval (including "implemented but pending approval").
- 🟡 markers should be individually updatable (use stable identifiers like task numbers, section names, or explicit IDs on the same line).
- 🟡 markers are removed only when items are resolved and you give approval.

**Retros on demand.** At any point you can ask the agent to **"retro"** to check for gaps, update documents, backfill requirements, and reflect on process.

### Work spec creation (skill `ai-dev-process-work-spec-creation`)

Structured planning and specification for complex features. Produces a planning document (design decisions, API sketch), normalized product requirements, and a work specification (tasks, subtasks, requirements traceability).

- Guide [`Guides/Spec/work-spec-creation.md`](Guides/Spec/work-spec-creation.md)

**Prerequisites:** Discuss the feature or problem with the agent in some detail before initiating the process -- scope, motivating use cases, architecture, solution approaches. A thorough upfront discussion produces a much higher quality first draft and minimizes iteration. A Jira ticket, feature request, or problem statement is a good starting point.

**Phases:**

1. **Planning document.** Agent summarizes the discussion into a document seeded with 🟡 open questions. Optional: for large efforts, the planning document can be organized into explicit phase sections; each phase runs its own mini-cycle (discussion, API sketch, requirements normalization, work spec) before moving to the next. Checkpoint: human reviews and resolves 🟡 items.
2. **Design discussion.** Agent proposes, human decides. Iterates until all 🟡 items are resolved.
3. **API sketch.** Agent drafts the API surfaces implied by the design. Checkpoint: human confirms the design is ready to proceed.
4. **Requirements normalization.** Agent promotes behaviors from the planning document into canonical requirements. Checkpoint: human acknowledges requirements updates.
5. **Work spec first pass.** Agent writes top-level tasks only (no subtasks). Checkpoint: human reviews the task list.
6. **Work spec second pass.** Agent adds subtasks, requirement IDs, and traceability mapping. Checkpoint: human reviews the completed work spec.

### Work spec implementation (skill `ai-dev-process-work-spec-implementation`)

Execute tasks from a completed work spec, one top-level task per cycle.

- Guide [`Guides/Spec/work-spec-implementation.md`](Guides/Spec/work-spec-implementation.md)

**Prerequisites:** A completed work spec.

**Phases (repeating):**

1. **Implement next top-level task.** Agent implements all subtasks under Task N. Checkpoint: agent stops after finishing Task N and waits before moving to Task N+1.

### Unit testing (skill `ai-dev-process-unit-testing`)

Plan-first testing workflow. The agent plans all tests upfront, then implements them one logical section at a time (e.g. "Success Tests", "Error Handling Tests"). Handles new test suites, additions to existing suites, and fixing failing tests.

- Guides [`Guides/Test/unit-testing-guide.md`](Guides/Test/unit-testing-guide.md), [`Guides/Test/unit-test-planning-guide.md`](Guides/Test/unit-test-planning-guide.md), [`Guides/Test/unit-test-infrastructure-guide.md`](Guides/Test/unit-test-infrastructure-guide.md), [`Guides/Test/unit-test-writing-guide.md`](Guides/Test/unit-test-writing-guide.md)

**Prerequisites:** Source code to test. Often triggered by a work spec task, but can be used independently.

**Phases:**

1. **Planning.** Agent creates test files organized into sections, with test stubs in each. Doc comments on every stub serve as the test plan. Checkpoint: human reviews the plan before implementation begins.
2. **Infrastructure** (per section). Agent identifies required test infrastructure for the current section (mocks, stubs, fixtures) and proposes additions. Checkpoint: human approves infrastructure changes (if any).
3. **Writing** (per section). Agent implements tests and then runs them. Checkpoints: agent stops after writing (before running tests), and stops after test results to confirm conclusions and next steps (including any proposed production-code fixes).

Phases 2-3 repeat for each section until none remain.

### Debugging (skill `ai-dev-process-debugging`)

Evidence-first problem resolution. Prevents "guessing fixes" loops by requiring observable evidence before drawing conclusions. The process defines a toolkit of effective strategies (targeted logging, possibility-space partitioning, minimal reproducers, bisection, invariant assertions) that guide the agent's debugging approach.

- Guides [`Guides/Core/debugging-guide.md`](Guides/Core/debugging-guide.md), [`Policies/debugging-process-rule.md`](Policies/debugging-process-rule.md)

**Prerequisites:** A bug, crash, or unexpected behavior to investigate. Provide whatever evidence you have (error messages, logs, screenshots, steps to reproduce).

**Phases (repeating):**

1. **Hypothesize and experiment.** Agent states the current possibility space, chooses a tactic, and proposes the smallest discriminating experiment. Checkpoints: pre-experiment, post-experiment, root cause, fix, and verify/close.

Repeats until the root cause is isolated.

### Dev retro (skill `ai-dev-process-dev-retro`)

Completeness backstop that can be used at any point during any workflow. Reviews what has transpired since the last retro (or since session start), identifies gaps, reconciles plan drift, updates documentation, backfills requirements, and reflects on process.

- Guide [`Guides/Process/dev-retro.md`](Guides/Process/dev-retro.md)

**Prerequisites:** Work to review. The agent reads work specs, planning docs, evidence artifacts, and requirements produced since the last retro.

**Phases:**

1. **Retro.** Agent performs the full checklist and reports findings with concrete follow-up suggestions. Checkpoint: agent stops after the retro output.
2. **Process suggestions.** If the retro identifies process improvements, agent writes them to the `process-tickets.md` working file. Checkpoint: human reviews and refines suggestions.
3. **Ticket filing (optional).** When the human is ready, agent files suggestions as GitHub issues on `ai-dev-process`:
   1. Agent checks GitHub MCP access, converts suggestions to issue drafts (applying confidentiality rules to strip project-specific details), and assigns labels from the explicit label list in [`Guides/Process/process-improvement-tickets.md`](Guides/Process/process-improvement-tickets.md).
   2. Human reviews drafts in the working file. Can edit, remove, or add drafts. Iterates until satisfied.
   3. Agent creates approved issues via MCP and records issue numbers in the working file.

### Report process problem (skill `ai-dev-process-report-process-problem`)

Ad-hoc intake for process friction or improvement ideas outside of a retro. The agent helps the developer articulate the problem and captures it in the `process-tickets.md` working file for later review or ticket filing.

- Guide [`Guides/Process/report-process-problem.md`](Guides/Process/report-process-problem.md)

**Prerequisites:** None -- can be triggered at any point during a session.

**Phases:**

1. **Intake.** Agent asks clarifying questions to understand the friction. Checkpoint: wait for the developer to say they are ready to capture the suggestion.
2. **Capture.** When the developer is ready, agent writes a structured suggestion to the working file for review and iteration. Checkpoint: human reviews the entry.
3. **Ticket filing (optional).** When the developer is ready, agent files suggestions as GitHub issues on `ai-dev-process`:
   1. Agent checks GitHub MCP access, converts suggestions to issue drafts (applying confidentiality rules to strip project-specific details), and assigns labels from the explicit label list in [`Guides/Process/process-improvement-tickets.md`](Guides/Process/process-improvement-tickets.md).
   2. Developer reviews drafts in the working file. Can edit, remove, or add drafts. Iterates until satisfied.
   3. Agent creates approved issues via MCP and records issue numbers in the working file.

### Update installation (skill `ai-dev-process-update-installation`)

Check for upstream `ai-dev-process` updates, review what changed, and re-run adapter runbooks.

- Guide [`Guides/Core/update-installation-guide.md`](Guides/Core/update-installation-guide.md)

**Prerequisites:** An existing installation ([`docs/ai-dev-process/install-state.json`](docs/ai-dev-process/install-state.json) must exist from the initial install).

**Phases:**

1. **Check and report.** Agent checks for upstream changes, pulls the latest, and presents the changelog delta. Checkpoint: human acknowledges before any runbooks are re-run.
2. **Re-run adapters.** Agent re-runs each installed adapter's install/update runbook to pick up new or changed assets.

### Working documents

- Working documents (plans, specs, progress logs) live under [`working-docs/`](working-docs/) and are organized by your current git branch.
- `<branch-path>` is the current branch name, with `/` decomposed into nested folders (so `feature/foo` becomes `feature/foo/`).
- Working docs are ephemeral and typically git-ignored.

## Development guide (for contributors)

This repo is typically maintained with an LLM. When you ask an LLM to make changes, point it at:
- [`README.md`](README.md) (this file)
- [`maintain-ai-dev-process.md`](maintain-ai-dev-process.md) (LLM maintainer runbook)
- [`maintain-retro.md`](maintain-retro.md) (post-change checklist to ensure nothing was forgotten)
- [`maintain-tickets.md`](maintain-tickets.md) (work through process improvement tickets labeled `agent ready`)

### LLM-assisted change workflow (recommended)

- Ask the LLM to:
  - propose a small plan,
  - make the minimal edits,
  - then report exactly which files changed and why.
- Require safety defaults (unless you explicitly override):
  - no commits
  - no deletions
  - no dependency changes

### Contributor rules

- Keep sources in this repo **IDE-neutral** (`.md`). IDE-specific outputs are generated into host repos by install/update runbooks.
- If you add/move/rename an asset, update:
  - [`assets.manifest.json`](assets.manifest.json)
  - [`README.md`](README.md) Usage section (developer-facing guides should be documented at file level)
- Update [`CHANGELOG.md`](CHANGELOG.md) for user-visible changes.
- Do not introduce scripts that mutate host repos; installers are LLM-runbook-driven and must follow the safety policies in [`Policies/`](Policies/).

