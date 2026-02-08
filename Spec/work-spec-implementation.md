
# Work Task Development Rules

## 🚨 CRITICAL RULE - READ FIRST 🚨

**NEVER remove 🟡 from the current task in the work spec document.**
- You can remove 🟡 from PREVIOUS tasks when starting a new task
- You NEVER remove 🟡 from the CURRENT task 
- Task completion (removing 🟡) happens when the human asks for the next task

---

## Commands

### Next Command

**Definition:** Any of `"begin"`, `"next"`, or `"continue"` — these are synonymous.

**Behavior:** Context determines the action:
- If waiting to proceed → mark previous task complete (if applicable) and begin next task
- If stopped due to ambiguities or unexpected challenges → resume implementation where you left off

**When executing next task:**
1. Find the first task marked with 🟡 (TODO) in the work spec
2. **If not the first task:** 
   - Remove 🟡 from the previous (just-completed) task
   - Remove 🟡 from any Requirements Inventory items in the *work spec only* that were fully satisfied by the completed task
   - Remove 🟡 from any API Inventory items fully implemented by the completed task
3. Implement the next task in sequence (referring to requirement IDs for full context)
4. Follow the Task N implementation rules below

**Progress Indicators:**
- 🟡 = TODO (task not yet complete)
- No marker = Complete (task finished)

### Next Command + `auto`

Next Command followed by `auto` (e.g., `"next auto"`) — Auto-advances through all remaining tasks without stopping at checkpoints.

**Behavior:**
- Automatically implements all remaining 🟡 tasks in sequence
- Removes 🟡 from completed tasks as it progresses
- When encountering blockers:
  1. **Ambiguity/unclear requirements** - Stop, leave 🟡 on problematic task and all remaining tasks, document the issue, wait for clarification
  2. **Compilation errors** - Attempt simple fixes (typos, imports), if persists: stop, leave 🟡, document error
  3. **Non-trivial issues** - Stop, leave 🟡, document problem, wait for human input
- At end: Reports progress (tasks completed vs tasks remaining with 🟡)
- Use when: Want to make maximum progress and batch human input items together

**Auto-fixes allowed:**
- Obvious typos (wrong variable names, enum values)
- Missing imports
- Simple compilation errors
- Minor API mismatches between spec and implementation

**Final report includes:**
- Tasks completed (🟡 removed)
- Tasks remaining (with 🟡) and reasons why they were skipped
- Human can then address remaining items with Next Command or approve/modify approach

## Task N Implementation

When implementing Task N:

### Step 1: Mark Previous Task Complete (if applicable)
- **IF** there is a previous task with 🟡: Remove 🟡 from it in work spec
- **IF** implementing the first task: Skip this step (no previous task exists)
- **Update Requirements Inventory (work spec only)**: Remove 🟡 from any requirements listed in the work spec inventory (e.g., `DATA-01`, `MIG-02`, `TEMP-01`) that were fully satisfied by the completed task
- **Update API Inventory**: Remove 🟡 from any APIs that were fully implemented by the completed task

**Important:**
- Do NOT add, remove, or modify any markers in the canonical requirements repository (`/requirements/**`).
- Canonical requirements are referenced by ID only and never carry progress state.
- All execution and progress tracking lives exclusively in the work spec.

### Step 2: Implement Current Task
- Read the task requirements carefully
- **Refer to the Requirements Inventory**: Each subtask cites requirement IDs (e.g., `DOC-01`, `VALID-02`). Use these to find the full requirement text and ensure the implementation satisfies it.
- Implement all code changes needed
- Test for linter errors if applicable
- **Update work spec if implementation differs from plan** (API names, file locations, etc.)

### Step 2.5: Stop on Ambiguity
If you encounter ambiguity, incompleteness, or potential errors in the spec during implementation:
- **STOP immediately** and ask the human for clarification
- Do NOT guess or make assumptions about the intended behavior
- Do NOT continue implementing with a flawed understanding
- Explain the ambiguity clearly and propose options if applicable
- Wait for the human to resolve the ambiguity

### Step 3: Stop and Wait
- **DO NOT** remove 🟡 from Task N in the work spec
- **DO NOT** proceed to the next task
- **DO NOT** update current task status
- Simply stop and wait for user authorization

### Step 4: During Implementation
- When asked questions, do not make changes unless explicitly instructed
- Focus only on the current task requirements

---

## 📋 Summary Checklist

For each task implementation:
- [ ] Remove 🟡 from previous task (if applicable)
- [ ] Remove 🟡 from completed requirements in Requirements Inventory
- [ ] Remove 🟡 from completed APIs in API Inventory
- [ ] Refer to requirement IDs during implementation to ensure full coverage
- [ ] Implement current task fully  
- [ ] Update work spec if implementation changed from plan
- [ ] Stop and wait
- [ ] ❌ DO NOT remove 🟡 from current task

### Why This Rule Exists
The human controls task progression. Removing 🟡 from the current task would signal the work is finished and reviewed, which only the human can determine.

---

## Unit Testing Tasks

When a task involves implementing unit tests, follow the **Unit Testing Guide** (`Test/unit-testing-guide.md`) as a mandatory sub-process.

**Integration with task workflow:**
1. The work spec task defines *what* to test (e.g., "Write unit tests for TemplateRenderer")
2. The unit testing guide defines *how* to test (planning, infrastructure, writing phases)
3. When the unit testing sub-process completes, return to the work spec workflow

**Task completion criteria:**
- All test files planned and approved
- All sections implemented (no 🟡 remaining in test files)
- Work spec task can be marked complete

**Do NOT:**
- Skip the planning phase
- Deviate from the unit testing guide process without approval
- Mark the work spec task complete until all testing is done
