# Integration doc install/update guidance (shared)

This is a shared guide for installers that create/update the host project's Integration doc at `docs/ai-dev-process/integration.md`.

## Goals

- The Integration doc is **project-owned operational glue**.
- It must be **agent-usable** and **deterministic** (prefer CLI commands; avoid GUI instructions).
- It must be safe to update on existing repos (don't delete/overwrite project-owned content).

## Rules

- Prefer **non-interactive command-line** commands.
- Do not invent simulator/device models or other environment-specific values from memory.
- Use 🟡 only for true project-specific missing constants/mappings.
- Treat placeholders like `<Scheme>`, `<TestPlan>`, `<TestTarget>` as **variables** the agent fills per task context (not 🟡 TODOs).

## Managed blocks

The Integration doc contains:
- a human-owned "Special instructions / overrides" section (must be read and obeyed first)
- LLM-managed blocks delimited by:
  - `<!-- BEGIN Managed-By: ai-dev-process | Section: ... -->`
  - `<!-- END Managed-By: ai-dev-process | Section: ... -->`

Installers may only create/update/remove content inside managed blocks.

### `required-values` block (human-filled form)

The `required-values` block is a managed "form" that humans fill in:
- Humans should remove 🟡 markers and delete any `INSTRUCTION:` lines once they fill a value.
- Installers must preserve filled values and must only restore 🟡 + instruction lines when required information is missing.

Standard instruction format:
- `INSTRUCTION: <what is needed>`

## Required behavior

When creating/updating the Integration doc:

1. Seed from `Templates/docs/ai-dev-process/integration.md` if missing.
2. Read and obey "Special instructions / overrides".
3. Detect which stacks apply (Xcode/Swift, Swift Package Manager, Android/Gradle, etc.) and then:
   - insert/update the corresponding managed blocks from:
     - `Templates/docs/ai-dev-process/integration-sections/xcode.md`
     - `Templates/docs/ai-dev-process/integration-sections/swift-package.md`
     - `Templates/docs/ai-dev-process/integration-sections/android-gradle.md`
   - remove irrelevant managed blocks if the stack is not present (or the human says omit).
   - Note: a project may use both `xcode` and `swift-package` sections (e.g., app targets built via Xcode and library submodules built as Swift packages).
4. If required project-specific constants/mappings cannot be inferred:
   - restore/leave 🟡 markers + instruction lines in the `required-values` block
   - STOP and ask the human the minimum questions needed

Discovery guidance (avoid brittle filename checklists):
- When migrating "legacy integration glue" into the Integration doc, **search broadly** (README, `docs/`, CI config/scripts, prior agent/rule docs) for build/test commands, destinations, and artifact path conventions.
- Do not assume specific legacy filenames exist. If you use examples (like `xcode-commands.md`), treat them as examples only and report **what you found**, not a list of "not found" files.

## Legacy cleanup (recommended, permission-gated)

If the Integration doc contains legacy installer-oriented sections (e.g., "Installer setup tasks"), propose removing them (with explicit approval).

