# PRD install/update guidance (shared)

This is a shared guide for installers that scaffold and migrate the host project's PRD (Product Requirements Document). See `Guides/Core/prd-guide.md` for canonical PRD content rules.

## Goals

- Establish the PRD layout in the host repo per the shape recorded in the integration doc's `Section: requirements`.
- Gather project-specific descriptions (product, apps) via gated discussion -- not via silent 🟡 stubs.
- Detect and propose migrations from earlier PRD layouts.
- Be safe to re-run on existing repos: never clobber human-authored content.

## Read first

- `Guides/Core/prd-guide.md` -- canonical PRD content rules, layout conventions, ID conventions, writing style.
- `Templates/docs/skai/integration-sections/requirements.md` -- the shape config that lives in the integration doc.
- `Install/integration-doc-install-update.md` -- the gated-discussion default for human input.

## Required behavior

### 1) Read the shape

Read the integration doc (`docs/skai/integration.md`), `Section: requirements`. The shape determines what (if anything) the install runbook scaffolds:

- `single-repo` -- scaffold the local PRD layout under the configured local root.
- `multi-repo-no-share` -- scaffold the local PRD layout under the configured local root.
- `hybrid` -- scaffold only the local-routed scopes; do not create folders for shared-routed scopes.
- `none` -- do not scaffold; emit a one-line trace ("PRD: none -- skipping scaffolding") and proceed.

If the shape value is missing, follow the gated-discussion default in `Install/integration-doc-install-update.md` to ask the human which shape applies. Write the chosen shape into the integration doc's Requirements section before proceeding.

### 2) Migration detection (during discover)

Before scaffolding, inspect the host repo for existing PRD content that may need migration:

- An existing `requirements/` folder without the `requirements.md` root index -- propose adopting the new layout (create indexes, scope folders, glossary; reorganize loose files into the appropriate scope).
- Existing PRD-like content under other paths (e.g. `docs/requirements/`, `prd/`, `specs/requirements/`) -- propose moving it under the configured local root.
- Existing requirement files at the root of `requirements/` (not under a scope folder) -- propose moving them into the appropriate scope.
- Loose files that look like requirement entries but lack stable IDs -- flag and propose normalizing (assign IDs).

Migrations are permission-gated. Present the migration plan (which files move where, what gets created, what IDs get assigned) at the Confirm gate and STOP for human approval before executing.

When updating an existing install (re-run): if all the seeded files already exist with the expected names, treat the layout as already established and skip scaffolding for those files. Re-surface any remaining 🟡 markers in those files as gates for the human to fill (per the gated-discussion default).

### 3) Scaffold the local PRD layout

Seed templates live at `Submodules/skai/Templates/requirements/`. Copy them into the host repo's local requirements root, omitting scopes routed to `shared` in the hybrid shape.

For `single-repo` and `multi-repo-no-share`, copy:

- `requirements/requirements.md` (root index)
- `requirements/glossary.md`
- `requirements/platform/platform.md`
- `requirements/domains/domains.md`
- `requirements/features/features.md`
- `requirements/apps/apps.md`

For `hybrid`, copy only the files whose scope is routed to `local` per the integration doc's scope routing.

Do not overwrite a destination that already exists. PRD seeds are project-owned content after creation; subsequent edits are made by workflows (work-spec normalization, retro backfill), not by the installer.

### 4) Gather descriptions via gated discussion

For each piece of content the seed templates leave as 🟡 (product description, per-app descriptions), follow the gated-discussion default:

- STOP, ask the human a focused question about that content.
- Write the canonical-format entry in the host file from the discussion. (The human supplies source material; the agent writes the prose.)
- If the human defers ("skip for now") or the runbook is in non-interactive `auto` mode, leave the 🟡 markers and INSTRUCTION lines in place; the next non-auto run surfaces them again.

Specific gates the install runbook drives (skip a gate if the corresponding seed file was not scaffolded):

- **Product description** -- one paragraph at the top of `<local-root>/requirements.md`. Gate prompt: "What does this product / system do? Who uses it? What problem does it solve?"
- **Per-app description(s)** -- for each app in the project, one line at the top of `<local-root>/apps/<app>/<app>.md`. If the list of apps is not obvious, ask the human first ("What apps does this repo contain?"), then gate per app for descriptions.

For each per-app file, copy `Submodules/skai/Templates/requirements/apps/per-app-stub.md` into `<local-root>/apps/<app-name>/<app-name>.md` (rename to match the app's name), then gate on the description.

After collecting descriptions, update the `<local-root>/apps/apps.md` catalog with one entry per app, using the catalog format defined in `Guides/Core/prd-guide.md`:

```
- [<app-name>](<app-name>/<app-name>.md) -- <one-line description>
```

This keeps the apps scope index in sync with the per-app index files.

### 5) Update the host README

If the host repo has a `README.md` and the shape is not `none`, inject or update a managed `Section: requirements` block using the template at `Templates/docs/skai/readme-requirements-block.md`. Substitute placeholders with values from the integration doc's Requirements section:

- `{shape}` -- the chosen shape
- `{local-root}` -- the local requirements root path
- `{shared-root}` -- the shared root path (omit the line for non-hybrid shapes)

The README block follows the standard managed-block convention:

```
<!-- BEGIN Managed-By: skai | Section: requirements -->
... content ...
<!-- END Managed-By: skai | Section: requirements -->
```

If the README does not exist, do not create one; gate to ask the human whether to create it (this is a project-level decision).

### 6) Reporting

After execution, list:

- Files scaffolded (paths)
- Migrations performed (if any)
- 🟡 markers remaining (paths + brief description, so the human knows what is still unfilled)

This makes any deferred items explicit and gives the human a chance to fill them at the next prompt.
