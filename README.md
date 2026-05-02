# `skai`

Reusable, mostly IDE-agnostic **agentic coding guides** and **policies** you can vendor into any repo.

This repo is designed to be installed as a **git submodule** and activated by an **LLM-runbook-driven installer** that:
- inspects the host repo (including legacy installs),
- proposes a migration plan,
- writes **managed files** only (safe updates),
- installs agent-facing assets into the host repo's expected directories (IDE/agent-specific),
- generates IDE-specific artifacts (e.g., Cursor `.mdc`) into the host repo.
- updates agent ignore files using managed blocks so multi-agent installs can coexist cleanly (permission-gated if the ignore files already exist).

Recommended host locations for agent-facing docs:
- Cursor: `.cursor/skills/skai-*/`
- Claude Code: `.claude/skills/skai-*/`
- Codex: `.agents/skills/skai-*/`

## Contents

- [Quick start](#quick-start-recommended)
- [How installs stay safe](#how-installs-stay-safe)
- [Integration document](#integration-document-how-to-use-it)
- [Project requirements (PRD)](#project-requirements-prd)
- [IDE clutter / autocomplete](#ide-clutter--autocomplete-recommended)
- [Usage](#usage)
  - [How all workflows work](#how-all-workflows-work)
  - [Work spec creation](#work-spec-creation-skill-skai-work-spec-creation)
  - [Work spec implementation](#work-spec-implementation-skill-skai-work-spec-implementation)
  - [Unit testing](#unit-testing-skill-skai-unit-testing)
  - [Debugging](#debugging-skill-skai-debugging)
  - [Dev retro](#dev-retro-skill-skai-dev-retro)
  - [Suggestion](#suggestion-skill-skai-suggestion)
  - [Update installation](#update-installation-skill-skai-update-installation)
  - [Working documents](#working-documents)
- [Development guide (for contributors)](#development-guide-for-contributors)

## Quick start (recommended)

### 1) Add the submodule (one-time)

From your host repo root:

```bash
git submodule add <REPO_URL> Submodules/skai
git submodule update --init --recursive
```

If you already have the submodule installed, just run:

```bash
git submodule update --init --recursive
```

### 2) Ask your agent to install/update (copy/paste)

Paste ONE of these prompts into your agent chat (from the host repo root).

#### Cursor prompt

> Install/update `skai` in this repo by following `Submodules/skai/Install/Cursor/install-update-cursor.md` ([`Install/Cursor/install-update-cursor.md`](Install/Cursor/install-update-cursor.md)).
>
> - If the `Submodules/skai` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/skai/integration.md` ([`docs/skai/integration.md`](docs/skai/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: skai`). Treat lookalike files without the header as legacy candidates.

#### Claude Code prompt

> Install/update `skai` in this repo by following `Submodules/skai/Install/ClaudeCode/install-update-claudecode.md` ([`Install/ClaudeCode/install-update-claudecode.md`](Install/ClaudeCode/install-update-claudecode.md)).
>
> - If the `Submodules/skai` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/skai/integration.md` ([`docs/skai/integration.md`](docs/skai/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: skai`). Treat lookalike files without the header as legacy candidates.

#### Codex prompt

> Install/update `skai` in this repo by following `Submodules/skai/Install/Codex/install-update-codex.md` ([`Install/Codex/install-update-codex.md`](Install/Codex/install-update-codex.md)).
>
> - If the `Submodules/skai` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/skai/integration.md` ([`docs/skai/integration.md`](docs/skai/integration.md)) as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Use `.agents/AGENTS.md` as the Codex instruction file.
> - Only overwrite files that contain the managed header (`Managed-By: skai`). Treat lookalike files without the header as legacy candidates.

These runbooks work with any IDE (JetBrains, Xcode, Android Studio, VS Code, etc.) or standalone agent environment. Stack-aware runbooks auto-detect the project stack and apply the appropriate guidance.

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
- Codex: use the MCP configuration location supported by your Codex environment.

## How installs stay safe

- **Integration doc (project-owned)**: [`docs/skai/integration.md`](docs/skai/integration.md) is the single source of truth for project-specific commands/paths (build/test/lint/etc). Templates live in [`Templates/`](Templates/).
- **Managed files**: host-project files written by the installer have a required header (see [`Install/managed-header.md`](Install/managed-header.md)). The installer overwrites only files that already contain this header.
- **Legacy installs**: lookalike files without the header are treated as **legacy candidates** and are not overwritten by default (see [`Install/conflict-precedence-policy.md`](Install/conflict-precedence-policy.md)).

## Integration document (how to use it)

The Integration doc ([`docs/skai/integration.md`](docs/skai/integration.md)) is the **project-owned** place where `skai` workflows get the concrete, copy/pasteable details they need to run deterministically (build/test commands, destinations, artifact paths, evidence expectations).

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
- The default during install/update is **gated discussion**: when the installer needs project-specific values it can't infer, it stops, asks the human, and writes the canonical entry from the discussion. 🟡 + `INSTRUCTION:` stubs are reserved for explicit deferral or non-interactive `auto` runs.

## Project requirements (PRD)

`skai` workflows can be backed by a project-owned PRD (Product Requirements Document) that captures behavioral and contractual requirements separate from implementation. The PRD is the canonical answer to "what must this system do?" and is referenced (read-only) by work-spec implementation, debugging, and unit testing. Authoring happens in the work-spec-creation and dev-retro workflows.

The full conventions live in the canonical guide: [`Guides/Core/prd-guide.md`](Guides/Core/prd-guide.md). Quick orientation:

- **Scopes**: `platform`, `domains`, `features`, `apps/<app-name>` -- pick exactly one per requirement (placement decision tree in the guide).
- **Layout**: each folder has a same-named index file (e.g. `requirements/features/features.md`). Per-requirement files use stable IDs and behavioral, implementation-agnostic prose.
- **Glossary**: `requirements/glossary.md` defines domain terms used across requirements.

### Repository shape

Projects vary in how they distribute code across repos. The PRD adapts via four shapes (recorded in the integration doc's `Section: requirements`):

- **`single-repo`** -- one repo, all PRD content lives there.
- **`multi-repo-no-share`** -- multiple repos, each maintains its own self-contained PRD.
- **`hybrid`** -- multiple repos, with a shared repo (typically a submodule) owning cross-cutting scopes (`platform`, `domains`, sometimes `features`); per-repo PRDs own app-specific content.
- **`none`** -- the project does not maintain a PRD; normalization and backfill are skipped.

The shape is chosen during install/update; the installer scaffolds the appropriate layout (templates at [`Templates/requirements/`](Templates/requirements/)) and gates on a product / app description so the LLM has anchoring context. Cross-repo references in the `hybrid` shape use a `shared:` ID prefix (e.g. `shared:ORDER-04`).

### How the PRD gets populated

The PRD is built up over time by two skai workflows:

- **Forward (planning -> PRD)** -- during `skai-work-spec-creation`, behaviors discovered in planning are normalized into the PRD before tasks are written.
- **Backward (code -> PRD)** -- during `skai-dev-retro`, behaviors discovered while working with existing code are backfilled into the PRD.

Both follow the content rules in [`Guides/Core/prd-guide.md`](Guides/Core/prd-guide.md) and use the **gated-discussion** approach: the LLM stops to discuss with the human when content can't be inferred, then writes the canonical-format entry from the discussion. Silent 🟡 + `INSTRUCTION:` stubs are reserved as fallback (deferral or `auto` mode).

## IDE clutter / autocomplete (recommended)

To reduce duplicate autocomplete/search results (submodule sources + installed assets), hide the submodule in your editor UI while keeping the submodule [`README.md`](README.md) visible.

Example for Cursor/VS Code workspace settings (`.vscode/settings.json`):

```json
{
  "files.exclude": {
    "**/Submodules/skai/**": true,
    "**/Submodules/skai/README.md": false
  },
  "search.exclude": {
    "**/Submodules/skai/**": true,
    "**/Submodules/skai/README.md": false
  }
}
```

Android Studio (JetBrains):
- In the Project tool window, right-click `Submodules/skai` → **Mark Directory as** → **Excluded**.
- Optionally also exclude `.claude/skills/skai-*` or `.agents/skills/skai-*` if you don't want the skill install artifacts in search results.
- Prefer local IDE excludes over committing `.idea` changes unless your repo explicitly versions IDE config.

## Usage

After installation, workflows are available as **skills** that your agent activates automatically based on conversational context.

### How all workflows work

**Minimal command structure.** You don't need to memorize strict commands:

1. Start with context that activates a skill/workflow (e.g. "Write a work spec", "Start unit testing", "Start debugging").
2. Progress through steps using **advance intent** (e.g. "begin", "next", "continue", "go ahead").
3. At certain steps the agent will **stop and wait** at a checkpoint (gate). At checkpoints, the agent ends its output with exactly one of these lines:
   - `⏳ GATE: Next: <thing>. Say "next" or what to change.`
   - `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`
   - `🏁 Complete. Let me know if anything needs adjustment.` (not a gate -- workflow finished)
4. Saying "next" (or similar) at a checkpoint counts as approval to proceed.
5. Adding **"auto"** to advance intent tells the agent to proceed without stopping at checkpoints, unless a universal STOP condition applies (e.g. "next auto", "begin auto").
6. You can bound auto: **"auto to <milestone>"** means "proceed until you are about to begin the milestone, then stop" (e.g. "next auto to task 7").

**Agent note-taking and progress tracking.** The agent keeps structured notes so you can pick up where you left off:

- The agent uses **🟡** markers in working documents to flag TODOs, open questions, and items pending your approval (including "implemented but pending approval").
- 🟡 markers should be individually updatable (use stable identifiers like task numbers, section names, or explicit IDs on the same line).
- 🟡 markers are removed only when items are resolved and you give approval.

**Retros on demand.** At any point you can ask the agent to **"retro"** to check for gaps, update documents, backfill requirements, and reflect on process.

### Work spec creation (skill `skai-work-spec-creation`)

Structured planning and specification for complex features. Produces a planning document (design decisions, API sketch), normalized product requirements, and a work specification (tasks, subtasks, requirements traceability).

- Guide [`Guides/Spec/work-spec-creation.md`](Guides/Spec/work-spec-creation.md)

**Prerequisites:** Discuss the feature or problem with the agent in some detail before initiating the process -- scope, motivating use cases, architecture, solution approaches. A thorough upfront discussion produces a much higher quality first draft and minimizes iteration. A Jira ticket, feature request, or problem statement is a good starting point.

**Phases:**

1. **Planning document.** Agent summarizes the discussion into a document seeded with 🟡 open questions. Optional: for large efforts, the planning document can be organized into explicit phase sections; each phase runs its own mini-cycle (discussion, API sketch, requirements normalization, work spec) before moving to the next. Gate: human reviews and resolves 🟡 items before the workflow advances.
2. **Design discussion.** Agent proposes, human decides. Iterates until all 🟡 items are resolved.
3. **API sketch.** Agent drafts the API surfaces implied by the design. Gate: human confirms the design is ready to proceed.
4. **Requirements normalization.** Agent promotes behaviors from the planning document into the project's PRD per [`Guides/Core/prd-guide.md`](Guides/Core/prd-guide.md). Gate: human acknowledges requirements updates.
5. **Work spec first pass.** Agent writes top-level tasks only (no subtasks). Gate: human reviews the task list.
6. **Work spec second pass.** Agent adds subtasks, requirement IDs, and traceability mapping. Gate: human reviews the completed work spec.

### Work spec implementation (skill `skai-work-spec-implementation`)

Execute tasks from a completed work spec, one top-level task per cycle.

- Guide [`Guides/Spec/work-spec-implementation.md`](Guides/Spec/work-spec-implementation.md)

**Prerequisites:** A completed work spec.

**Phases (repeating):**

1. **Implement next top-level task.** Agent implements all subtasks under Task N. Gate: agent stops after finishing Task N and waits before moving to Task N+1.

### Unit testing (skill `skai-unit-testing`)

Plan-first testing workflow. The agent creates an orchestration document for the overall testing session, plans all tests upfront, runs one infrastructure pass across all planned tests, and then implements tests one logical section at a time (e.g. "Success Tests", "Error Handling Tests"). Handles new test suites, additions to existing suites, and fixing failing tests.

- Guides [`Guides/Test/unit-testing-guide.md`](Guides/Test/unit-testing-guide.md), [`Guides/Test/unit-test-planning-guide.md`](Guides/Test/unit-test-planning-guide.md), [`Guides/Test/unit-test-infrastructure-guide.md`](Guides/Test/unit-test-infrastructure-guide.md), [`Guides/Test/unit-test-writing-guide.md`](Guides/Test/unit-test-writing-guide.md)

**Prerequisites:** Source code to test. Often triggered by a work spec task, but can be used independently.

**Phases:**

1. **Planning.** Agent chooses a session name for the current testing effort and creates `working-docs/<branch-path>/<session-name>/testing/unit-testing.md` (following [`Guides/Core/working-doc-conventions.md`](Guides/Core/working-doc-conventions.md)), adds `Planning 🟡`, and creates test files organized into sections with test stubs in each. Doc comments on every stub serve as the test plan. At the planning gate, `Planning 🟡` remains until the human approves advancing to infrastructure.
2. **Infrastructure.** Agent identifies required test infrastructure across all planned tests in the testing session (stubs, fixtures, production code abstractions) and proposes additions. The orchestration document keeps `Infrastructure 🟡` until the human approves advancing to writing. Related infrastructure docs and artifacts live under the same `working-docs/<branch-path>/<session-name>/...` session folder.
3. **Writing** (per section, file-by-file). Agent implements tests and then runs them section-by-section, finishing the current file before moving to the next. Gates: agent stops after writing (before running tests), and stops after test results to confirm conclusions and next steps (including any proposed production-code fixes). The orchestration document keeps `Writing 🟡` until all sections in the testing session are approved complete. If a test requires infrastructure that wasn't identified in Phase 2, it is skipped, the missing infrastructure is documented, and the human can decide at the next planned gate whether to re-enter the infrastructure phase or defer that skipped work.

Phase 3 repeats for each section until none remain.

### Debugging (skill `skai-debugging`)

Evidence-first problem resolution. Prevents "guessing fixes" loops by requiring observable evidence before drawing conclusions. The process defines a toolkit of effective strategies (targeted logging, possibility-space partitioning, minimal reproducers, bisection, invariant assertions) that guide the agent's debugging approach.

- Guides [`Guides/Core/debugging-guide.md`](Guides/Core/debugging-guide.md), [`Policies/debugging-process-rule.md`](Policies/debugging-process-rule.md)

**Prerequisites:** A bug, crash, or unexpected behavior to investigate. Provide whatever evidence you have (error messages, logs, screenshots, steps to reproduce).

**Phases (repeating):**

1. **Hypothesize and experiment.** Agent states the current possibility space, chooses a tactic, and proposes the smallest discriminating experiment. Gates: pre-experiment and post-experiment during iteration, then hard gates for root cause, fix, and verify/close.

Repeats until the root cause is isolated.

### Dev retro (skill `skai-dev-retro`)

Completeness backstop that can be used at any point during any workflow. Reviews what has transpired since the last retro (or since session start), identifies gaps, reconciles plan drift, updates documentation, backfills requirements, and reflects on process.

- Guide [`Guides/Process/dev-retro.md`](Guides/Process/dev-retro.md)

**Prerequisites:** Work to review. The agent reads work specs, planning docs, evidence artifacts, and requirements produced since the last retro.

**Phases:**

1. **Retro.** Agent performs the full checklist, reports findings, and completes immediately if no process suggestions were generated.
2. **Process improvement follow-up (optional).** If the retro identifies process improvements, agent lists them in the retro output and stops at a handoff gate. On `next`, the agent enters [`Guides/Process/process-improvement.md`](Guides/Process/process-improvement.md), which drafts `process-tickets.md`, lets the human review/edit the resulting `## 🟡 Ticket: ...` entries, and files the remaining ones on `next`.

### Suggestion (skill `skai-suggestion`)

Ad-hoc process improvement suggestions outside of a retro. The agent helps the developer articulate an idea, problem, or feature request, captures it as a ticket draft, and optionally files it as a GitHub issue.

- Guide [`Guides/Process/process-improvement.md`](Guides/Process/process-improvement.md)

**Prerequisites:** None -- can be triggered at any point during a session.

**Phases:**

1. **Understand and draft.** Agent asks clarifying questions to understand the suggestion (skipped if the developer provides enough detail up front). Once it has enough context, it stops at a ready-to-draft gate. On `next`, it chooses a session name, writes one or more `## 🟡 Ticket: ...` drafts to `working-docs/<branch-path>/<session-name>/process-tickets.md`, and presents them for review.
2. **Review and file (optional).** The draft-review gate is the main filing gate: the human can revise or remove drafts there, or say `next` to file the remaining `## 🟡 Ticket: ...` entries as GitHub issues on `skai`. On filing, the draft marker is removed and the entry records the filed issue number.

### Update installation (skill `skai-update-installation`)

Check for upstream `skai` updates, review what changed, and re-run adapter runbooks.

- Guide [`Guides/Core/update-installation-guide.md`](Guides/Core/update-installation-guide.md)

**Prerequisites:** An existing installation ([`docs/skai/install-state.json`](docs/skai/install-state.json) must exist from the initial install).

**Phases:**

1. **Check and report.** Agent checks for upstream changes, updates the submodule as needed, and presents the changelog delta. Gate: human acknowledges before any runbooks are re-run.
2. **Re-run adapters.** Agent re-runs each installed adapter's install/update runbook to pick up new or changed assets.

### Working documents

- Working documents (plans, specs, progress logs) live under [`working-docs/`](working-docs/) and are organized by your current git branch.
- `<branch-path>` is the current branch name, with `/` decomposed into nested folders (so `feature/foo` becomes `feature/foo/`).
- Working docs are ephemeral and typically git-ignored.

## Development guide (for contributors)

This repo is typically maintained with an LLM. When you ask an LLM to make changes, point it at:
- [`README.md`](README.md) (this file)
- [`maintain-skai.md`](maintain-skai.md) (LLM maintainer runbook)
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
