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
> - Use `docs/ai-dev-process/integration.md` as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

#### JetBrains (IntelliJ IDEA) + Claude Code prompt

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/JetBrains-ClaudeCode/install-update-jetbrains-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

#### Xcode (MCP-hosted) + Claude Code prompt (EXPERIMENTAL)

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/Xcode-ClaudeCode/install-update-xcode-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.
> - This runbook is experimental: if any Xcode+MCP assumption doesn't match this repo's setup, STOP and ask me what convention to use.

#### Android Studio + Claude Code prompt (Android stack)

> Install/update `ai-dev-process` in this repo by following `Submodules/ai-dev-process/Install/AndroidStudio-ClaudeCode/install-update-androidstudio-claudecode.md`.
>
> - If the `Submodules/ai-dev-process` submodule is missing, add it there.
> - Do a discovery pass first, then propose a migration plan, then WAIT for approval before writing.
> - Use `docs/ai-dev-process/integration.md` as the project-owned Integration doc and migrate any legacy build/test command notes into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

## How installs stay safe

- **Integration doc (project-owned)**: `docs/ai-dev-process/integration.md` is the single source of truth for project-specific commands/paths (build/test/lint/etc). Templates live in `Templates/`.
- **Managed files**: host-project files written by the installer have a required header (see `Install/managed-header.md`). The installer overwrites only files that already contain this header.
- **Legacy installs**: lookalike files without the header are treated as **legacy candidates** and are not overwritten by default (see `Install/conflict-precedence-policy.md`).

## Integration document (how to use it)

The Integration doc (`docs/ai-dev-process/integration.md`) is the **project-owned** place where `ai-dev-process` workflows get the concrete, copy/pasteable details they need to run deterministically (build/test commands, destinations, artifact paths, evidence expectations).

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

To reduce duplicate autocomplete/search results (submodule sources + installed assets), hide the submodule in your editor UI while keeping the submodule `README.md` visible.

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

## Asset inventory

The authoritative list of assets and their intended usage is in `assets.manifest.json`.

### Human-facing guides (file-level)

- `Guides/Core/debugging-guide.md`
  - Debugging / problem-resolution tactics, evidence-first loop, and stop conditions (human-in-the-loop).

- `Guides/Core/working-doc-conventions.md`
  - Conventions for creating working documents (planning docs, work specs, test work docs): all live under `working-docs/`, organized by git branch path. Referenced by workflow guides.

- `Guides/Spec/work-spec-creation.md`
  - Work-spec workflow in four phases:
    - Planning (three stages: scope discussion → ideation/questions/discussion with 🟡 marker protocol → API sketch)
    - Requirements normalization (promote behaviors from planning into `/requirements/**`)
    - Work spec first pass (high-level tasks for review)
    - Work spec second pass (detailed subtasks after approval)

- `Guides/Spec/work-spec-implementation.md`
  - How an agent executes a work spec using 🟡 markers and `begin/next/continue` checkpoints.

- `Guides/dev-retro.md`
  - End-of-session dev retro checklist: self-review, gaps, plan drift reconciliation, doc updates, retro requirements backfill, and process reflection.

- `Guides/update-installation-guide.md`
  - Procedure for updating the `ai-dev-process` submodule: check for upstream changes, pull, report changelog deltas to the human, and re-run installed adapter runbooks. Reads from `docs/ai-dev-process/install-state.json`.

- `Guides/Test/unit-testing-guide.md`
  - Orchestrator for the unit testing workflow; delegates to the three sub-guides.

- `Guides/Test/unit-test-planning-guide.md`
  - Plan-first process: create complete test stubs + doc comments across all sections before implementation.

- `Guides/Test/unit-test-infrastructure-guide.md`
  - Identify/propose required test infrastructure (stubs/fixtures/utilities), then implement with human approval.

- `Guides/Test/unit-test-writing-guide.md`
  - Write, run, and iterate on tests; capture evidence; debug failures using the core debugging guide.

### Human-facing templates (copied into host repos)

- `Templates/docs/ai-dev-process/integration.md`
  - Template for the project-owned Integration doc (`docs/ai-dev-process/integration.md`).

### Reference (mostly for the installer/LLM, not humans day-to-day)

- `Install/`
  - IDE-specific install/update runbooks and policies (managed header, precedence rules).
- `Policies/`
  - Agent behavior and coding policies (some stack-specific, e.g. Swift code organization).

## Development guide (for contributors)

This repo is typically maintained with an LLM. When you ask an LLM to make changes, point it at:
- `README.md` (this file)
- `maintain-ai-dev-process.md` (LLM maintainer runbook)
- `maintain-retro.md` (post-change checklist to ensure nothing was forgotten)

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
  - `assets.manifest.json`
  - `README.md` Asset inventory (developer-facing guides should be documented at file level)
- Update `CHANGELOG.md` for user-visible changes.
- Do not introduce scripts that mutate host repos; installers are LLM-runbook-driven and must follow the safety policies in `Policies/`.

