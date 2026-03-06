Managed-By: skai
Managed-Id: guide.work-spec
Managed-Source: Guides/Spec/work-spec-creation.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

# Work Specification Guide

## Purpose

Orchestrates the work specification creation process. Work specifications provide structured documentation for complex coding tasks that require multiple steps, coordination between components, and careful planning. They serve as both implementation roadmaps and progress tracking tools.

**Key Principles:**
- **Agent-sufficient document**: A fresh agent should be able to implement by reading this spec *and* following the explicitly listed inputs (referenced docs/files/links). No implicit context.
- **Communication Tool**: Describes what code to write, not how to write it
- **No Code**: Contains no actual code - only descriptions of what needs to be implemented

**Overall Process:**
1. **Planning Phase**: Collaborative design through three stages -- scope discussion, ideation/questions/discussion, API sketch. Produces a planning document with all design decisions resolved.
2. **Requirements Normalization**: Promote product/system behaviors discovered during planning into the canonical requirements repository.
3. **Work Spec First Pass**: Write high-level tasks only (no subtasks) for review.
4. **Work Spec Second Pass**: Add detailed subtasks after approval.

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md`.

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After drafting the planning document (with 🟡 open questions).
- After resolving all 🟡 items and completing the API sketch (human confirms readiness to proceed).
- After requirements normalization updates to `/requirements/**` (human acknowledges before proceeding).
- After the work spec first pass (high-level tasks only) for review.
- After the work spec second pass (subtasks + traceability) for review.

---

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

**Behavior:** Context determines the action. The same command drives every phase of the process; the agent infers which step to execute based on the current state of the conversation and any existing documents.

**Workflow-specific `auto` rules:**

`auto` can be useful after the human has already approved the planning document and there are no remaining 🟡 items, to batch requirements normalization, work spec first pass, and work spec second pass. If the planning document still has unresolved 🟡 items, this workflow is blocked on human decisions and `auto` should STOP at that checkpoint.

**Phase actions on advance intent:**

- **Planning -- Create Planning Document:** summarize the scope discussion into a planning document; seed with 🟡 open questions for Stage 1.
- **Planning -- Resolve Open Questions:** continue the iterative discussion, updating the document as decisions are made.
- **Requirements Normalization:** promote product/system behaviors from the planning document into the canonical requirements repository.
- **Work Spec -- First Pass:** create work specification with high-level tasks only (no subtasks, no Traceability). Allows human to review overall sequence before details.
- **Work Spec -- Second Pass:** add detailed subtasks, 🟡 indicators, and the Traceability section.

---

## Planning Phase

Before writing a work spec, the design must be worked through in a planning document. The planning phase is a collaborative, iterative process with two stages.

**Prerequisite:** An informal scope discussion has already taken place (in chat, a meeting, a Jira ticket, etc.). The human and agent have discussed the problem space -- what is being solved, motivating use cases, scope boundaries, and affected architecture. The human enters the planning phase when they are ready to formalize the discussion into a planning document (e.g., "begin planning").

**Creating the planning document** (begin/next/continue):
1. Summarize the scope discussion so far (Problem, Current Architecture, initial approach framing).
2. Seed the document with 🟡-marked open questions and discussion topics derived from the conversation -- these become the agenda for Stage 1.

### Stage 1: Ideation, Questions, Discussion

Work through the design collaboratively. The agent proposes design elements and the human refines, redirects, or approves.

**Planning document rules (tightened):**

- The agent MUST seed the conversation with proposals.
  - The planning doc should not be a passive transcript. It should contain concrete proposals/options to help the human decide.

**🟡 Marker Protocol for Planning Documents:**

- Mark all unresolved discussion items with 🟡:
  - open questions
  - proposals/options under consideration
  - undecided design points / tradeoffs
- Avoid over-granularity:
  - If a single proposal contains many sub-bullets, prefer one 🟡 marker on the proposal line rather than 🟡 on every sub-bullet.
  - Use per-sub-bullet 🟡 only when individual sub-items can be independently accepted/rejected or are likely to be worked in different iterations.
- Do not propagate markers up the hierarchy:
  - Do NOT put 🟡 on a topic heading (or parent bullet) solely because it contains child 🟡 items.
  - Only mark the specific unresolved item line(s) with 🟡 so the marker count reflects true open items.
- Organize the document by topic (natural structure). This is a hard requirement:
  - Do NOT create workflow-shaped sections like "Questions", "Discussion", or "Decisions".
  - Do NOT repeat the same topic across multiple sections ("Topic X" in Discussion and again in Questions).
  - Do NOT create "Decision" sub-sections (or "Decision:" labels). Decisions are expressed by replacing the unresolved text inline.
  - Instead: each topic heading contains its own inline 🟡 questions/proposals/tradeoffs where they naturally belong.
- When the human explicitly approves a resolution:
  - REPLACE the 🟡 item inline with the approved plan/requirement/decision text (no separate "Questions" section, no "approved" marker).
  - Do NOT remove 🟡 preemptively. Only replace when the human has explicitly decided.
- The planning document is ready for the next stage when all 🟡 items have been replaced with approved content.

**Anti-patterns (do not do this):**

- "Questions" section + "Discussion" section + the same topic mentioned in both (duplicate content, multiple 🟡 markers for the same unresolved item).
- "Decisions" section that mirrors earlier proposals/questions.
- A topic described once as a proposal (🟡) and again elsewhere as a question (🟡) instead of being a single coherent entry.

**Preferred pattern (topic-first, single source of truth):**

- Write one section per topic.
- Put the unresolved items (🟡) directly under that topic.
- When approved, replace the unresolved line(s) in place with the approved requirement/decision text.

**Recommended planning document shape (suggested, not required):**

- Problem / goal
- Current architecture / constraints (if relevant)
- Topics (one section per major topic)
  - Each topic contains:
    - brief context
    - 🟡 items (questions/proposals/tradeoffs) inline
    - resolved items replaced inline with the approved text
    - any non-goals / deferrals (explicit)

Example topic shape:

- `## <Topic name>`
  - Context: ...
  - 🟡 Proposal: ...
    - Option A: ...
    - Option B: ...
  - 🟡 Question: ...
  - Approved: ... (after replacing 🟡 content inline)

**Optional: phased planning documents (only when the plan is large):**

Sometimes the planning grows large enough that it should be executed in phases. In that case, the planning document is still a single document, but it is structured so each phase has its own planning and API sketch content for traceability.

Rules:
- Default: no phases. Use phases only when the human or agent proposes them and there is an informal agreement on the phase breakdown.
- Phase sections MUST come last in the document (after any global planning content).
- Global planning content MAY exist before phase sections. Later phases may augment or supersede earlier decisions, but do not rewrite earlier phase sections.
- Each phase runs its own mini-cycle in order:
  1. Proposals, questions, discussion (Stage 1)
  2. API sketch (Stage 2)
  3. Requirements normalization
  4. Work spec writing
- Process-flow note: when Phase N is complete, the next step is to begin Phase N+1 (or conclude if there are no more phases).
  - Stopping condition: at the end of Phase N's cycle, STOP and wait using the standard gate line (see `Guides/Core/process-flow.md`). The "Next" step should be "Begin Phase N+1" (or "Complete" if there are no more phases).
  - Advance intent at that point is the signal to initialize Phase N+1's Stage 1 discussion content (seed proposals/questions under that phase section).
- Phase sections can start as lightweight placeholders (scope, rough idea). When it is time to begin a phase, the human tells the agent to initialize the phase's discussion content.

Recommended phased planning shape:
- Overview / global context (optional)
- Global topics (optional)
- Phase sections (last):
  - `## Phase 1: <name>`
    - `### Scope` (goal, in-scope, non-goals, dependencies, exit criteria)
    - `### Stage 1: Proposals, questions, discussion` (🟡 until resolved)
    - `### Stage 2: API sketch` (API "as of Phase 1")
    - `### Requirements normalization` (what will be added/updated in `/requirements/**` for this phase)
    - `### Work spec` (link to this phase's work spec)
    - `### Supersedes / changes vs earlier phases` (optional; explicit notes when Phase N counteracts Phase < N)
  - `## Phase 2: <name>` (repeat the same shape)

**Discussion principles:**
- Present findings and analysis, not pre-selected options. The human is the architect.
- When the human asks a question, answer it directly -- do not reframe it as a choice between options you've invented.
- Use concrete scenarios (design-by-use-case) to drive design decisions rather than abstract analysis.
- Capture both the decision and the reasoning in the document.

### Stage 2: API Sketch

Refine the design to the level of non-private API surfaces -- the interfaces through which components in the system connect and interact. This bridges the gap between high-level decisions and the work spec's implementation tasks.

**API sketch conventions:**
- Show APIs in the context of their enclosing type. Do not write isolated function signatures; instead write the type signature with the relevant member(s) enclosed within it.
- **New types/members**: Show the type and its new members.
- **Unchanged APIs**: Omit entirely -- do not list them, do not comment on their absence. Omission *is* the signal that the API is unchanged.
- **Protocol conformances**: When a protocol gains new members, do not repeat the signatures on conforming types. List the conforming types that need updating as a comment, but do not restate the API.
- **Removed APIs**: Show in the context of the enclosing type with ~~strikethrough~~ formatting.
- **Modified APIs**: Show the old signature with ~~strikethrough~~ immediately followed by the new signature.
- **API documentation**: Include doc comments on all new and modified APIs. The sketch is the first place these are written and they carry forward into implementation.
- **Code block formatting**: Use fenced code blocks with the language identifier (e.g., ` ```swift `) for syntax highlighting.

**Example:**

```swift
protocol Storing {
    ~~func save(_ output: StorageOutput, id: String) throws~~
    func save(_ output: StorageOutput, id: String, context: PerformContext) throws
    func loadFile(_ reference: FileReference) throws -> Data    // new
}

struct FileReference: Codable {                                 // new type
    let filename: String
    let type: FileType
}
```

Output: the planning document's "API Sketch" or equivalent section describes the non-private surface area -- types, protocols, and their relationships -- with enough specificity that a work spec can reference them.

### Planning Document Completeness

The planning phase is complete when:
- All 🟡 markers have been resolved (replaced with decisions).
- The key types and their relationships are described.
- The design has been validated against concrete use cases.
- The human confirms readiness to proceed to requirements normalization.

If the planning document uses phases:
- The overall planning phase may remain intentionally incomplete for later phases.
- For the current phase to proceed to requirements normalization, that phase section must have:
  - all 🟡 markers in the phase resolved, and
  - an API sketch for the phase, and
  - the human's approval to proceed with that phase.

Checkpoint: STOP and wait for the human to confirm readiness to proceed.

---

## Canonical Requirements (PRD) Normalization Step

After the planning phase is complete, product/system behaviors discovered during planning must be normalized into the canonical requirements repository.

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
  "Requirements Repository Organization".

### Canonical requirements writing style (anti-implementation guidance)

Write canonical requirements as if authored by a **product manager with no knowledge of the codebase**:

- Focus on **user-visible behavior**, **domain invariants**, and **system contracts**.
- Describe **what must be true**, not how it is achieved.
- Every requirement should be verifiable from the outside (a user, QA, or another system), without reading code.

**Do not include implementation/technical details such as:**

- Specific data structures, algorithms, or execution strategies (e.g., "use caching", "use a queue", "debounce", "run in background task")
- Storage mechanisms (e.g., "persist to disk as JSON", "CoreData", "SQLite", "FileStorage")
- Concurrency / threading / actors / async design (e.g., "use async/await", "MainActor", "perform off the main thread")
- Concrete Swift identifiers, file paths, or code formatting (backticked types, `.swift` filenames, method names, initializer signatures)
- Tooling and patterns (dependency injection frameworks, logging frameworks, testing frameworks)

**If a detail is important but inherently technical:**

- Put it in the **work spec** under "Work-spec requirements (technical / transitional)" (e.g. `MIG-01`, `TEMP-02`) instead of the canonical requirements repo.

#### Quick self-check (before writing to `/requirements/**`)

- Can this be understood by a non-engineer without loss of meaning?
- Does it mention *any* code identifier, file, module, dependency, or framework? If yes → rewrite.
- Is it phrased as a behavior/contract ("must/should/will") rather than a plan ("implement/add/refactor")?

#### Examples

- ✅ "The system must detect and report circular references in templated documents."
- ❌ "The `AssetCatalog` should DFS templates and throw `CircularReferenceError`."

- ✅ "Users must be able to view all validation issues for an asset in a single report."
- ❌ "Accumulate errors during parsing and return an aggregated error array."

The work specification references canonical requirement IDs produced by this step.

Checkpoint: STOP and wait for the human to acknowledge requirements updates before starting the work spec draft.

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

---

## File Naming Convention

Work specification and planning files are working documents. Create them following `Guides/Core/working-doc-conventions.md`.

**Planning Document:**
- Subpath: (none)
- File name: `[spec-name]-planning.md`
- Full path: `working-docs/<branch-path>/[spec-name]-planning.md`

**Work Spec Document:**
- Subpath: (none)
- File name: `[spec-name]-work-spec.md`
- Full path: `working-docs/<branch-path>/[spec-name]-work-spec.md`

Where `[spec-name]` = the specification name (e.g., `observable-wrapper`).

**Examples** (branch: `work/step-refactor`):
- Planning: `working-docs/work/step-refactor/observable-wrapper-planning.md`
- Work Spec: `working-docs/work/step-refactor/observable-wrapper-work-spec.md`

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
- Items start with 🟡.

**Rules:**
- Inventory must be complete (capture all technical requirements + constraints + deferred / non-goal items introduced by this work).
- Tasks and subtasks must cite requirement IDs.

### 3.6 Non-goals / Deferred
Explicitly list anything deferred/out-of-scope from planning (also with IDs), so it can't disappear.

### 4. Relevant Files Section
List files that will be modified, organized by importance:
- **Core Implementation**: Files central to the main changes
- **Supporting Files**: Files requiring minor updates or configuration changes

### 5. Task List Section
The heart of the specification - actionable implementation steps.

### 5.5 Traceability (Requirement ↔ Task)
Include a short mapping section that ensures every requirement is implemented or explicitly deferred:

- Each requirement ID maps to at least one task, or to a deferred/non-goal item
- No "orphan" tasks (tasks should point back to at least one requirement ID)

## Task List Guidelines

### Task Structure

```markdown
1. **Task Name** 🟡
   - **Done when:** [Short, stable completion criteria]
   - **Implementation**
     1. Sub-task description (REQ-ID)
     2. Sub-task description (REQ-ID)
   - **Verification**
     3. Run <command(s)> and define what "pass" means (REQ-ID) [evidence: ...]
```

Sub-tasks use indented numbered lists. Number subtasks sequentially across the task (including Verification). Referencing "1.2" means task 1, sub-task 2 -- the nesting is implied by indentation.

Tasks are created with 🟡 (see `Guides/Core/process-flow.md` for marker semantics).

### Task Naming
- Use descriptive action-oriented names
- Focus on what will be accomplished, not how
- Examples: "Update Configuration Model", "Implement Coordinate Conversion"

### Sub-task Numbering
- Use indented numbered lists for sub-tasks; the N.N reference (e.g., 1.1, 2.3) is implied by the task number and sub-task position
- Makes it easy to reference specific items during implementation
- Enables precise progress tracking

### Task Scope
- Each task should be implementable in a single focused session
- Break large tasks into smaller, manageable pieces
- Aim for 3-6 sub-tasks per main task
- Default expectation: each top-level task leaves the codebase in a runnable state for its verification whenever practical (build/tests).
  - Allowed exception (explicit in the work spec): **Refactor tranche** -- a small sequence of top-level tasks that may temporarily break compilation, but must end with a **Green Gate** task/subtask that restores compilation and runs the relevant verification commands.

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
  - Verification subtasks are mandatory for each top-level task (build/test/lint as applicable).
  - Source of truth for project-specific commands/paths is `docs/skai/integration.md` (do not invent commands).
  - Evidence must be captured inline on the verification subtask line using an evidence bracket:
    - Format: `[evidence: <command variant>; exit <code>; output: <optional link(s)>]`
    - Example: `[evidence: <command>; exit 0; output: [output](working-docs/<branch-path>/work-spec/<spec-name>/evidence/<slug>.txt)]`
    - On failure: persist full output to a file and link it.
    - On success: a linked output file is optional; still record command variant and exit code inline.

## What to Exclude

### Unit Testing Tasks (optional; ask first; last)

Work specs may or may not require unit testing (e.g., some teams don't unit test view/UI code).

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

🟡 marker semantics are defined in `Guides/Core/process-flow.md`.

Work-spec-specific rule:
- Only mark main tasks with 🟡 (not sub-tasks)

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
   - **Done when:** [Behavioral + verification-based completion criteria]
   - **Implementation**
     1. [Specific sub-task] (CAT-A-01)
     2. [Specific sub-task] (CAT-A-02, CAT-B-01)
     3. [Specific sub-task] (DEFER-01 if deferring something explicitly)
   - **Verification**
     4. [Run command(s) from integration doc; define "pass"] (CAT-A-01) [evidence: exit 0; output: [log](working-docs/<branch-path>/work-spec/<spec-name>/evidence/<slug>.txt)]

2. **[Task Name]** 🟡
   - **Done when:** [Behavioral + verification-based completion criteria]
   - **Implementation**
     1. [Specific sub-task] (CAT-A-01)
     2. [Specific sub-task] (CAT-B-01)
   - **Verification**
     3. [Run command(s) from integration doc; define "pass"] (CAT-B-01) [evidence: exit 0; output: [log](working-docs/<branch-path>/work-spec/<spec-name>/evidence/<slug>.txt)]

3. **[Task Name]** 🟡
   - **Done when:** [Behavioral + verification-based completion criteria]
   - **Implementation**
     1. [Specific sub-task] (CAT-A-02)
     2. [Specific sub-task] (CAT-A-02)
     3. [Specific sub-task] (CAT-B-01)
   - **Verification**
     4. [Run command(s) from integration doc; define "pass"] (CAT-A-02) [evidence: exit 0; output: [log](working-docs/<branch-path>/work-spec/<spec-name>/evidence/<slug>.txt)]

## Traceability
- CAT-A-01 → Tasks 1, 2
- CAT-A-02 → Tasks 1, 3
- CAT-B-01 → Tasks 1, 2, 3
- DEFER-01 → Non-goals / Deferred