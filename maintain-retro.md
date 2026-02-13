# Maintenance retro checklist (LLM + human)

Use this after making changes to `ai-dev-process` to ensure nothing was forgotten.

## 1) Inventory: what changed?

- List the files you changed/added/removed.
- State whether the change is user-visible (affects host repo installs) or internal only.

## 2) Required bookkeeping

- Update `CHANGELOG.md` (Unreleased) to reflect major changes.
- Update `assets.manifest.json` if you added/moved/renamed any installable asset or installer dependency (templates/sections/runbooks/policies).
- Update `README.md`:
  - Quick start prompts if any runbook paths changed
  - Asset inventory (human-facing guides at file level)
  - Any new conventions/invariants developers need to know
- Update `maintain-ai-dev-process.md` if you changed invariants or introduced a new maintenance rule.

## 3) Integration doc architecture checks (common misses)

- If you changed Integration behavior:
  - `Templates/docs/ai-dev-process/integration.md` still has a human-owned “Special instructions / overrides” section.
  - Stack-specific templates exist under `Templates/docs/ai-dev-process/integration-sections/`.
  - Runbooks still describe how to merge/update/remove the managed blocks.
  - 🟡 is used only for true project-specific constants/mappings (not for variables or standard procedures).

## 4) Installer safety checks

- Runbooks still enforce:
  - discover → classify → plan → confirm → execute
  - permission-gated deletion/replacement of legacy candidates
  - managed headers for generated files
  - managed symlinks by validated target path
  - managed blocks for project-owned files like ignore files / Integration doc

## 5) Consistency checks

- Search for stale paths (e.g. old install target directories) and update all occurrences.
- Ensure new files referenced by runbooks exist at those paths.

## 6) Final report

- Provide a short summary of what changed and why.
- List touched files.

