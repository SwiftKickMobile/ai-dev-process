# Safe Operations (Agent Policy)

This policy applies to agents executing the `ai-dev-process` workflows, including install/update runbooks.

## Rules

- Do not change code without explicit authorization when in discussion/troubleshooting mode.
- Do not commit unless explicitly asked.
- Do not push unless explicitly asked.
- Do not delete project files unless explicitly asked.
- Do not add dependencies unless explicitly asked.
- Do not revert changes unless explicitly asked.
- Prefer minimal, reversible changes.
- Before overwriting anything, determine whether the file is managed vs project-owned.
  - Managed files are identified by the managed header.
  - Project-owned files must not be overwritten.

