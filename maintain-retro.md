# Maintenance retro checklist (LLM + human)

Use this after making changes to `ai-dev-process` to ensure nothing was forgotten.

## Retro intent (read this first)

This retro is a **backstop for completeness**, not a git/diff report.

- Do **not** do a detailed “what changed” writeup unless explicitly asked.
- Prefer scanning docs/templates/runbooks for **consistency and missing updates**.
- The expected output is either:
  - a short list of **misses / follow-ups**, or
  - **“Retro complete; no misses found.”**

## 1) Inventory (high level only)

- In 1–2 bullets: what you changed (e.g., “Integration template format”, “runbook behavior”, “new asset added”).
- State whether it’s **host-install-visible** (affects host repo installs) or internal only.

## 2) Required bookkeeping

- Ensure `CHANGELOG.md` reflects major user-visible changes (section naming/versioning is fine either way).
- Ensure `assets.manifest.json` is updated if you added/moved/renamed any installable asset or installer dependency (templates/sections/runbooks/policies).
- Ensure `README.md` is updated if needed:
  - Quick start prompts if any runbook paths changed
  - Asset inventory (human-facing guides at file level)
  - Any new conventions/invariants developers need to know
- Ensure `maintain-ai-dev-process.md` is updated if you changed invariants or introduced a new maintenance rule.

## 3) Integration doc architecture checks (common misses)

- If you changed Integration behavior:
  - `Templates/docs/ai-dev-process/integration.md` still has a human-owned “Special instructions / overrides” section.
  - Stack-specific templates exist under `Templates/docs/ai-dev-process/integration-sections/`.
  - Runbooks still describe how to merge/update/remove the managed blocks.
  - 🟡 is used only for true project-specific constants/mappings (not for variables or standard procedures).
  - Integration section templates are **integration-doc-ready** content (no 🟡 TODO lists; no installer/human instruction prose; include copy/pasteable CLI templates with `<...>` variables).

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

## 6) Retro output (keep it short)

- If you found misses: list them as **actionable follow-ups** (1–6 bullets).
- If you found none: say **“Retro complete; no misses found.”**

