# Temporary Discussion Document

Status: Historical discussion document. Items below may be implemented; treat checkboxes as the authoritative status.

Date: 2026-02-07

## Quick index

- [Session outline](#your-session-outline-verbatim-intent)
- [Inventory](#repository-inventory-what-exists-now)
- [Doc-by-doc understanding](#initial-understanding-what-each-document-is-for)
- [Portability findings](#initial-findings-project-specific-vs-ide-specific-vs-generally-reusable)
- [Project-supplied docs contract](#project-supplied-documents-integration-points-intentional-externalization)
- [Step 3: Improvements discussion](#step-3-improvements-discussion-notes)
- [Step 4: Install/update discussion](#discussion-decisions-we-should-align-on-these)
- [Meta: revert premature edits](#meta-changes-already-made-to-be-reverted)

---

## Your session outline (verbatim intent)

- [x] **1. Review all documents for understanding**
- [x] **2. Review all documents for project or IDE-specific content that needs to be refactored**
- [x] **3. General discussion: explore potential improvements to the documents**
- [x] **4. General discussion: discuss how the documents will be installed/updated on a project**
- [x] **5. Develop installation/update guide**
   - The guide is the LLM agent’s instructions for installation and update
   - This repo will be added as a Submodule
   - Installed by generating/placing managed files into appropriate locations (e.g. `.cursor/rules/ai-dev-process/`)
   - README is for developers; install should be “ask the LLM to install/update”

Additional constraints you stated during discussion:
- Some abstractions intentionally refer to **project-supplied documents** that contain project-specific details.
- Examples should be **project and IDE agnostic** (no project terms like “SalesPro”).
- Core docs should be **IDE-neutral**; IDE-specific installation guides can exist. Engineers will tell the LLM which IDE they’re using.

---

## Repository inventory (what exists now)

- `Core/`
  - `debugging-guide.md`
- `Policies/`
  - `coding-patterns.md`
  - `error-handling.md`
  - `unauthorized-changes.md`
  - `safe-operations.md`
  - `swift-code-organization.md` (stack-specific)
- `Install/`
  - `Cursor/install-update-cursor.md`
  - `AndroidStudio-ClaudeCode/install-update-androidstudio-claudecode.md`
  - `managed-header.md`
  - `conflict-precedence-policy.md`
- `Templates/`
  - `docs/ai-dev-process/integration.md`
- `Spec/`
  - `work-spec.md`
  - `work-spec-implementation.md`
  - `retro-prd.md`
- `Test/`
  - `unit-testing-guide.md`
  - `unit-test-planning-guide.md`
  - `unit-test-infrastructure-guide.md`
  - `unit-test-writing-guide.md`
- Root
  - `README.md`
  - `assets.manifest.json`
  - `CHANGELOG.md`

---

## Initial understanding (what each document is for)

### `Dev/` (legacy; migrated)

Originally these were Cursor `.mdc` files under `Dev/`. They have been migrated to IDE-neutral sources:
- Policies live under `Policies/`
- Guides live under `Core/`
- Cursor `.mdc` is generated into host projects during install/update

### `Spec/` (work specification and requirements normalization)

- `Spec/work-spec.md`
  - A “work specification” template/process for complex tasks:
    - Two-pass structure (high-level then detailed)
    - Inputs section for agent-sufficiency
    - Requirements inventory with IDs
    - Traceability (requirements ↔ tasks)
  - Assumes an org practice of a canonical requirements repository at `/requirements/**` (repo-structure-specific).

- `Spec/work-spec-implementation.md`
  - Execution protocol for implementing a work spec using 🟡 markers and “begin/next/continue” commands.

- `Spec/retro-prd.md`
  - A short instruction to retrofit “canonical requirements” into `/requirements/**` based on review/discussion.

### `Test/` (unit testing runbooks)

These documents define a staged workflow:
- plan tests (stubs and docs)
- identify/implement infrastructure
- write tests and run them
- checkpoint/advance with “next”

However, the examples and execution guidance currently assume:
- Swift Testing syntax
- Xcode build/test commands and `.xcresult`
- a DI container reset pattern

---

## Initial findings: project-specific vs IDE-specific vs generally reusable

### A) Project-specific items that should be removed from examples

Observed category:
- App/product names in examples (standardize on a fictional product: **LumenNotes**)
- Concrete app-specific protocol/type names
- Concrete vendor names where not essential (e.g. naming a specific error reporting service)

Desired direction:
- Replace with placeholders or neutral names:
  - Prefer consistent fictional names in examples:
    - `LumenNotesError`, `LumenNotesUIError`, `LumenNotesLogger`
    - “error reporter” / “telemetry sink” (generic service name)
    - `ErrorTaxonomy` examples use feature-oriented categories (no vendor)

### B) IDE-specific items (Cursor / `.mdc` / `.cursor` assumptions)

Observed category:
- `.mdc` rule files are Cursor-specific packaging.
- Some docs reference `.cursor/...` as the location of sibling guides.

Desired direction (per your constraint):
- Core docs should be IDE-neutral.
- Provide IDE-specific installation guides (Cursor, VS Code, etc.) that map “core docs” into the IDE’s expected locations.

### C) Language/toolchain-specific items (Swift/Xcode/testing frameworks)

Observed category:
- Stack-specific policies (e.g., `Policies/swift-code-organization.md`).
- Testing runbooks in `Test/*` reference Swift Testing, Xcode commands, `.xcresult`, and a specific DI pattern.

Two viable directions to discuss:
1. Keep these as “Swift/Xcode adapters” alongside the core testing process.
2. Extract a tool-agnostic “testing workflow backbone” and add language/tooling adapters for:
   - Swift/Xcode
   - JS/Jest
   - Python/pytest
   - etc. (as needed)

---

## “Project-supplied documents” integration points (intentional externalization)

These guides correctly want to avoid hard-coding project decisions. They should explicitly reference a *project-owned* “integration” doc (name TBD) for:

- **Test execution commands**
  - Example need: “how to run only these tests; where outputs live; how to capture logs; CI parity”
  - Today: some guides refer to `xcode-commands.md` and/or `.cursor/xcode-commands.md`

- **Dependency injection/testing conventions**
  - Example need: “how to register stubs/mocks; how to reset global state; serialization requirements”

- **Canonical requirements repository (if the org uses one)**
  - Example need: “where requirements live; naming/ID conventions; scope taxonomy”
  - If a project does not use `/requirements/**`, the “canonical requirements” step should be optional or replaced with the project’s equivalent.

Proposal for discussion:
- Create a single “Project Integration” doc contract that lists what each project must supply.
  - Then adapters (Cursor, etc.) link to that doc in the IDE’s preferred location.

---

## Improvements to discuss (no edits yet)

## Step 3: Improvements discussion notes

Your improvement categories:
1. Making assets more project/IDE agnostic
2. Critique against industry best practices: what can be improved / what’s missing

### 3.1 Making assets more project/IDE agnostic

#### Core vs Adapters split (explicit)

Goal: make it obvious what is portable everywhere vs what is Cursor/Swift/Xcode-specific.

Proposed structure direction (example):
- `Core/` (IDE-neutral, toolchain-neutral where possible)
- `Adapters/Cursor/` (Cursor `.mdc`, `.cursor` install mapping)
- `Adapters/SwiftXcode/` (Swift rules, Xcode test runner specifics)

#### Remove project-specific nouns from examples

Rule: examples must not include app/product names (e.g. “SalesPro”), or concrete project type names.

Use placeholders like:
- `ProjectError` / `DomainError` / `AppError`
- `UIError` / `UserFacingError`
- `ErrorReporter` / `TelemetrySink`
- `Logger`

Each guide that needs concrete names should list **Project Integration Points** (what the host project must supply).

#### Normalize cross-document references

Replace hard-coded references like:
- `.cursor/unit-test-*.md`
- `.cursor/xcode-commands.md`
- Cursor `@work-spec.md` include style

With IDE-neutral phrasing:
- “See your project’s Integration doc at: `<path>`”
- “See the adapter for your IDE/tooling”

#### Make language-specific assets opt-in

Rename and/or group assets so non-Swift projects don’t accidentally install Swift rules.

Examples:
- `swift-code-organization.mdc`
- `swift-coding-patterns.mdc`
- `swift-debugging-examples.md` (if examples remain language-specific)

### 3.2 Critique vs industry best practices (improve / missing)

#### Add a manifest to drive installs/updates

Problem: docs drift from installer logic without a canonical source of truth.

Proposed: introduce a small manifest (format TBD) listing:
- asset ID
- core vs adapter
- source path (in this repo)
- install targets by IDE (Cursor, etc.)
- optional tags (language/tooling)

This lets an agent do deterministic installs and safe updates.

#### Versioning + change communication

Decision: add this to **this repo** (not to host projects).

Add:
- A lightweight version (even if just date-based)
- A `CHANGELOG.md` (or release notes section) describing what changed

So agents can explain updates and users can audit changes.

#### Define conflict and precedence policy

Best practice: never overwrite user content silently.

Define rules for:
- existing non-symlink file at install destination
- existing rule with same filename
- project-customized overrides (e.g. `*.local` takes precedence)

#### Expand “safety posture” beyond unauthorized changes

You already have “unauthorized changes.” Consider a short “Safe Operations” policy covering:
- don’t commit/push unless asked
- don’t add dependencies unless asked
- don’t delete user files
- keep updates minimal and reversible
- never revert without explicit permission

#### Testing guidance: separate workflow backbone from tool adapters

Your testing workflow shape is strong (plan → infra → implement → run → triage).

Best practice: keep that workflow in Core, and move execution specifics into adapters:
- Swift/Xcode adapter (current content)
- (future) Jest, pytest, etc.

Additionally:
- The unit test failure workflow should explicitly reference the Debugging / Problem-Resolution guide for tactics selection, justification, and stop conditions (avoid “guessing fixes” during test failure triage).

### 3.3 Debugging / problem-resolution guide improvements (needs expansion + adapters)

Observation:
- The current “facts over speculation + logging” approach is valuable, but it’s only one tactic.
- Goal: give the agent a *toolbox* of tactics to prevent “guessing fixes” cycles.
- Debugging in real projects is typically **human-in-the-loop** (AI cannot operate the app); guide should define this collaboration contract.

Principle: pick a tactic, justify it, run it, reassess:
- Debugging should involve **selecting a tactic** and explicitly stating why it’s the best next step.
- If the tactic isn’t producing new evidence quickly, switch tactics (don’t keep guessing).

#### Debugging toolbox (core, IDE/tool-agnostic)

Each tactic should be usable without assuming a specific IDE. The guide should teach when to use each one.

- **Facts-first evidence collection**
  - Add minimal instrumentation at decision points to produce definitive evidence.
  - Output should help rule out possibilities, not just “print everything.”

- **Partition the possibility space (A/B/C → B1/B2/B3)**
  - Maintain a small set (2–4) of mutually exclusive (or explicitly overlapping) possibilities.
  - Run discriminating experiments to eliminate partitions until the root cause statement is isolated.

- **Minimal working implementation → move toward real until it breaks**
  - Start from a small implementation that works.
  - Change one dimension at a time toward the real implementation until the first change that introduces the issue is identified.

- **Binary search / bisection**
  - Use feature flags/config toggles to bisect within a runtime pipeline.
  - Use `git bisect` when the bug is a regression and the commit range is known.

- **Minimal harness**
  - Create a deterministic reproducer (tiny unit test, small sample app/screen, or standalone reproducer) that isolates the behavior.
  - Goal: “repro on demand” so experiments are cheap.

- **Invariants & assertions**
  - Identify what must be true; add temporary assertions/guards to fail fast at the first invalid state.

#### Human-in-the-loop protocol (core)

Make explicit that the human provides runtime observation:
- Human runs the app/tests and provides outputs (logs, screenshots, repro confirmation, stack traces).
- Agent proposes the smallest next experiment and explains:
  - what evidence it will produce
  - how that evidence will eliminate partitions / narrow the search

#### Stop conditions (core)

The guide should instruct the agent to stop and consult the human when:
- The next step requires product intent (expected behavior is ambiguous).
- Required evidence is unavailable (human hasn’t provided logs/output needed to proceed).
- The current tactic isn’t producing new discriminating evidence after a small number of iterations.
- The agent is about to propose a destructive or wide-reaching change.

#### Adapters (minimal; only where needed)

Keep adapters limited to “how to produce and capture evidence” for a given stack:
- Logging API conventions and filtering
- How the human should capture and share outputs with the agent
- Known tooling limitations that affect evidence (e.g., missing unit test log visibility)

Install-time behavior:
- Installer infers stack from repo, then asks the developer to confirm.
- Installed debugging guidance references the correct evidence-capture adapter for that stack.

---

## Discussion decisions (we should align on these)

1. What is the minimum “core” set that should be IDE-neutral?
   - Candidate: `Spec/` docs, `Policies/unauthorized-changes.md`, and `Core/debugging-guide.md` (minus stack-specific examples).

### Rule/guide classification (current decision)

These are the intended roles for the migrated “Dev” documents:

- **Swift-specific (adapter)**
  - `Policies/swift-code-organization.md`: Swift-only code organization policy

- **Core (portable)**
  - `Policies/coding-patterns.md`: core coding patterns (not Swift-specific in intent)
  - `Policies/error-handling.md`: core error handling principles
  - `Policies/unauthorized-changes.md`: core agent safety rule
  - `Policies/safe-operations.md`: core safety policy for installers/agents

- **Core guide with minimal adapters**
  - `Core/debugging-guide.md`: debugging/problem-resolution process (guide, not an auto-applied “rule”)
    - Adapters only where needed (currently mainly logging API + evidence capture constraints)

2. Do we want Cursor rule files in this repo at all?
   - Decision: No. This repo contains only IDE-neutral `.md` sources.
   - Cursor rule files (`.mdc`) are generated into the host project during installation/update under:
     - `.cursor/rules/ai-dev-process/*.mdc`
   - Update safety: installer only overwrites files it manages (identified via managed-by header).

3. Do we want to keep Swift/Xcode testing as the first adapter, but rewrite examples to avoid project terms?
   - Decision: Yes. Swift/Xcode (SwiftUI + Swift Testing) is the first supported adapter; examples should use the LumenNotes theme (no real project terms).

3.5. Add Android adapter (for Android Studio + Claude Code + Kotlin + Jetpack Compose)
   - Goal: Android devs can install/update this repo and get Android-appropriate guides and Claude Code instructions.
   - Adapter selection: installer infers Android stack from Gradle/Kotlin/Android project structure, then asks dev to confirm.
   - Claude Code instructions location:
     - Detect existing project convention first.
     - Observed conventions:
       - `agent-app-android`: has `claude.md` (lowercase) at repo root (no `.claude/` folder).
       - `salespro-android`: no Claude instruction file found.
     - Default if none exists: create/manage root `CLAUDE.md` (or follow org preference).
   - Android unit test stack (observed):
     - `agent-app-android`: JUnit4 + MockK + kotlinx-coroutines-test; AndroidX JUnit + Espresso for instrumentation.
     - `salespro-android`: JUnit4 + Robolectric + AndroidX test libs (no Kotest / MockK / Mockito observed).

4. What should the “project-supplied docs” contract look like?
   - Decision: One.
   - Note: today this information is often split across the project `README.md` (human-facing) and a commands doc (e.g. `xcode-commands.md`). Installer should migrate/normalize those details into the single Integration doc over time.

---

## Step 4: Install/update discussion notes

### 4.1 Core vs adapters (confirmed direction)

- Core assets are IDE-neutral plain Markdown (`.md`) in this repository.
- IDE-specific installation/update guides exist (Cursor first).
- Cursor installation/update is hybrid:
  - LLM-runbook orchestrates detection, migration, and conflict handling.
  - Mechanical `.md` → `.mdc` conversion is performed deterministically (no scripts).
  - Updates follow deterministic overwrite rules for managed files only.

### 4.2 Managed file safety (don’t touch project-specific files)

Constraints:
- No filename prefixes (filenames must remain simple and prompt-friendly).
- Project may contain additional project-specific Cursor rules in `.cursor/rules/`.

Decisions:
- Install all managed Cursor rule files under a dedicated subfolder:
  - `.cursor/rules/ai-dev-process/*.mdc`
- Assume Cursor loads rules from subfolders under `.cursor/rules/`.
  - If this assumption is false in practice, revise the layout later.
- Every managed file written into the host project must include a “managed by” header so updates are safe.
  - Update rule: only overwrite files that are either (a) missing, or (b) already contain the managed header.
  - Migration note: legacy copies that look like managed assets but do NOT contain the managed header are **legacy candidates**.
    - Do NOT overwrite legacy candidates by default.
    - Instead, migrate by generating new managed files into the new locations, then optionally propose cleanup of legacy candidates as a separate, explicit step.
  - Never overwrite project-owned files lacking the header.

### 4.3 Cursor adapter behavior (no scripts)

- Cursor adapter writes/updates `.mdc` files directly into the host project (generated from core `.md` sources).
- Core `.md` remains the canonical source of truth; generated `.mdc` is treated as cache/installation artifact.

### 4.4 Migration responsibility (LLM-driven install/update)

When installing into an existing project, the installer must assume relevant project-specific guidance may already exist (often scattered across existing docs).

Installer responsibilities:
- Discover existing project docs that contain integration details (examples):
  - Project `README.md` (or docs folder)
  - `xcode-commands.md` (or equivalents)
  - Existing `.cursor/*` docs/rules
- Extract relevant “integration” information (test/build commands, simulator/device selection, CI parity, output capture conventions, etc.).
- Ensure the information is normalized into the project’s Integration document (the project-owned source of truth).
- Update new installs to reference the Integration document (not the old scattered docs), without breaking the project:
  - Prefer leaving legacy docs in place (do not delete) unless explicitly asked.
  - If references would break, add compatibility pointers (e.g., “moved to Integration doc at <path>”).

### 4.5 Migration-capable update algorithm (recognize legacy installs and migrate)

Goal: deploy this submodule onto an existing project that may already have copies of many files installed (some obsolete). The installer must recognize the existing install and migrate to the current system safely.

#### Phase 1: Discovery (read-only; no changes)

Inventory the host repo for:
- Submodule state:
  - Is `ai-dev-process` already a submodule? where? what commit?
- Existing install artifacts:
  - Cursor rules and related docs under `.cursor/rules/**` and `.cursor/**`
  - docs under `docs/**` (or other common documentation folders)
  - root instruction files (e.g. `README.md`, `claude.md`, `CLAUDE.md`)
- Legacy coupling signals:
  - references to `xcode-commands.md`
  - references to old guide paths (e.g. `.cursor/unit-test-...`, Cursor `@work-spec.md` includes)
  - duplicated guides/rules in multiple locations

#### Phase 2: Classification (decide what is safe to touch)

Classify each discovered file into one of:
- **Managed (safe to overwrite)**: contains the `Managed-By: ai-dev-process` header.
- **Legacy candidate (do not overwrite yet)**: appears to be an older copy of a managed asset but lacks the header.
  - Identification should use: filename + strong content fingerprint (not just filename).
- **Project-owned (hands off)**: custom project content or unrelated rules/docs.

Safety rules:
- Never overwrite project-owned files.
- Never overwrite legacy candidates without an explicit human-approved migration plan.

#### Phase 3: Plan the migration (write the plan; do not execute yet)

Produce an explicit migration plan that maps current state → target state:
- Target end-state (current system):
  - Single project-owned Integration doc (source of truth for project-specific commands/conventions).
    - Default location: `docs/ai-dev-process/integration.md` (installer may detect and reuse an existing integration doc elsewhere).
    - Legacy `xcode-commands.md` content is migrated into the Integration doc.
  - Cursor `.mdc` rules generated into `.cursor/rules/ai-dev-process/`.
  - All generated/managed files include managed-by header + `Source:` pointer back to submodule `.md`.
- File operations list:
  - Files to create (new managed outputs)
  - Managed files to update in place
  - Legacy candidates to supersede (create new managed equivalents first; leave legacy in place by default)
- Content migration:
  - Extract relevant integration info from existing scattered docs (e.g. `README.md`, `xcode-commands.md`).
  - Normalize into the Integration doc.
  - Add compatibility pointers in legacy docs if needed (do not delete unless asked).

#### Process docs should refer to rules (not README)

Rule: process docs should reference required rules/policies by name (e.g., “Error Handling rule”) rather than pointing at `README.md` sections.

Installer behavior:
- If a required rule doc does not exist in the host project, prompt the human to specify the source of truth for that rule (often the project `README.md`).
- The installer/agent then transfers the policy content into the rule doc (managed) and updates process docs to refer to the rule.
- If the project `README.md` later changes, the human prompts the agent to update the rule doc accordingly.

#### Phase 4: Confirmation gate (human approval required)

Before any writes:
- Present a concise summary of planned file changes (create/update/supersede).
- Call out any risky operations explicitly (renames, deletions, touching non-managed files).
- Wait for human confirmation (“begin/next/continue”) before executing.

#### Phase 5: Execute safely (ordered, reversible)

Recommended order:
1. Create or update the Integration doc and migrate/normalize content into it.
2. Generate/update Cursor rules into `.cursor/rules/ai-dev-process/` with managed headers.
3. Update references in newly generated artifacts to point to the Integration doc.
4. Optionally propose cleanup of legacy copies as a separate, explicit step (default: leave them).

### 4.6 Integration doc contract (Xcode/iOS projects)

The Integration doc contains only operational glue needed for agents to execute process docs deterministically (not general policy like error handling).

Required sections (recommended):
- **How to run tests**
  - exact `xcodebuild` commands and common variants
  - `-only-testing` patterns
  - how to produce `.xcresult` + where to write output artifacts
- **Build/compile commands**
  - minimal commands to compile key targets/schemes
- **Simulator/device selection**
  - preferred simulator(s) + OS version(s)
  - how to list/select destinations
  - any required boot/permissions setup
- **Project structure mapping**
  - main app target(s)
  - test targets
  - module/package layout notes relevant to test placement and imports
- **Environment/setup prerequisites**
  - Xcode version
  - Swift toolchain constraints
  - required secrets/config (what exists, where it comes from; no secret values)
  - required local services (if any)
- **Output/log collection contract (human-in-the-loop)**
  - what the human should paste back (output file path, `.xcresult` path, crash logs if needed)
  - known limitations affecting evidence collection
- **Conventions assumed by process docs**
  - where work docs go (`work/<branch>/...` and `work/<branch>/testing/...`)
  - repo-specific naming requirements (scheme/target names)

---

## Meta: changes already made (to be reverted)

During the session, changes were made before discussion checkpoints. You requested reverting those changes.

Status update:
- `AGENT-INSTALL-UPDATE.md`: removed (obsolete draft)
- `Dev/*.mdc`: removed from this repo (Cursor `.mdc` is now generated into host projects)
- `README.md`: updated to reflect the agreed system

