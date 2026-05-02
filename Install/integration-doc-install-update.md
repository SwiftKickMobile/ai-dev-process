# Integration doc install/update guidance (shared)

This is a shared guide for installers that create/update the host project's Integration doc at `docs/skai/integration.md`.

## Goals

- The Integration doc is **project-owned operational glue**.
- It must be **agent-usable** and **deterministic** (prefer CLI commands; avoid GUI instructions).
- It must be safe to update on existing repos (don't delete/overwrite project-owned content).

## Gated discussion as default for human input

When the install runbook needs project-specific information that cannot be inferred from the host repo, the default is:

1. STOP at a gate.
2. Ask the human a focused question (or a small list of related questions).
3. Receive the answer.
4. Write the canonical-format entry into the relevant file. The human supplies source material; the agent writes the prose.

Reserve 🟡 + `INSTRUCTION:` stubs for cases where the human explicitly defers ("skip for now") or the runbook is in non-interactive `auto` mode. In `auto`, the runbook leaves the stubs and continues; the next non-auto run surfaces them as gates again.

This rule applies to:
- Integration doc fields in the `required-values` block
- Integration doc fields in the `Section: requirements` block
- PRD product / app descriptions during scaffolding (see `Install/prd-install-update.md`)

The seed templates encode the 🟡 + INSTRUCTION fallback shape. The runbook should prefer to fill them via discussion at install time rather than leaving stubs.

## Rules

- Prefer **non-interactive command-line** commands.
- Do not invent simulator/device models or other environment-specific values from memory.
- Use 🟡 only for true project-specific missing constants/mappings (and only as the fallback shape per the gated-discussion default above).
- Treat placeholders like `<Scheme>`, `<TestPlan>`, `<TestTarget>` as **variables** the agent fills per task context (not 🟡 TODOs).

## Managed blocks

The Integration doc contains:
- a human-owned "Special instructions / overrides" section (must be read and obeyed first)
- LLM-managed blocks delimited by:
  - `<!-- BEGIN Managed-By: skai | Section: ... -->`
  - `<!-- END Managed-By: skai | Section: ... -->`

Installers may only create/update/remove content inside managed blocks.

### `required-values` block (human-filled form)

The `required-values` block is a managed "form" that humans fill in:
- Humans should remove 🟡 markers and delete any `INSTRUCTION:` lines once they fill a value.
- Installers must preserve filled values and must only restore 🟡 + instruction lines when required information is missing.

Standard instruction format:
- `INSTRUCTION: <what is needed>`

## Required behavior

When creating/updating the Integration doc:

1. Seed from `Templates/docs/skai/integration.md` if missing.
2. Read and obey "Special instructions / overrides".
3. Insert/update the `Section: requirements` managed block from `Templates/docs/skai/integration-sections/requirements.md`. This block is always present (regardless of stack) and records the project's PRD shape and routing. Fill it via gated discussion per the rule above.
4. Detect which stacks apply (Xcode/Swift, Swift Package Manager, Android/Gradle, etc.) and then:
   - insert/update the corresponding managed blocks from:
     - `Templates/docs/skai/integration-sections/xcode.md`
     - `Templates/docs/skai/integration-sections/swift-package.md`
     - `Templates/docs/skai/integration-sections/android-gradle.md`
   - remove irrelevant managed blocks if the stack is not present (or the human says omit).
   - Note: a project may use both `xcode` and `swift-package` sections (e.g., app targets built via Xcode and library submodules built as Swift packages).
5. If required project-specific constants/mappings cannot be inferred:
   - Follow the gated-discussion default above: STOP, ask the human, then write the canonical entry.
   - If the human defers or the runbook is in `auto` mode, restore/leave 🟡 markers + INSTRUCTION lines in the relevant block.

Discovery guidance (avoid brittle filename checklists):
- When migrating "legacy integration glue" into the Integration doc, **search broadly** (README, `docs/`, CI config/scripts, prior agent/rule docs) for build/test commands, destinations, and artifact path conventions.
- Do not assume specific legacy filenames exist. If you use examples (like `xcode-commands.md`), treat them as examples only and report **what you found**, not a list of "not found" files.

## Legacy cleanup

On every install or update, review the existing Integration doc for content that has been superseded by canonical `skai` guides or templates. Content that was reasonable in an earlier version of skai may now duplicate or contradict a canonical source.

Examples:
- Working-doc conventions embedded inline (now owned by `Guides/Core/working-doc-conventions.md`)
- Process-flow rules or marker semantics (now owned by `Guides/Core/process-flow.md`)
- Legacy installer-oriented sections (e.g., "Installer setup tasks")

Propose removing any such content and wait for approval before deleting.

