# `ai-dev-process`

Reusable, mostly IDE-agnostic **agentic coding guides** and **policies** you can vendor into any repo.

This repo is designed to be installed as a **git submodule** and activated by an **LLM-runbook-driven installer** that:
- inspects the host repo (including legacy installs),
- proposes a migration plan,
- writes **managed files** only (safe updates),
- generates IDE-specific artifacts (e.g., Cursor `.mdc`) into the host repo.

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
> - Use `docs/ai-dev-process/integration.md` as the project-owned Integration doc and migrate any legacy `xcode-commands.md` content into it (do not delete legacy files unless I explicitly approve).
> - Only overwrite files that contain the managed header (`Managed-By: ai-dev-process`). Treat lookalike files without the header as legacy candidates.

#### Android Studio + Claude Code prompt

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

## Asset inventory

The authoritative list of assets and their intended usage is in `assets.manifest.json`.

### Human-facing guides (file-level)

- `Core/debugging-guide.md`
  - Debugging / problem-resolution tactics, evidence-first loop, and stop conditions (human-in-the-loop).

- `Spec/work-spec.md`
  - Work-spec workflow in three phases:
    - Planning (clarify scope, assumptions, inputs, open questions, and **API Sketches**)
    - PRD creation/update (normalize externally observable behavior into canonical requirements, when applicable)
    - Work spec creation (two-pass: high-level tasks for review, then detailed subtasks)

- `Spec/work-spec-implementation.md`
  - How an agent executes a work spec using 🟡 markers and `begin/next/continue` checkpoints.

- `Spec/retro-prd.md`
  - How to retrofit/normalize externally observable behavior into `/requirements/**` (when your org uses a canonical requirements repo).

- `Test/unit-testing-guide.md`
  - Orchestrator for the unit testing workflow; delegates to the three sub-guides.

- `Test/unit-test-planning-guide.md`
  - Plan-first process: create complete test stubs + doc comments across all sections before implementation.

- `Test/unit-test-infrastructure-guide.md`
  - Identify/propose required test infrastructure (stubs/fixtures/utilities), then implement with human approval.

- `Test/unit-test-writing-guide.md`
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

