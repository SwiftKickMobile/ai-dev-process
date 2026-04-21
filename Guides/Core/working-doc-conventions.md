Managed-By: skai
Managed-Id: guide.working-doc-conventions
Managed-Source: Guides/Core/working-doc-conventions.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Working Document Conventions

Working documents (planning docs, work specs, test work docs, test artifacts) are ephemeral files that track in-progress work. They live under a single top-level folder, are organized by git branch, and are grouped into named session folders so multiple efforts on the same branch do not collide.

## Location

All working documents live under `working-docs/` at the project root.

Within `working-docs/`, the current git branch name is used to create a subdirectory path. If the branch name contains `/`, each segment becomes a nested folder.

Inside the branch path, each workflow session must choose a required `session-name` folder. This session folder distinguishes separate efforts on the same branch and contains all primary documents and related artifacts for that effort.

**Path formula:** `working-docs/<branch-path>/<session-name>/<subpath>/<filename>`

Where:
- `<branch-path>` = the current git branch name, with `/` decomposed into nested directories
- `<session-name>` = a human-distinguishable name for the current effort on that branch
- `<subpath>` = workflow-specific subfolder (may be empty)
- `<filename>` = the document name (specified by the workflow guide)

Notes:
- The `session-name` is required for workflows that create working docs.
- Use a concise, stable name that helps distinguish different sessions on the same branch.
- Workflow guides may use a more specific term such as `spec-name`, but it still occupies the required `session-name` path segment.

**Examples:**

| Branch | Branch path |
|---|---|
| `work/step-refactor` + `observable-wrapper` | `working-docs/work/step-refactor/observable-wrapper/` |
| `work/foo` + `retro-cleanup` | `working-docs/work/foo/retro-cleanup/` |
| `feature/auth-flow` + `login-tests` | `working-docs/feature/auth-flow/login-tests/` |
| `main` + `ticket-maintenance` | `working-docs/main/ticket-maintenance/` |

## Creating a working document

1. Determine the current git branch:
   ```bash
   git branch --show-current
   ```
2. Choose a `session-name` for this effort.
3. Construct the full path using the formula above (the calling guide specifies `<subpath>` and `<filename>`).
4. Create the directory structure:
   ```bash
   mkdir -p working-docs/<branch-path>/<session-name>/<subpath>
   ```
5. Write the document.

## Examples

- Work spec planning:
  - `working-docs/<branch-path>/observable-wrapper/observable-wrapper-plan.md`
- Work spec implementation:
  - `working-docs/<branch-path>/observable-wrapper/observable-wrapper-impl.md`
- Unit testing orchestration:
  - `working-docs/<branch-path>/login-tests/testing/unit-testing.md`
- Unit test suite infrastructure:
  - `working-docs/<branch-path>/login-tests/testing/LoginViewModelTests/infrastructure.md`
- Process improvement drafts:
  - `working-docs/<branch-path>/retro-cleanup/process-tickets.md`
