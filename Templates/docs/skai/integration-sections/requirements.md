## PRD / Requirements

Records how this project's PRD (Product Requirements Document) is organized. See `Submodules/skai/Guides/Core/prd-guide.md` for the canonical guide.

- 🟡 **PRD shape**: {human response}
  - INSTRUCTION: One of `single-repo`, `multi-repo-no-share`, `hybrid`, or `none`. See the canonical PRD guide for definitions.

- 🟡 **Local requirements root**: {human response}
  - INSTRUCTION: Path to the local PRD root, typically `requirements/`. Set to `n/a` if shape is `none` or this repo holds no local PRD.

- 🟡 **Shared requirements root**: {human response}
  - INSTRUCTION: For `hybrid` shape only: path to the shared PRD submodule root (e.g. `Submodules/shared-prd/requirements/`). Set to `n/a` for non-hybrid shapes.

- 🟡 **Scope routing**: {human response}
  - INSTRUCTION: For `hybrid` shape only: declare which scopes are local vs shared. Format (one per line):
    - `platform: <local|shared>`
    - `domains: <local|shared>`
    - `features: <local|shared>`
    - `apps: <local|shared>`
  - Set to `n/a` for non-hybrid shapes.

- **Glossary location**: `<local-requirements-root>/glossary.md`
