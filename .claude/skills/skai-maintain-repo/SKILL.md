---
name: skai-maintain-repo
description: Maintains the skai repository itself (invariants, templates, runbooks, manifest, and changelog sync). Use whenever making multi-file or non-trivial changes to skai, especially under Guides/, Templates/, Install/, Policies/, assets.manifest.json, README.md, or CHANGELOG.md. Always run the maintenance retro after significant edits.
---
<!-- Managed-By: skai | Managed-Id: skill.skai-maintain-repo | Managed-Source: .cursor/skills/skai-maintain-repo/SKILL.md | Managed-Adapter: claude-code | Managed-Updated-At: 2026-04-01 -->

# SKAI: Maintain this repo

## Instructions

- Read `maintain-skai.md` and follow it as the source of truth.
- If you touch Integration templates or behavior, also read `Install/integration-doc-install-update.md`.
- After any significant edit set (multi-file change, path moves, installer behavior changes, new assets/skills, integration template changes), run the checklist in `maintain-retro.md` and report only misses/follow-ups.
