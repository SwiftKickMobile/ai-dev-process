Managed-By: skai
Managed-Id: guide.prd
Managed-Source: Guides/Core/prd-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-05-03

# PRD Guide

## Purpose

This guide is the canonical reference for **Product Requirements Documents** (PRD) in `skai`-using projects. It defines what a PRD is, where it lives, how it is structured, how individual requirements are written, and how its content is authored and maintained over time.

The PRD is a **living library of behavioral and contractual requirements** for a system. Its primary audience is the **LLM**: when implementing, debugging, or testing, agents consult the PRD to know what behavior the system must exhibit. Humans read it too, but the primary consumer is the agent.

This guide is referenced by:
- `Guides/Spec/work-spec-creation.md` -- forward normalization (planning to PRD)
- `Guides/Process/dev-retro.md` -- backfill (discovered behavior to PRD)
- Install runbooks (`Install/Cursor/...`, `Install/ClaudeCode/...`, `Install/Codex/...`) -- install-time scaffolding and migration

Those workflows own the runtime mechanics (gates, advance intent). This guide owns the canonical content rules.

### What belongs in the PRD vs. the work spec

- **PRD (this guide)**: behavioral / contractual requirements. What the system must do as observed from the outside. Implementation-agnostic.
- **Work spec**: technical / transitional requirements (e.g. `MIG-01`, `TEMP-02`). Migration steps, refactor decisions, intermediate constructs. These do not belong in the PRD.

A behavior that survives a future rewrite belongs in the PRD. A constraint that exists only for the duration of a specific change belongs in the work spec.

## Scopes

PRD content is organized by **scope**. Each requirement lives in exactly one scope.

- **`platform`** -- system-wide invariants that hold true regardless of which app or feature uses them. Protocol-level, infrastructure-level, format-level rules.
  - Example: "All persisted documents must round-trip losslessly through the canonical serialization format."

- **`domains`** -- business / domain rules about *what things are*. Entities, their invariants, valid state transitions, relationships. Noun-level.
  - Example: "An Order in `pending` may transition to `confirmed` or `cancelled`; it may not skip directly to `shipped`."

- **`features`** -- user-facing capabilities reusable across multiple apps. Verb-level -- things a user can *do*. Distinguished from domains in that they describe user-visible behavior, not business invariants.
  - Example: "Users can mark items as favorites; favorited items appear in a dedicated list sorted by most recent."

- **`apps`** -- behavior specific to a single app and not shared with others.
  - Example: "In the CMS, the home screen shows pending review queues for the current user's role."

### Placement decision tree

For each requirement, choose exactly one scope:

1. If the rule applies regardless of any app or feature -> `platform`
2. Else if it defines what an entity is or how it behaves as a domain object -> `domains`
3. Else if multiple apps offer it as a user-facing capability -> `features`
4. Else -> `apps/<app-name>`

### Edge case: feature vs. domain

The `domains` / `features` line is the most commonly confused. A useful test: if the rule still applies when there is no UI, it is a domain rule; if it only matters because a user is interacting with it, it is a feature.

In practice, complex behaviors often need both:
- a **domain entry** that defines the data model and invariants
- a **feature entry** that defines the user-visible behavior

When this happens, cross-link the two (see "Per-requirement file shape" below).

### Prohibited structures

- Do **not** organize requirements by Xcode project, Gradle module, package, target, or framework.
- Do **not** create per-team or per-codebase folders.
- Scope is by behavioral domain, not by implementation grouping.

## Repository shape

Projects vary in how they distribute code across repositories. The PRD must adapt to that. Four shapes are supported:

- **`single-repo`** -- one repo, all PRD content lives at `requirements/` in that repo.
- **`multi-repo-no-share`** -- multiple repos, each maintains its own self-contained PRD. No cross-repo references.
- **`hybrid`** -- multiple repos, with a **shared** repo (typically pulled in as a submodule) that owns cross-cutting scopes (`platform`, `domains`, sometimes `features`), and per-repo PRDs that own app-specific content (`apps`, sometimes `features`).
- **`none`** -- the project does not maintain a PRD. Normalization and backfill are skipped by the calling workflow (with a one-line trace).

The chosen shape and scope routing for the current repo are recorded in the host project's Integration doc (`docs/skai/integration.md`) under the `requirements` managed section. Workflows reading or updating the PRD must read that section first to know where requirements live.

### Choosing a shape

- A single repo with a single team -> `single-repo`.
- Multiple repos, loosely coupled, no shared concepts -> `multi-repo-no-share`.
- Multiple repos with genuine shared concepts (cross-repo `Order` definition, shared platform contracts) -> `hybrid`.
- Small project that does not benefit from a PRD -> `none`.

### Hybrid shape: scope routing

In the hybrid shape, the Integration doc declares which scopes are local and which are shared. Typical routing:
- `platform` -> shared
- `domains` -> shared
- `features` -> per-project decision (shared if features are genuinely cross-app; local otherwise)
- `apps` -> local

The shared submodule is a separate repo with its own PRD layout. App repos do not create local `platform/` or `domains/` folders if those scopes are routed to shared.

## Layout & file conventions

PRD files follow a consistent, predictable layout so an agent can navigate them without reading each one.

### Hierarchy

For a repo that hosts PRD content, the layout under the local requirements root is:

```
requirements/
  _requirements.md              <- root index (system overview + map of scopes)
  glossary.md                   <- domain terms shared across requirements
  platform/
    _platform.md                <- scope index (catalog of platform requirement files)
    <topic>.md                  <- one or more requirement files
  domains/
    _domains.md                 <- scope index
    <entity>.md                 <- typically one file per domain entity
  features/
    _features.md                <- scope index
    <feature>.md                <- one file per feature
  apps/
    _apps.md                    <- scope index (catalog of apps)
    <app-name>/
      _<app-name>.md            <- per-app index
      <flow>.md                 <- one or more requirement files for that app
```

A scope-level file (e.g. `features/widgets.md`) may grow large enough to become a folder (e.g. `features/widgets/`). When it does, the new folder gets its own index file `widgets/_widgets.md` and the original content is split across files inside the folder.

### Naming convention: `<folder>/_<folder>.md`

Every folder that contains an index file uses the convention `<folder>/_<folder>.md`. The index file shares its name with its containing folder, prefixed with `_` (underscore).

Why:
- The leading `_` sorts the index file above the leaf requirement files in most editors' file explorers (Cursor / VS Code, etc.), so humans can find it immediately rather than hunting for it among alphabetically-sorted leaves.
- No `README.md` collisions in editor tabs (every tab has a meaningful name).
- The path itself documents what the file is.
- Easy to reference from prose ("see `features/_features.md`").

### Root index (`requirements/_requirements.md`)

Contains:
- A short paragraph describing the product / system at a high level.
- A brief restatement of the scopes present in this repo and what each means. (The canonical scope definitions live in this guide; the root index restates them briefly so an LLM landing here has immediate context.)
- A pointer to the glossary.
- For hybrid repos: a note that some scopes live in a shared submodule and a pointer to that path (sourced from the Integration doc).

The product description is gathered at install time via gated discussion (see "Authoring via gated discussion" below).

### Scope indexes (`<scope>/_<scope>.md`)

Contains:
- A one-line reminder of what this scope is. Example: `Features = user-facing capabilities reusable across apps; see Guides/Core/prd-guide.md for the full taxonomy.`
- A catalog: a list of the requirement files in this scope.

Each catalog entry uses this shape:

```markdown
- [<filename>](<filename>.md) -- <one-line purpose statement>
```

Optional fields per catalog entry, included only when relevant:
- Cross-references: `(uses: shared:ORDER-04, FAVORITES-02)` if this file depends on requirements elsewhere
- Status: `(deprecated)` if the requirement is being phased out

The catalog grows during normalization and backfill. New entries are added as new requirement files are created.

### Glossary (`requirements/glossary.md`)

A single alphabetical list of domain terms used across requirements. Each entry is one short paragraph.

What goes here:
- Domain entities and concepts (`Order`, `User`, `Session`, `Workspace`, etc.)
- Status names that recur across requirements (e.g. "Pending", "Confirmed")
- Project-specific jargon

What does *not* go here:
- Scope definitions (`platform`, `domains`, etc.) -- those live in this guide and are restated briefly in the root index.
- General software engineering terms.

The glossary starts empty and grows organically. There is no install-time prompt to populate it.

### Per-requirement file shape (soft template)

Each requirement file (e.g. `features/favorites.md`, `domains/order.md`) follows this shape:

```markdown
# <Title>

<One-paragraph purpose: what this is, in 1-3 sentences.>

## Requirements

- **<ID>**: <Behavioral statement.>
- **<ID>**: <Behavioral statement.>

## Cross-references

- <link to related requirement, prefixed with `shared:` if cross-repo>
```

The `## Cross-references` section is optional and may be omitted if there are none.

No status field, no implementation notes, no progress markers, no `🟡` markers. The PRD is a stable contract; transient state belongs in working docs.

## Writing style

PRD content is written as if authored by a **product manager with no knowledge of the codebase**.

### Required style

- Focus on **user-visible behavior**, **domain invariants**, and **system contracts**.
- Describe **what must be true**, not how it is achieved.
- Every requirement should be verifiable from the outside (a user, QA, or another system) without reading code.
- Use `must` / `should` / `will` voice -- not `implement` / `add` / `refactor`.

### Forbidden details

PRD content must not include:
- Specific data structures, algorithms, or execution strategies (e.g. "use caching", "use a queue", "debounce", "run in background task")
- Storage mechanisms (e.g. "persist to disk as JSON", "CoreData", "SQLite")
- Concurrency / threading / actors / async design (e.g. "use async/await", "MainActor", "perform off the main thread")
- Concrete code identifiers, file paths, or code formatting (backticked types, filenames, method names, initializer signatures)
- Frameworks or third-party tooling

If a detail is important but inherently technical, it belongs in the work spec under "Work-spec requirements (technical / transitional)" (e.g. `MIG-01`, `TEMP-02`), not in the PRD.

### Self-check before writing

Before adding or updating a requirement, verify:
1. Could a non-engineer read this without losing meaning?
2. Does it mention any code identifier, file, module, dependency, or framework? If yes, rewrite.
3. Is it phrased as a behavior / contract (`must` / `should` / `will`) rather than a plan (`implement` / `add` / `refactor`)?

### Examples

- Good: "The system must detect and report circular references in templated documents."
- Bad: "The `AssetCatalog` should DFS templates and throw `CircularReferenceError`."

- Good: "Users must be able to view all validation issues for an asset in a single report."
- Bad: "Accumulate errors during parsing and return an aggregated error array."

## ID conventions

Each requirement has a stable ID used to reference it from work specs and from other requirements.

### Format

- IDs are uppercase, with a scope-specific prefix and a numeric suffix.
- Examples: `ORDER-04`, `FAVORITES-02`, `DOC-PROMPT-04`.
- Prefixes are project-defined. There is no global registry; the goal is local readability and stability.

### Stability

- IDs do not change when files are moved, renamed, or reorganized.
- When a requirement is deprecated, leave the ID in place with a `(deprecated)` note in the catalog. Do not reuse the ID.

### Cross-repo references (hybrid shape)

When referencing a requirement that lives in a different repo (typically the shared submodule):

- Local references (within the current repo) use the bare ID: `ORDER-04`.
- Cross-repo references use the `shared:` prefix: `shared:ORDER-04`.

The owning side is bare; the referencing side prefixes. So `domains/order.md` in the shared repo writes `**ORDER-04**: ...` (bare), but `apps/mobile/checkout.md` in an app repo writes `(uses: shared:ORDER-04)` (prefixed). The same ID appears bare or prefixed depending on which repo you are reading from.

The `shared:` prefix only indicates "this lives outside the current repo". It does not encode the actual path; the Integration doc's `requirements` section provides the path mapping.

## Authoring via gated discussion

When PRD content needs human input that the LLM cannot infer (product description, app description, requirement prose, naming, scope decisions), the calling workflow stops at a gate, discusses with the human, and then writes the entry in the canonical format described in this guide. The human supplies source material; the LLM writes the prose.

This applies during install scaffolding, work-spec normalization, and retro backfill. Each calling workflow owns its own gate mechanics; this guide only defines the principle.

`🟡` + `INSTRUCTION:` stubs are reserved for cases where the human explicitly defers ("skip for now") or where the workflow is running in non-interactive `auto` mode. They are not the default.

## Maintenance touchpoints

Three workflows interact with the PRD:

- **Install scaffolding** -- adapter install runbooks (`Install/Cursor/install-update-cursor.md`, `Install/ClaudeCode/install-update-claudecode.md`, `Install/Codex/install-update-codex.md`) create the local PRD layout on first install, gather the product / app descriptions via gated discussion, and detect migrations from previous shapes when re-run.
- **Forward normalization** -- `Guides/Spec/work-spec-creation.md` (Requirements Normalization step) promotes behaviors from a planning doc into the PRD before tasks are written.
- **Backfill** -- `Guides/Process/dev-retro.md` (Product requirements backfill step) reconciles the PRD against behavior discovered during a session.

The `update-installation` workflow (`Guides/Core/update-installation-guide.md`) re-runs install scaffolding when `skai` itself is updated; this re-surfaces any unfilled `🟡` stubs as gates.

The PRD is **read-only** from work-spec implementation, debugging, and unit testing. Those workflows consume requirements but never mutate them.

## Cross-references

- `Guides/Spec/work-spec-creation.md` -- requirements normalization step
- `Guides/Process/dev-retro.md` -- requirements backfill step
- `Guides/Core/working-doc-conventions.md` -- shared file / path conventions
- `Templates/docs/skai/integration-sections/requirements.md` -- the integration-doc managed section that records this project's PRD shape and routing
- `Install/integration-doc-install-update.md` -- canonical guidance for install runbooks updating the integration doc
