Managed-By: skai
Managed-Id: guide.unit-testing
Managed-Source: Guides/Test/unit-testing-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

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

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After planning is complete (all files/sections/tests are stubbed and marked 🟡).
- Before running tests for a section (ensure writing is complete for the section).
- After test execution results are gathered (human can approve conclusions and next step).
- At the end of `auto` runs (report what was completed vs what was skipped and why).

---

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

**Behavior:** Each sub-guide defines its own checkpoints. When a sub-process completes, control returns here and the orchestrator waits for advance intent before proceeding.

**Workflow-specific `auto` rules:**

`auto` advances through all remaining sections/tests without stopping at sub-guide checkpoints.

Skip and continue behavior:
- **Infrastructure changes needed**: skip tests in that section, leave 🟡 markers, document infrastructure requirements, continue to next section.
- **Non-trivial test failure**: skip failing tests, leave 🟡 markers, document failure details, continue with remaining tests.

Auto-fixes allowed: obvious typos, missing imports, simple compilation errors.

At end: report which tests were completed and which were skipped (with reasons).

---

## Workflow

**If you want to deviate from the process below, STOP and ask first.** Do not skip phases, reorder steps, or combine phases without explicit approval.

### Step 1: Planning

**Trigger:** Advance intent

**Process:** Execute `Guides/Test/unit-test-planning-guide.md`

**Output:** Test file with sections marked 🟡 (indicating TODO)

**Checkpoint:** Wait for advance intent.

---

### Step 2: Section Implementation (repeat per section)

**Trigger:** Advance intent

**For each section marked 🟡:**

1. **Infrastructure Phase**
   - Execute `Guides/Test/unit-test-infrastructure-guide.md` for this section
   - Stops at checkpoints defined in that guide
   - Advance intent after infrastructure complete → Proceed to Writing Phase

2. **Writing & Execution Phase**
   - Execute `Guides/Test/unit-test-writing-guide.md` for this section
   - Stops at checkpoints defined in that guide
   - When all tests pass and work documented, writing phase is complete

3. **Section complete**
   - On advance intent, section is done

**Repeat:** Find next 🟡 section and repeat process

**When no 🟡 sections remain:** All testing complete

---

## Work Documents

Created during implementation, following `Guides/Core/working-doc-conventions.md`:
- Subpath: `testing/<suite-name>`
- `<suite-name>` = test file name without "Tests.swift" (e.g., `TemplateRenderer` from `TemplateRendererTests.swift`)
- See sub-guides for specific file names and structure.

---

## Example Workflow

### Single Type (One Test File)

```
--- Planning ---
Human: "begin"
AI: [Creates TemplateRendererTests.swift with sections:
     - Render Success Tests 🟡
     - Error Handling Tests 🟡]
AI: [CHECKPOINT - stops]

--- Section 1 ---
Human: "next"
AI: [Finds first 🟡 section: Render Success Tests]
AI: [Executes infrastructure guide]
AI: [CHECKPOINT - stops after infrastructure analysis]
Human: "next"
AI: [Marks infrastructure complete, proceeds to writing guide]
AI: [Implements tests]
AI: [CHECKPOINT - stops before running tests]
Human: "next"
AI: [Runs tests, documents results]
AI: [Tests pass - CHECKPOINT - presents results]
Human: "next"
AI: [Removes 🟡 from Render Success Tests]

--- Section 2 ---
Human: "next"
AI: [Finds next 🟡 section: Error Handling Tests]
...
[Continues until all sections complete]

--- Done ---
Human: "next"
AI: "All sections complete. Testing done!"
```

### Multiple Types (Multiple Test Files)

```
--- Planning (creates ALL test files upfront) ---
Human: "begin"
AI: [Creates PlaceholderParserTests.swift with sections:
     - Doc Placeholder Tests 🟡
     - Fragment Placeholder Tests 🟡]
AI: [Creates MarkdownTokenizerTests.swift with sections:
     - Heading Tests 🟡
     - Code Fence Tests 🟡]
AI: [Creates TemplateRendererTests.swift with sections:
     - Render Success Tests 🟡
     - Error Handling Tests 🟡]
AI: [CHECKPOINT - stops with list of all planned files]

--- File 1: PlaceholderParserTests ---
Human: "next"
AI: [Finds first 🟡 section in first file: Doc Placeholder Tests]
AI: [Executes infrastructure guide]
...
[Completes all sections in PlaceholderParserTests]

--- File 2: MarkdownTokenizerTests ---
Human: "next"
AI: [Moves to first 🟡 section in next file: Heading Tests]
...
[Completes all sections in MarkdownTokenizerTests]

--- File 3: TemplateRendererTests ---
Human: "next"
AI: [Moves to first 🟡 section in final file: Render Success Tests]
...
[Completes all sections]

--- Done ---
Human: "next"
AI: "All test files complete. Testing done!"
```

### Using "next auto"

```
Human: "next auto"
AI: [Completes all Feature B Tests - no infrastructure, all pass]
AI: [Completes all Helper Tests - no infrastructure, all pass]
AI: [Encounters Feature C Tests - new stub needed]
AI: [Skips all Feature C Tests, leaves all 🟡, documents infrastructure needs]
AI: [Completes 3 of 5 Feature D Tests - 2 fail with unclear logic]
AI: [Removes 🟡 from 3 passing tests, keeps 🟡 on 2 failing, documents failures]
AI: [Completes all Feature E Tests - no infrastructure, all pass]
AI: "Auto-advance complete. Completed: B (5/5), Helper (3/3), D (3/5), E (4/4). Skipped: C (0/6 - infrastructure), D (2/5 - test failures)"
Human: [Reviews skipped tests]
Human: "next"
AI: [Works on Feature C Tests with human guidance for infrastructure]
```

---

## Quick Reference

**Sub-guides:**
- Planning: `Guides/Test/unit-test-planning-guide.md`
- Infrastructure: `Guides/Test/unit-test-infrastructure-guide.md`
- Writing & Execution: `Guides/Test/unit-test-writing-guide.md`
