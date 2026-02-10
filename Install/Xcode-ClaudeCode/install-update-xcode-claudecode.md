# Xcode (MCP-hosted) + Claude Code adapter: install/update (EXPERIMENTAL)

Purpose: install/update `ai-dev-process` into a repo where developers use **Xcode (26.3+) MCP-hosted Claude Code**.

This runbook is **experimental** because the exact Xcode+MCP conventions may differ by environment.

## Assumptions (stop if false)

Assume the following, unless the host repo already establishes a different working convention:

- Claude instructions file is rooted at either `CLAUDE.md` or `claude.md` (prefer existing; default to `CLAUDE.md`).
- Claude ignore file is `.claudeignore`.
- Agent-facing process docs live at `.claude/agent/ai-dev-process/` (symlinks to repo-owned sources).
- Mixed-agent repos may also contain `.cursor/**`; Claude sessions should ignore Cursor-specific assets by default.

If any assumption is false in the host repo’s setup, STOP and ask the human what file/path/convention to use.

## Inputs (required reading)

- `assets.manifest.json`
- `Install/managed-header.md`
- `Install/conflict-precedence-policy.md`
- `Policies/safe-operations.md`
- `Templates/docs/ai-dev-process/integration.md`

## Migration-capable algorithm (required)

Follow the discover → classify → plan → confirm → execute workflow.

### 1) Discover (read-only)

- Identify whether `ai-dev-process` is already present as a submodule (and where).
- Infer stack (signals: `.xcodeproj`, `.xcworkspace`, `Package.swift`, `*.swift`).
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Inventory existing install artifacts:
  - Claude instruction files (`claude.md`, `CLAUDE.md`, `.claude/**`)
  - `.cursor/**` (may exist in mixed-agent repos; treat as separate installation)
  - `docs/**`
- Identify any existing docs containing integration details (to migrate into Integration doc).

### 2) Classify

Classify each discovered artifact:
- **Managed**: has the managed header (`Managed-By: ai-dev-process`) → safe to overwrite.
- **Managed symlink**: a symlink that points into `Submodules/ai-dev-process/...` at the expected target path → safe to replace/update.
- **Legacy candidate**: looks like a managed asset but lacks the header → do not overwrite.
- **Project-owned**: custom → do not overwrite.

### 3) Plan (no changes yet)

Prepare a concrete plan:
- Files to create
- Files to update (managed only, including managed symlinks)
- Legacy candidates to supersede (create new canonical outputs)
- Integration doc migration items
- Legacy cleanup proposals (permission-gated)

### 4) Confirm (human gate)

Present the plan and wait for human approval before writing.

If updating the submodule, include an “update review” section:
- Summarize changes between the old SHA and new SHA for relevant paths:
  - `Core/`, `Policies/`, `Spec/`, `Test/`, `Templates/`, `Install/`, `assets.manifest.json`, `README.md`
- If you cannot compute the diff yourself, STOP and ask the human to provide the diff output.

### 5) Execute (safe order)

1. Ensure submodule is present/updated.
2. Create/update `docs/ai-dev-process/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
   - If you cannot find the required integration information in-repo:
     - Create/seed the Integration doc from `Templates/docs/ai-dev-process/integration.md`.
     - Fill only what you can source with high confidence.
     - Add explicit 🟡 placeholders for missing items.
     - STOP and ask the human for the missing items before proceeding.
   - Prefer non-interactive command-line commands (e.g., `xcodebuild ...`) over GUI instructions (“open Xcode…”). If you can’t produce command-line commands with high confidence, leave 🟡 placeholders and ask.
3. Create/update the Claude instruction file (`claude.md` vs `CLAUDE.md`) using managed headers.
4. Symlink repo-owned guides into `.claude/agent/ai-dev-process/` for convenient prompting.
5. Create/update ignore files (permission-gated if they already exist and are project-owned):
   - Update `.claudeignore` by inserting/updating a managed block:
     - Exclude `.cursor/**` so Claude sessions don’t ingest Cursor-specific assets by default.
     - Exclude `Submodules/ai-dev-process/**` to reduce clutter, but un-exclude:
       - `Submodules/ai-dev-process/README.md`
       - `Submodules/ai-dev-process/Install/**`
       - `Submodules/ai-dev-process/assets.manifest.json`
6. Optionally propose cleanup of legacy candidates as a separate explicit step.

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Device/simulator destination conventions (if applicable)
- Known evidence-capture limitations (if any)

