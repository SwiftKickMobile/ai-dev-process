Managed-By: skai
Managed-Id: guide.unit-testing
Managed-Source: Guides/Test/unit-testing-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-07

# Unit Testing Guide

## Purpose

Orchestrates the unit testing workflow. This guide coordinates the overall process while delegating detailed execution to specialized sub-guides.

**Use Cases:**
- **Writing tests for a single type** - One test file, one source type
- **Writing tests for multiple types** - Multiple test files planned upfront, implemented sequentially
- **Adding new tests to existing suites** - Adding sections or tests to existing files
- **Fixing failing tests** - Addressing test failures in existing tests

**Multi-Type Testing:**
When testing multiple types in a single session:
1. **Plan all test files upfront** - Planning phase creates all test files with 🟡 sections
2. **Implement one file at a time** - Complete all sections in one file before moving to the next
3. **Sequential by file** - Aligns with work spec subtask structure

**Sub-guides:**
- **Planning**: `Guides/Test/unit-test-planning-guide.md`
- **Infrastructure**: `Guides/Test/unit-test-infrastructure-guide.md`
- **Writing & Execution**: `Guides/Test/unit-test-writing-guide.md`

## Gates

Core rule: every time the agent is waiting on the human, the message must end with a `⏳ GATE:` line. The only normal exception is full workflow completion, which uses `🏁 Complete. Let me know if anything needs adjustment.`

Use these standard gate lines:
- Planned gate: `⏳ GATE: Next: <what happens after your response>. Say "next" or what to change.`
- Blocked gate: `⏳ GATE: Blocked: <reason>. Resolve and say "next" to continue.`

Planned gates are the expected review points of this workflow. At each planned gate:
1. Summarize what was completed and what should happen next.
2. End with the planned gate line.
3. STOP and wait for the human.

In the planned gate line, `<what happens after your response>` should describe what the agent will do after the human gives advance intent.

If an unexpected blocker prevents continued work, use the blocked gate line and STOP until the human resolves it.

Workflow-specific gate note:
- This guide is an orchestrator. Some planned gates are emitted by the planning/infrastructure/writing sub-guides. When a sub-guide stops at a gate, control returns here and the human's response resumes the orchestrated testing flow.

When the workflow finishes (all testing complete), use: `🏁 Complete. Let me know if anything needs adjustment.`

Planned gates for this workflow:
- After planning is complete (all files/sections/tests are stubbed and marked 🟡).
- After the global infrastructure review is complete for all planned tests (owned by the infrastructure sub-guide).
- Before running tests for a section (owned by the writing sub-guide).
- After test execution results are gathered for a section (owned by the writing sub-guide).
- At the end of `next auto`, if work remains or the run stopped early, to report what was completed vs what was skipped and why.

---

## Advance intent

Advance intent moves past the current gate. Common signals: "next", "continue", "go ahead", "do it".

Rules:
- Recognized as approval to move past a gate only after you output a `⏳ GATE:` line.
- "we should...", "let's..." = discussion/context-setting, NOT authorization.
- Outside a gate, interpret "begin"/"next"/"continue" using the active workflow step below. Do not use them to skip phases or clear section markers early.

`auto` = advance intent that bypasses planned gates only. Blocked gates always require explicit human resolution.
`auto to <target>` = auto-advance but STOP before the named workflow target. Use stable identifiers such as `<suite-name>` or `<suite-name>/<section-name>`.

Progress tracking:
- Default rule: 🟡 = TODO or pending approval. Do not clear 🟡 without human approval.
- This workflow owns an orchestration document at `working-docs/<branch-path>/<session-name>/testing/unit-testing.md`.
- `session-name` is the required name for this overall testing effort on the current branch.
- The orchestration document is the parent workflow artifact and owns these gate/phase markers:
  - `Planning 🟡`
  - `Infrastructure 🟡`
  - `Writing 🟡`
- Planning creates the test files/sections/tests with 🟡 markers and also seeds the orchestration document with the parent phase markers.
- At the planning gate, `Planning 🟡` remains in the orchestration document until the human gives advance intent.
- At the infrastructure gate, `Infrastructure 🟡` remains in the orchestration document until the human gives advance intent.
- At section-level writing gates, the section's 🟡 marker remains until the relevant writing-guide gate is approved.
- `Writing 🟡` remains in the orchestration document until all planned sections across the testing session are complete and the human gives advance intent at the final section-completion gate.
- If writing later discovers missing infrastructure, leave `Writing 🟡` in place and re-add or keep `Infrastructure 🟡` in the orchestration document so the parent workflow clearly shows that another infrastructure pass is required.
- If a section is skipped because infrastructure remains missing or a non-trivial failure remains unresolved, leave that section's 🟡 marker in place.

**Behavior:** This guide orchestrates the sub-guides. When a sub-process stops at a gate, control returns here and the human's next response resumes the global workflow, whether that means entering infrastructure, continuing the current file's section writing, or deciding how to handle skipped work.

**Workflow-specific `auto` rules:**

- `auto` advances through all remaining work without stopping at planned gates.
- Complete the global infrastructure pass before starting section-by-section writing.
- In the orchestration document:
  - clear `Planning 🟡` only after the planning gate is auto-approved
  - clear `Infrastructure 🟡` only after the infrastructure gate is auto-approved
  - keep `Writing 🟡` until all remaining sections in the testing session are actually complete
- Skip and continue behavior:
  - **Missing infrastructure discovered during writing**: skip the affected tests/section, leave 🟡 markers, document what infrastructure is missing, and continue to the next section. At the next planned gate, the human can decide whether to re-enter the infrastructure phase.
  - **Non-trivial test failure**: skip failing tests, leave 🟡 markers, document failure details, continue with remaining tests or next section.
- Auto-fixes allowed: obvious typos, missing imports, simple compilation errors.
- `auto to <target>` should use stable identifiers such as `<suite-name>` or `<suite-name>/<section-name>`. Do not use a bare section name by itself.
- If auto completes everything with no skipped work and no bounded target stop, finish with `🏁 Complete...`.
- If auto stops early because work remains, report which tests were completed and which were skipped (with reasons) at the planned gate.

---

## Workflow

**If you want to deviate from the process below, STOP and ask first.** Do not skip phases, reorder steps, or combine phases without explicit approval.

### Step 1: Planning

**Trigger:** Advance intent

**Process:**
1. Create the orchestration document for this workflow:
   - **Session name:** `<session-name>`
   - **Subpath:** `testing`
   - **File name:** `unit-testing.md`
   - **Full path:** `working-docs/<branch-path>/<session-name>/testing/unit-testing.md`
   - Start with `Planning 🟡` in the checklist.
2. Execute `Guides/Test/unit-test-planning-guide.md`

**Output:**
- Test file(s) with sections marked 🟡 (indicating TODO)
- Orchestration document updated with:
  - `Infrastructure 🟡`
  - `Writing 🟡`

**Gate:** STOP with the planned gate line. Leave `Planning 🟡` in the orchestration document. `Next` should say you will clear `Planning 🟡` and begin the infrastructure pass for all planned tests.

---

### Step 2: Infrastructure (one pass for all planned tests)

**Trigger:** Advance intent

**For all planned tests in the current testing session:**

1. **Infrastructure pass**
   - Execute `Guides/Test/unit-test-infrastructure-guide.md` once for the whole testing session
   - That guide covers **all planned sections/tests across all planned test files**, not one suite or one section at a time
   - Follow the gates defined in that guide
   - Advance intent after infrastructure completion → clear `Infrastructure 🟡` in the orchestration document and proceed to section writing

---

### Step 3: Section Writing & Execution (repeat per section, file-by-file)

**Trigger:** Advance intent

**For each section marked 🟡 in the current file:**

1. **Writing & Execution Phase**
   - Execute `Guides/Test/unit-test-writing-guide.md` for this section
   - Follow the gates defined in that guide
   - When the writing guide reaches the post-results gate:
     - if the section passed, advance intent completes that section
     - if the section stopped with skipped tests due to missing infrastructure, advance intent returns control here with that section still marked 🟡

2. **Section complete**
   - Remove 🟡 from the completed section only after the human gives advance intent at the relevant gate
   - If no 🟡 sections remain anywhere, clear `Writing 🟡` in the orchestration document only after that final advance intent and complete the workflow
   - If no 🟡 sections remain anywhere, complete the workflow

**Repeat:** Finish the current file before moving to the next file:
- Find the next 🟡 section in the current file and repeat Step 3.
- If the current file has no remaining runnable sections, the human may decide at the next planned gate whether to re-enter infrastructure or move to the next file.
- Move to the next file only after the current file's remaining work is either completed or explicitly deferred by the human.

---

## Work Documents

Created during implementation, following `Guides/Core/working-doc-conventions.md`:

**Orchestration document (owned by this guide):**
- **Subpath:** `testing`
- **File name:** `unit-testing.md`
- **Full path:** `working-docs/<branch-path>/<session-name>/testing/unit-testing.md`
- **Purpose:** Tracks parent-workflow progress across planning, one global infrastructure pass, and overall writing completion.
- **Structure:**
  ```markdown
  # Unit Testing
  
  ## Context
  This document tracks the overall unit-testing workflow for [scope].
  
  ## Checklist
  
  - Planning 🟡
  - Infrastructure 🟡
  - Writing 🟡
  
  ## Notes
  
  - [High-level status, skips, or handoff notes]
  ```

**Sub-guide documents:**
- Infrastructure guide: `working-docs/<branch-path>/<session-name>/testing/infrastructure.md`
- Writing guide: per-section writing/execution document under `working-docs/<branch-path>/<session-name>/testing/<suite-name>/`

---

## Example Workflow

### Single Type (One Test File)

```
--- Planning ---
Human: "begin"
AI: [Creates `working-docs/<branch-path>/<session-name>/testing/unit-testing.md` with `Planning 🟡`]
AI: [Creates TemplateRendererTests.swift with sections:
     - Render Success Tests 🟡
     - Error Handling Tests 🟡]
AI: [Adds `Infrastructure 🟡` and `Writing 🟡` to `unit-testing.md`]
AI: [⏳ GATE: Next: Begin the infrastructure pass for all planned tests.]

--- Infrastructure ---
Human: "next"
AI: [Removes `Planning 🟡` from `unit-testing.md`]
AI: [Executes infrastructure guide for all planned tests in the session]
AI: [⏳ GATE emitted by infrastructure guide]
Human: "next"
AI: [Removes `Infrastructure 🟡` from `unit-testing.md`]
AI: [Marks infrastructure complete, proceeds to first section]

--- Section 1 ---
Human: "next"
AI: [Finds first 🟡 section: Render Success Tests]
AI: [Implements tests]
AI: [⏳ GATE emitted by writing guide before running tests]
Human: "next"
AI: [Runs tests, documents results]
AI: [⏳ GATE emitted by writing guide with test results]
Human: "next"
AI: [Removes 🟡 from Render Success Tests]

--- Section 2 ---
Human: "next"
AI: [Finds next 🟡 section: Error Handling Tests]
...
[Continues until all sections complete]

--- Done ---
Human: "next"
AI: [Removes `Writing 🟡` from `unit-testing.md`]
AI: "🏁 Complete. Let me know if anything needs adjustment."
```

### Multiple Types (Multiple Test Files)

```
--- Planning (creates ALL test files upfront) ---
Human: "begin"
AI: [Creates `working-docs/<branch-path>/<session-name>/testing/unit-testing.md` with `Planning 🟡`]
AI: [Creates PlaceholderParserTests.swift with sections:
     - Doc Placeholder Tests 🟡
     - Fragment Placeholder Tests 🟡]
AI: [Creates MarkdownTokenizerTests.swift with sections:
     - Heading Tests 🟡
     - Code Fence Tests 🟡]
AI: [Creates TemplateRendererTests.swift with sections:
     - Render Success Tests 🟡
     - Error Handling Tests 🟡]
AI: [Adds `Infrastructure 🟡` and `Writing 🟡` to `unit-testing.md`]
AI: [⏳ GATE: Next: Begin the infrastructure pass for all planned tests.]

--- Infrastructure ---
Human: "next"
AI: [Removes `Planning 🟡` from `unit-testing.md`]
AI: [Executes infrastructure guide for all planned tests across all planned test files]
AI: [⏳ GATE emitted by infrastructure guide]
Human: "next"
AI: [Removes `Infrastructure 🟡` from `unit-testing.md`]

--- File 1: PlaceholderParserTests ---
Human: "next"
AI: [Finds first 🟡 section in first file: Doc Placeholder Tests]
...
[Completes all sections in PlaceholderParserTests]

--- File 2: MarkdownTokenizerTests (after File 1 is complete) ---
Human: "next"
AI: [Moves to first 🟡 section in next file: Heading Tests]
...
[Completes all sections in MarkdownTokenizerTests]

--- File 3: TemplateRendererTests (after File 2 is complete) ---
Human: "next"
AI: [Moves to first 🟡 section in final file: Render Success Tests]
...
[Completes all sections]

--- Done ---
Human: "next"
AI: [Removes `Writing 🟡` from `unit-testing.md`]
AI: "🏁 Complete. Let me know if anything needs adjustment."
```

### Using "next auto"

```
Human: "next auto"
AI: [Creates/updates `working-docs/<branch-path>/<session-name>/testing/unit-testing.md`, auto-clears `Planning 🟡`, completes the global infrastructure pass, auto-clears `Infrastructure 🟡`]
AI: [Completes all Feature B Tests - infrastructure already covered, all pass]
AI: [Completes all Helper Tests - infrastructure already covered, all pass]
AI: [Encounters Feature C Tests - missing infrastructure was not identified earlier]
AI: [Skips all Feature C Tests, leaves section 🟡 markers in place, re-adds or keeps `Infrastructure 🟡`, documents missing infrastructure needs]
AI: [Completes 3 of 5 Feature D Tests - 2 fail with unclear logic]
AI: [Removes 🟡 from 3 passing tests, keeps 🟡 on 2 failing, keeps `Writing 🟡`, documents failures]
AI: [Completes all Feature E Tests - no infrastructure, all pass]
AI: "Auto-advance complete. Completed: B (5/5), Helper (3/3), D (3/5), E (4/4). Skipped: C (0/6 - missing infrastructure discovered during writing), D (2/5 - test failures)"
Human: [Reviews skipped tests]
Human: "next"
AI: [At the next planned gate, the human decides to re-enter infrastructure; the agent updates `Infrastructure 🟡` work and then resumes section writing]
```

---

## Quick Reference

**Sub-guides:**
- Planning: `Guides/Test/unit-test-planning-guide.md`
- Infrastructure: `Guides/Test/unit-test-infrastructure-guide.md`
- Writing & Execution: `Guides/Test/unit-test-writing-guide.md`

**Parent progress artifact:**
- `working-docs/<branch-path>/<session-name>/testing/unit-testing.md`
