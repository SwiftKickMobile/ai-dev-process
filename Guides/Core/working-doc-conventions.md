Managed-By: ai-dev-process
Managed-Id: guide.working-doc-conventions
Managed-Source: Guides/Core/working-doc-conventions.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-19

# Working Document Conventions

Working documents (planning docs, work specs, test work docs, test artifacts) are ephemeral files that track in-progress work. They live under a single top-level folder and are organized by git branch.

## Location

All working documents live under `working-docs/` at the project root.

Within `working-docs/`, the current git branch name is used to create a subdirectory path. If the branch name contains `/`, each segment becomes a nested folder.

**Path formula:** `working-docs/<branch-path>/<subpath>/<filename>`

Where:
- `<branch-path>` = the current git branch name, with `/` decomposed into nested directories
- `<subpath>` = workflow-specific subfolder (may be empty)
- `<filename>` = the document name (specified by the workflow guide)

**Examples:**

| Branch | Branch path |
|---|---|
| `work/step-refactor` | `working-docs/work/step-refactor/` |
| `work/foo` | `working-docs/work/foo/` |
| `feature/auth-flow` | `working-docs/feature/auth-flow/` |
| `main` | `working-docs/main/` |

## Creating a working document

1. Determine the current git branch:
   ```bash
   git branch --show-current
   ```
2. Construct the full path using the formula above (the calling guide specifies `<subpath>` and `<filename>`).
3. Create the directory structure:
   ```bash
   mkdir -p working-docs/<branch-path>/<subpath>
   ```
4. Write the document.
