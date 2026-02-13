# JetBrains + Claude Code adapter: install/update (LLM runbook)

Purpose: install/update `ai-dev-process` into a repo where developers use a JetBrains IDE (e.g., IntelliJ IDEA) and Claude Code.

Notes:
- This adapter is **stack-aware** (Swift/Xcode, Android/Kotlin, etc.). Infer from the repo and then confirm with the developer.
- The Integration doc remains project-owned at `docs/ai-dev-process/integration.md`.

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
- If this is an update, record the current submodule commit SHA (pre-update) and the intended new SHA (post-update).
- Infer stack (signals: `.xcodeproj`, `.xcworkspace`, `Package.swift`, `build.gradle`, `settings.gradle`, `*.swift`, `*.kt`).
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
  - `Guides/`, `Policies/`, `Templates/`, `Install/`, `assets.manifest.json`, `README.md`
- If you cannot compute the diff yourself, STOP and ask the human to provide the diff output.

### 5) Execute (safe order)

1. Ensure submodule is present/updated.
2. Create/update `docs/ai-dev-process/integration.md` (migrate legacy command docs into it; do not delete legacy docs by default).
   - If you cannot find the required integration information in-repo:
     - Create/seed the Integration doc from `Templates/docs/ai-dev-process/integration.md`.
     - Fill only what you can source with high confidence.
     - Add explicit 🟡 placeholders for missing items.
     - STOP and ask the human for the missing items before proceeding.
   - Prefer non-interactive command-line commands over GUI instructions. If you can’t produce command-line commands with high confidence, leave 🟡 placeholders and ask.
   - Follow `Install/integration-doc-install-update.md` for how to update the Integration doc safely (managed blocks + human overrides).
3. Create/update the Claude instruction file (`claude.md` vs `CLAUDE.md`) using managed headers.
4. Symlink repo-owned guides into `.claude/agent/ai-dev-process/` for convenient prompting.
5. Create/update ignore files (permission-gated if they already exist and are project-owned):
   - Update `.claudeignore` by inserting/updating a managed block:
     - Exclude `.cursor/**` so Claude sessions don’t ingest Cursor-specific assets by default.
     - Do NOT exclude `Submodules/ai-dev-process/**` here; use editor UI excludes for autocomplete/search clutter instead.
6. Optionally propose cleanup of legacy candidates as a separate explicit step.

Required Integration doc fields to request (minimum set):
- Build/compile command(s)
- Unit test command(s): run all + run a single test/subset
- How to capture full output (paths/artifacts the human should paste back)
- Device/simulator/emulator conventions (if applicable)
- Known evidence-capture limitations (if any)

