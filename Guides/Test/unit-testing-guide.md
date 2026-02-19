Managed-By: ai-dev-process
Managed-Id: guide.unit-testing
Managed-Source: Guides/Test/unit-testing-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-19

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

---

## Commands

### Next Command

**Definition:** Any of `"begin"`, `"next"`, or `"continue"` -- these are synonymous.

**Behavior:** Context determines the action:
- If waiting to proceed → mark current step complete (removes 🟡 where applicable), execute next step
- If stopped due to ambiguities or unexpected challenges → resume where you left off

**Mechanics:**
- Hands off to sub-process: Sub-process defines its own checkpoints and command protocol
- When sub-process completes: Returns to checkpoint, waits for Next Command to advance
- If no more work remains: Done

### Next Command + `auto`

Next Command followed by `auto` (e.g., `"next auto"`) -- Auto-advances through sections until infrastructure needed or non-trivial failure.

**First step (Planning):**
- Executes the planning process (see planning guide)
- Output: Test file with sections marked 🟡 (TODO)
- Stops when planning is complete

**`"next auto"`**
- Auto-advances through all remaining sections/tests without stopping at checkpoints
- When encountering tests needing human input:
  1. **Infrastructure changes needed** - Skip tests in that section, leave 🟡 markers, continue to next section
  2. **Non-trivial test failure** - Skip failing tests, leave 🟡 markers, continue with remaining tests
- Completes all tests that can be completed automatically
- At end: Reports which tests were skipped and why
- Use when: Want to make maximum progress and batch human input items together

**Skip and Continue Behavior:**

*When infrastructure changes needed (entire section):*
- Leave section marked with 🟡
- Leave all tests in section marked with 🟡
- Document infrastructure requirements in work document
- Continue to next 🟡 section

*When non-trivial test failure (individual tests):*
- Remove 🟡 from passing tests in section
- Leave 🟡 on failing tests
- Leave 🟡 on section if any tests still have 🟡
- Document failure details in work document
- Continue with remaining tests in section or move to next section

*Auto-fixes allowed:*
- Obvious typos (wrong variable names, enum values)
- Import missing modules
- Simple compilation errors
- Execution continues after auto-fix

**Final report includes:**
- Tests completed (with 🟡 removed)
- Tests skipped (with 🟡 remaining) with reasons
- Human can then address skipped items with Next Command or approve/modify proposed solutions

---

## Process Adherence

**CRITICAL: Follow the defined process exactly. If you want to deviate, ask first.**

The testing process is designed for:
- **Incremental progress** - Complete and verify one section before moving to next
- **Focused work** - Easier to review/approve smaller chunks
- **Early validation** - Catch issues early before they multiply
- **Clear checkpoints** - Human can approve/redirect after each section

**If you identify a reason to deviate:**
1. **STOP** - Don't proceed with the deviation
2. **EXPLAIN** - Document why you think deviation would be beneficial
3. **ASK** - Wait for human approval before deviating
4. **PROCEED** - Only deviate if explicitly approved

**Examples of deviations that require approval:**
- Working on multiple sections at once (even if they share infrastructure)
- Skipping infrastructure phase because "it's obvious"
- Combining multiple phases into one
- Reordering the defined steps
- Creating global work documents instead of section-specific ones

**Never assume a shortcut is acceptable, even if it seems efficient.**

---

## Workflow

### Step 1: Planning

**Trigger:** Next Command

**Process:** Execute `Guides/Test/unit-test-planning-guide.md`

**Output:** Test file with sections marked 🟡 (indicating TODO)

**Checkpoint:** Wait for Next Command

---

### Step 2: Section Implementation (repeat per section)

**Trigger:** Next Command

**For each section marked 🟡:**

1. **Infrastructure Phase**
   - Execute `Guides/Test/unit-test-infrastructure-guide.md` for this section
   - Stops at checkpoints defined in that guide
   - Next Command after infrastructure complete → Proceed to Writing Phase

2. **Writing & Execution Phase**
   - Execute `Guides/Test/unit-test-writing-guide.md` for this section
   - Stops at checkpoints defined in that guide
   - When all tests pass and work documented, writing phase is complete

3. **Mark Complete**
   - Remove 🟡 from section
   - Section is done

**Repeat:** Find next 🟡 section and repeat process

**When no 🟡 sections remain:** All testing complete

---

## Progress Tracking

**In test files:**
- `🟡` = TODO / Not yet implemented
- No emoji = Completed and approved

**Work documents:**
- Created per section during implementation, following `Guides/Core/working-doc-conventions.md`
- Subpath: `testing/<suite-name>`
- File name: `<section-name>-<phase>.md`
- Full path: `working-docs/<branch-path>/testing/<suite-name>/<section-name>-<phase>.md`
- `<suite-name>` = test file name without "Tests.swift" (e.g., `TemplateRenderer` from `TemplateRendererTests.swift`)
- `<section-name>` = section name in kebab-case (e.g., `success-tests`, `error-handling`)
- `<phase>` = `infrastructure` or `writing`
- Example: `working-docs/work/feature-branch/testing/TemplateRenderer/success-tests-writing.md`

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
AI: [Tests pass - removes 🟡 from Render Success Tests]

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

**Commands:**
- Next Command (`"begin"` / `"next"` / `"continue"`) - Proceed
- Next Command + `auto` - Auto-advance until infrastructure needed or non-trivial failure

**Process Flow:**
1. Planning → Creates ALL test files with 🟡 sections (single or multiple files) → CHECKPOINT
2. For each file (sequential):
   - For each section: infrastructure → CHECKPOINT → writing → CHECKPOINT → Remove 🟡
   - Complete all sections in file before moving to next file
3. Next auto → Auto-complete all possible sections, skip those needing human input, report at end
4. When no 🟡 remain in any file → Done

**Process Adherence:**
- Follow the process exactly as defined
- Work on ONE section at a time (even if infrastructure is shared)
- If you want to deviate: STOP, EXPLAIN, ASK, wait for approval
- Never assume shortcuts are acceptable

**Details:** See sub-guides for complete processes
