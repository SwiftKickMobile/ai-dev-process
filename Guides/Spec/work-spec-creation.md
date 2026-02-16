Managed-By: ai-dev-process
Managed-Id: guide.work-spec
Managed-Source: Guides/Spec/work-spec-creation.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-08

# Work Specification Guide

## Purpose

Orchestrates the work specification creation process. Work specifications provide structured documentation for complex coding tasks that require multiple steps, coordination between components, and careful planning. They serve as both implementation roadmaps and progress tracking tools.

**Key Principles:**
- **Agent-sufficient document**: A fresh agent should be able to implement by reading this spec *and* following the explicitly listed inputs (referenced docs/files/links). No implicit context.
- **Communication Tool**: Describes what code to write, not how to write it
- **No Code**: Contains no actual code - only descriptions of what needs to be implemented

**Two-Pass Process:**
1. **First Pass**: Write high-level tasks only (no subtasks) for review
2. **Second Pass**: Add detailed subtasks after approval

---

## Canonical Requirements (PRD) Normalization Step

Before creating a work specification, product/system behavior must be normalized into the canonical requirements repository.

**Process (lightweight):**
1. Review the planning document.
2. For each product or system behavior:
   - If it already exists in the requirements repository → reuse its ID.
   - If it is new or changes existing behavior → add or update a requirement entry in `/requirements/**`.
3. Do not create tasks, subtasks, progress markers, or implementation decisions in the requirements repository.

**Rules:**
- The requirements repository contains behavioral / contractual requirements only.
- Requirements must be written as **implementation-agnostic black-box behavior**.
- It must not name concrete types, functions, files, initializers, modules, targets, or third-party libraries/frameworks.
- It must not contain 🟡 / TODO / pending markers.
- Git history is the source of change/audit information.
- Requirements must be placed into the correct scope folder as defined in
  “Requirements Repository Organization”.

### Canonical requirements writing style (anti-implementation guidance)

Write canonical requirements as if authored by a **product manager with no knowledge of the codebase**:

- Focus on **user-visible behavior**, **domain invariants**, and **system contracts**.
- Describe **what must be true**, not how it is achieved.
- Every requirement should be verifiable from the outside (a user, QA, or another system), without reading code.

**Do not include implementation/technical details such as:**

- Specific data structures, algorithms, or execution strategies (e.g., “use caching”, “use a queue”, “debounce”, “run in background task”)
- Storage mechanisms (e.g., “persist to disk as JSON”, “CoreData”, “SQLite”, “FileStorage”)
- Concurrency / threading / actors / async design (e.g., “use async/await”, “MainActor”, “perform off the main thread”)
- Concrete Swift identifiers, file paths, or code formatting (backticked types, `.swift` filenames, method names, initializer signatures)
- Tooling and patterns (dependency injection frameworks, logging frameworks, testing frameworks)

**If a detail is important but inherently technical:**

- Put it in the **work spec** under “Work-spec requirements (technical / transitional)” (e.g. `MIG-01`, `TEMP-02`) instead of the canonical requirements repo.

#### Quick self-check (before writing to `/requirements/**`)

- Can this be understood by a non-engineer without loss of meaning?
- Does it mention *any* code identifier, file, module, dependency, or framework? If yes → rewrite.
- Is it phrased as a behavior/contract (“must/should/will”) rather than a plan (“implement/add/refactor”)?

#### Examples

- ✅ “The system must detect and report circular references in templated documents.”
- ❌ “The `AssetCatalog` should DFS templates and throw `CircularReferenceError`.”

- ✅ “Users must be able to view all validation issues for an asset in a single report.”
- ❌ “Accumulate errors during parsing and return an aggregated error array.”

The work specification references canonical requirement IDs produced by this step.

---

## Requirements Repository Organization

All canonical requirements MUST be written into `/requirements/**` using the following scope rules.

Exactly one scope must be chosen for each requirement.

### Scopes

- `/requirements/platform`
  System-wide and cross-app behavioral contracts.
  (e.g. document formats, templating rules, identity rules, rendering semantics)

- `/requirements/domains`
  Business / domain rules shared across apps and tools.
  (e.g. entities, invariants, validation rules, relationships, state transitions)

- `/requirements/features`
  Reusable, user-facing features shared across multiple consumer apps.
  (e.g. search, favorites, offline, entitlements, content browsing)

- `/requirements/apps`
  App-specific behavior and flows.
  (e.g. consumer app only rules, CMS-only behavior, app-specific integrations)

### Placement rule

When promoting requirements from planning:

1. If the behavior applies to all apps → use `platform`
2. Else if it defines domain meaning or rules → use `domains`
3. Else if it is a reusable end-user feature across consumer apps → use `features`
4. Else → use `apps/<app-name>`

### Prohibited structures

- Do NOT organize requirements by Xcode project.
- Do NOT organize requirements by module or package.
- Do NOT create per-target or per-framework requirement folders.

## Commands

### Next Command

**Definition:** Any of `"begin"`, `"next"`, or `"continue"` — these are synonymous.

**Behavior:** Context determines the action:
- If waiting to proceed → execute next step
- If stopped due to ambiguities or unexpected challenges → resume where you left off

**First Step:**
- Creates work specification document
- Includes: Title, Motivation, Functional Requirements, Requirements Inventory, Non-goals / Deferred, Relevant Files, Task List
- Task list contains only main tasks (numbered 1, 2, 3, etc.)
- No subtasks included
- No Traceability section (deferred to second step because the human may restructure the task list)
- Stops when high-level structure is complete
- Allows human to review overall sequence before details

**Second Step:**
- Adds detailed subtasks to each main task
- Subtasks numbered (1.1, 1.2, 2.1, 2.2, etc.)
- All tasks marked with 🟡 indicator (TODO)
- Adds the Traceability section (requirement ↔ task mapping) now that the task list is finalized
- Provides implementation-ready detail
- Completes the work specification

---

## File Naming Convention

Work specification and planning files follow this naming pattern:

**Planning Document Format:** `<branch-name>/[spec-name]-planning.md`

**Work Spec Document Format:** `<branch-name>/[spec-name]-work-spec.md`

**Examples:**
- Planning: `work/step-refactor/observable-wrapper-planning.md`
- Work Spec: `work/step-refactor/observable-wrapper-work-spec.md`

Where:
- `<branch-name>` = The git branch you're working on (e.g., `work/step-refactor`)
- `[spec-name]` = The specification name you provide when creating the document (e.g., `observable-wrapper`)
- Suffix = `-planning` for planning documents, `-work-spec` for work specifications

**Note:** `<branch-name>` includes the `work/` prefix (e.g., `work/feature-branch`).

**Branch name folders:** If the branch name contains `/`, it is decomposed into folders.
 - Example branch: `work/foo`
 - Example planning doc path: `work/foo/example-planning.md`
 - Example work spec path: `work/foo/example-work-spec.md`

**When creating documents:**
- For planning: Provide the spec name, and the document will be created at `<branch-name>/[spec-name]-planning.md`
- For work spec: Provide the spec name, and the document will be created at `<branch-name>/[spec-name]-work-spec.md`

## Structure

### 1. Title
Use a clear, descriptive title that captures the main objective:
- ✅ "Crosshairs-Directed Tractor Beam System"
- ❌ "Update Tractor Beam"

### 1.5 Inputs (Required Reading)
At the top of the work spec, list every document/file a fresh agent must read to execute the spec without guessing.

- Planning doc(s) (paths)
- Related work specs (paths), if any
- Key implementation files/folders (paths) for existing conventions
- Any external docs (links), if truly required

### 2. Motivation Section
Explain **why** this work is needed:
- Current limitations or problems
- Desired end state and benefits
- How the change improves user experience or system architecture

### 3. Functional Requirements Section
Break down **what** needs to change into logical categories:
- Group related changes together
- Focus on functional outcomes, not implementation details
- Use bullet points for clarity

### 3.5 Requirements Inventory (Reference + Local, Anti-Omission Mechanism)

To avoid missing requirements from planning, include an inventory with stable IDs.

This inventory is split into two scopes.

#### Canonical requirements (by reference only)
- List only IDs from the canonical requirements repository (e.g. `DOC-02`, `PROMPT-04`).
- Do NOT restate or redefine their content here.
- Do NOT use 🟡 markers for canonical requirements.

#### Work-spec requirements (technical / transitional)
- Each local requirement gets an ID (e.g. `DATA-01`, `MIG-02`, `TEMP-01`).
- These may describe technical or implementation-level decisions.
- Items start with 🟡 and the 🟡 is removed when complete (no ✅ — absence of 🟡 means done).

**Rules:**
- Inventory must be complete (capture all technical requirements + constraints + deferred / non-goal items introduced by this work).
- Tasks and subtasks must cite requirement IDs.
- When a task is completed, remove 🟡 from any work-spec requirements it fully satisfied.

### 3.6 Non-goals / Deferred
Explicitly list anything deferred/out-of-scope from planning (also with IDs), so it can’t disappear.

### 4. Relevant Files Section
List files that will be modified, organized by importance:
- **Core Implementation**: Files central to the main changes
- **Supporting Files**: Files requiring minor updates or configuration changes

### 5. Task List Section
The heart of the specification - actionable implementation steps.

### 5.5 Traceability (Requirement ↔ Task)
Include a short mapping section that ensures every requirement is implemented or explicitly deferred:

- Each requirement ID maps to at least one task, or to a deferred/non-goal item
- No “orphan” tasks (tasks should point back to at least one requirement ID)

## Task List Guidelines

### Task Structure
N. Task Name 🟡
N.1. Sub-task description
N.2. Sub-task description
N.3. Sub-task description

**Progress Indicators:**
- 🟡 = TODO (task not yet complete)
- No marker = Complete (absence of 🟡 means done)

Tasks are created with 🟡 and the 🟡 is removed when complete.

### Task Naming
- Use descriptive action-oriented names
- Focus on what will be accomplished, not how
- Examples: "Update Configuration Model", "Implement Coordinate Conversion"

### Sub-task Numbering
- Always number sub-tasks (1.1, 1.2, 2.1, 2.2, etc.)
- Makes it easy to reference specific items during implementation
- Enables precise progress tracking

### Task Scope
- Each task should be implementable in a single focused session
- Break large tasks into smaller, manageable pieces
- Aim for 3-6 sub-tasks per main task

### Task Dependencies
- Order tasks logically based on dependencies
- Earlier tasks should provide foundation for later ones
- Consider data flow and component relationships

## What to Include

### ✅ Implementation Tasks
- Code changes and refactoring
- Data structure updates
- Algorithm modifications
- Configuration changes
- Documentation updates

### ✅ Integration Tasks
- Connecting components
- Updating interfaces
- Coordinate space conversions
- API modifications

### ✅ Verification Tasks
- Ensuring correct integration
- Validating data flow
- Checking for linter errors
- Confirming expected behavior

## What to Exclude

### Unit Testing Tasks (optional; ask first; last)

Work specs may or may not require unit testing (e.g., some teams don’t unit test view/UI code).

Rules:
- The agent MUST ask the human what unit testing (if any) is required for this work.
- If unit testing is required, add unit testing tasks at the END of the task list.
  - Keep them high-level (what to test and where), and defer the detailed testing workflow to the Unit Testing guides.

### ❌ Deployment Tasks
- Build configuration
- Release preparation
- Environment setup
- Production deployment

### ❌ External Dependencies
- Third-party library updates
- System configuration changes
- Infrastructure modifications

## Writing Style

**Scope note:** This section applies to **work specifications** (tasks, subtasks, and work-spec requirements), not to the canonical requirements repository in `/requirements/**`.

### Be Specific
- ✅ "Add `deviceDistance: Float` field to `TractorBeamConfig` (default: 0.1)"
- ❌ "Update configuration"

### Use Active Voice
- ✅ "Replace `cameraToPrizeDirection` with `player.aim`"
- ❌ "The camera direction should be replaced"

### Focus on Outcomes
- ✅ "Calculate camera-to-crosshairs direction vector for `player.aim`"
- ❌ "Do some math to figure out the direction"

### No Code Examples
- ✅ "Add `deviceDistance` parameter with default value 0.1"
- ❌ "Add `var deviceDistance: Float = 0.1` to the struct"

### Standalone Clarity
- Provide enough context that implementation can proceed without questions *after reading the Inputs section*
- Define what each component should do and how they should interact
- Explain the purpose behind each change

### Maintain Abstraction Levels
- Keep implementation details in sub-tasks
- Main tasks should describe functional goals
- Avoid mixing high-level and low-level concerns

## Progress Tracking

### Status Indicators
- **🟡 (TODO)**: Task not yet started or in progress
- **No marker (Complete)**: Task finished and verified (absence of 🟡 means done)

### Tracking Rules
- Tasks are created with 🟡 indicator
- Remove 🟡 when task is complete (no ✅ — absence of 🟡 means done)
- Only mark main tasks (not sub-tasks)
- Mark task complete when all sub-tasks are finished
- Update during implementation as tasks are completed

### Status Visibility
- 🟡 tasks show what remains to be done
- Tasks without 🟡 are complete
- Enables easy resumption after interruptions

## Example Template

```markdown
# [Feature/System Name]

## Inputs (Required Reading)
- [Planning doc path]
- [Other referenced docs/specs/links]
- [Key files/folders to inspect for conventions]

## Motivation
[Why this work is needed - current problems and desired outcomes]

## Functional Requirements
### [Category 1]
- [Requirement description]
- [Requirement description]

### [Category 2]
- [Requirement description]
- [Requirement description]

## Requirements Inventory

### Canonical requirements (by reference only)
- DOC-02
- PROMPT-04

### Work-spec requirements (technical / transitional)
- 🟡 CAT-A-01: ...
- 🟡 CAT-A-02: ...
- 🟡 CAT-B-01: ...

## Non-goals / Deferred
- DEFER-01: ...

## Relevant Files
### Core Implementation
- `path/to/main/file.swift` - [Brief description of changes]
- `path/to/other/file.swift` - [Brief description of changes]

### Supporting Files
- `path/to/config/file.swift` - [Brief description of changes]

## Task List

1. **[Task Name]** 🟡
   1.1. [Specific sub-task] (CAT-A-01)
   1.2. [Specific sub-task] (CAT-A-02, CAT-B-01)
   1.3. [Specific sub-task] (DEFER-01 if deferring something explicitly)

2. **[Task Name]** 🟡
   2.1. [Specific sub-task] (CAT-A-01)
   2.2. [Specific sub-task] (CAT-B-01)

3. **[Task Name]** 🟡
   3.1. [Specific sub-task] (CAT-A-02)
   3.2. [Specific sub-task] (CAT-A-02)
   3.3. [Specific sub-task] (CAT-B-01)

## Traceability
- CAT-A-01 → Tasks 1, 2
- CAT-A-02 → Tasks 1, 3
- CAT-B-01 → Tasks 1, 2, 3
- DEFER-01 → Non-goals / Deferred