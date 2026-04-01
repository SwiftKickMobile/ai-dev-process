---
name: skai-integration-doc
description: Defines how installers create/update the host Integration doc (managed blocks, placeholder usage, stack sections merge/remove, and discovery scanning). Use when updating the Integration template/sections or when debugging installer-generated integration docs.
---
<!-- Managed-By: skai | Managed-Id: skill.skai-integration-doc | Managed-Source: .cursor/skills/skai-integration-doc/SKILL.md | Managed-Adapter: claude-code | Managed-Updated-At: 2026-04-01 -->

# SKAI: Integration doc (installer behavior)

## Instructions

- Read `Install/integration-doc-install-update.md` and follow it as the source of truth.
- When editing templates, also read:
  - `Templates/docs/skai/integration.md`
  - `Templates/docs/skai/integration-sections/xcode.md`
  - `Templates/docs/skai/integration-sections/swift-package.md`
  - `Templates/docs/skai/integration-sections/android-gradle.md`
- Keep 🟡 only for missing constants/mappings; treat `<...>` placeholders as agent-filled variables.
