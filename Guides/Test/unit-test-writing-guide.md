Managed-By: skai
Managed-Id: guide.unit-test-writing
Managed-Source: Guides/Test/unit-test-writing-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

# Unit Test Writing & Execution Guide

## Purpose

Defines the process for writing and executing test logic.

**Prerequisites:**
- Test plan approved
- Infrastructure ready (stubs, fixtures, utilities)

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After Phase 1 (Write Tests): present what was implemented and wait before running tests.
- During Phase 2 (Execute & Fix):
  - If tests did not run: STOP, surface the concrete error, and wait for advance intent after proposing the next attempt.
  - If you are about to change production code based on a failing test: propose the fix and STOP for approval.
  - When all tests pass: present results and STOP. Do not remove 🟡 markers until the human gives advance intent.

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

---

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

**Behavior:** Context determines the action:
- If waiting to proceed → remove 🟡 from the current phase (where applicable), advance to next phase
- If stopped due to ambiguities or unexpected challenges → resume where you left off

---

## Process Overview

**Two phases:**

1. **Phase 1: Write Tests** - Implement test logic
2. **Phase 2: Execute & Fix** - Run tests and fix failures

**Progress tracking:** 🟡 markers (see `Guides/Core/process-flow.md`) are used in two places:
- **Test file:** 🟡 on test functions and section MARKs
- **Work document:** 🟡 on phase checklist items

**Flow:**
1. Phase 1 (Write Tests) → checkpoint
2. Advance intent → Phase 2 (Execute & Fix) → iterate until all tests pass → checkpoint
3. Advance intent → remove 🟡 from passing tests → writing complete

**Missing infrastructure during writing:** If you discover a test requires infrastructure that was not provided during the infrastructure phase, do not create the infrastructure inline. Skip the test (mark it as blocked on infrastructure) and continue with the rest of the section. The human can re-enter the infrastructure phase later to address skipped tests.

**Two scenarios:**
- **Writing new tests**: Start at Phase 1
- **Fixing failing tests**: Start at Phase 2 (skip writing phase)

---

## Phases

### Phase 1: Write Tests

**Goal:** Implement test logic for all 🟡 tests in the section

**Checkpoint:** STOP after this phase and wait for advance intent.

### Phase 2: Execute & Fix

**Iterative:** May require multiple investigation/fix cycles

**After Phase 2:** When all tests pass, STOP and present results. Remove 🟡 markers only after advance intent.

---

## Phase 1: Write Tests

**🚨 CRITICAL REQUIREMENT: Every test MUST start with `defer { Container.shared.reset() }` 🚨**

**Symptom of missing defer:**
- Individual tests pass when run alone
- Tests fail when run together as a suite
- = State pollution between tests

### Process

**What to do:**

**0. Create work document** (per `Guides/Core/working-doc-conventions.md`):
   - **Subpath**: `testing/<suite-name>`
   - **File name**: `<section-name>-writing.md`
   - **Full path**: `working-docs/<branch-path>/testing/<suite-name>/<section-name>-writing.md`
   - `<suite-name>` = test file name without "Tests.swift" (e.g., `TemplateRenderer` from `TemplateRendererTests.swift`)
   - `<section-name>` = section name in kebab-case (e.g., `success-tests`, `error-handling`)
   - **Example**: `working-docs/work/feature-branch/testing/TemplateRenderer/success-tests-writing.md`
   - **Structure**:
     ```markdown
     # [Suite Name] - [Section Name] Tests - Writing & Execution
     
     ## Context
     This document tracks the writing and execution work for implementing unit tests.
     
     **Source:** [path to source file]
     **Tests:** [path to test file with line range]
     **Infrastructure:** [link to infrastructure work document]
     
     ## Checklist
     
     - Phase 1: Write Tests 🟡
     - Phase 2: Execute & Fix 🟡
     
     ## Phase 1: Write Tests
     
     [Implementation details will go here]
     ```

1. **Review project testing conventions**
  - See `docs/skai/integration.md` for project-specific requirements
   - Check for dependency injection patterns, test suite structure, and test infrastructure

2. **Review approved infrastructure**
   - Know which stubs, fixtures, and helpers are available
   - Understand how to configure them

3. **Implement test logic**
   - **🚨 FIRST LINE of EVERY test: `defer { Container.shared.reset() }`**
   - **If adding to existing section**: Only implement 🟡 tests
   - **If new section**: Implement all 🟡 tests
   - Follow Arrange → Act → Assert pattern
   - Use approved stubs and fixtures
   - **Keep 🟡 on test functions** - they remain until tests pass in Phase 2

4. **Add supporting code** (if needed)
   - Constants (test values, URLs, etc.)
   - Variables (shared test state)
   - Helpers (shared test setup logic)
   - **Check existing first** to avoid duplication

5. **Ensure test isolation**
   - Each test must run independently
   - No shared mutable state between tests
   - Use Factory container registration per test
   - Clean up with `defer { Container.shared.reset() }` at the start of each test
   - **Never use `deinit`** - it's not actor-isolated and can cause data races

6. **Document in work document**
   - List all implemented tests
   - Note any helpers or constants added
   - Describe test approach

7. **STOP - Checkpoint**

**Important:** Do NOT run tests yet. Wait for advance intent.

**Note:** 🟡 remains on test functions during Phase 1. They are only removed in Phase 2 when tests pass.

---

### Test Structure Pattern

**Use Swift Testing framework (not XCTest) for all new tests**

**Every test requires both:**
- `///` doc comment -- explains what the test verifies and how (for code review)
- `@Test("...")` display name -- concise behavior sentence for test navigator and CI output (not a duplicate of the function name)

**🚨 CRITICAL: EVERY test MUST start with `defer { Container.shared.reset() }` 🚨**

```swift
import Testing
import Factory
@testable import ModuleName

@Suite(.serialized) @MainActor
struct MyComponentTests {

    /// Verifies that a successful operation returns the expected value.
    /// Stubs the dependency to return a success response.
    @Test("Returns expected value on success") func testSomething() async throws {
        // 🚨 REQUIRED: Clean up at end of test (before registering dependencies)
        defer { Container.shared.reset() }
        
        // Arrange - Set up dependencies and state
        let stubDataProvider = StubDataProvider()
        let stubService = StubService()
        
        Container.shared.dataProvider.register { stubDataProvider }
        Container.shared.service.register { stubService }
        
        // Configure stubs
        stubDataProvider.responses = [
            .success(expectedData)
        ]
        stubService.currentState = .active
        
        let sut = SystemUnderTest()
        
        // Act - Execute the behavior being tested
        let result = try await sut.performAction()
        
        // Assert - Verify expected outcomes (use #expect, not XCTest assertions)
        #expect(result.value == expectedValue)
        #expect(stubDataProvider.requests.count == 1)
    }
}
```

**Why defer is required:**
- Individual tests may pass but fail when run together without it
- Prevents state pollution between tests
- Must be FIRST line in test body (before any other code)
- **Tests that pass individually but fail in a suite = missing defer**

---

### Using Stubs

**Stub configuration:**
```swift
// Component stub
let stubComponent = ComponentStub()
stubComponent.nextResult = .success(expectedResult)

// Service mock
let mockService = MockService()
await mockService.setState(.active)

// Another service mock
let mockAnotherService = MockAnotherService()
```

**Stub verification:**
```swift
// Verify interactions
let calls = await stubComponent.calls
#expect(calls.count == 2)

// Access with await if stub is an actor
#expect(mockService.operationCallCount == 1)
```

---

### Using Fixtures

**Always use fixtures for test data:**
```swift
// Good
let entity = EntityFixtures.valid
let data = DataFixtures.complete
let config = ConfigFixtures.default

// Bad - Don't create ad-hoc data
let entity = Entity(id: "123", name: "Test", ...)
```

**Search for existing fixtures first:**
```bash
# Find existing fixtures
grep -r "struct.*Fixtures" path/to/fixtures/
```

---

### Testing Async Code

**Configure stubs to return async results:**
```swift
// Configure stub with multiple responses
let stubComponent = ComponentStub()
stubComponent.responses = [
    .success(firstResult),
    .failure(ComponentError.failure),
    .success(secondResult)
]

// Test async behavior
let result1 = try await sut.performOperation()  // Gets first response
let result2 = try await sut.performOperation()  // Gets second response  
let result3 = try await sut.performOperation()  // Gets third response
```

**Observe publisher changes:**

See `docs/skai/integration.md` for project-specific test utilities (e.g., a `PublisherSpy` equivalent), if any.

```swift
let spy = PublisherSpy(viewModel.$state)

let value = try await spy.execute {
    await viewModel.performAction()
}

#expect(value == .active)
```

---

### Testing Concurrent Operations

**Use Task for concurrent execution:**
```swift
// Launch concurrent operations
let task1 = Task { try await sut.request1() }
let task2 = Task { try await sut.request2() }

// Wait for completion
let result1 = try await task1.value
let result2 = try await task2.value

// Verify both succeeded
#expect(result1.value == expected1)
#expect(result2.value == expected2)
```

---

### Test Organization

**Follow type-organization pattern with MARK sections:**
```swift
@Suite(.serialized) @MainActor
struct MyComponentTests {
    
    // MARK: - Success Tests
    
    @Test("Returns result on basic success") func testBasicSuccess() async throws {
        defer { Container.shared.reset() }
        // Test logic...
    }
    
    @Test("Handles complex multi-step success") func testComplexSuccess() async throws {
        defer { Container.shared.reset() }
        // Test logic...
    }
    
    // MARK: - Error Tests
    
    @Test("Propagates network error") func testNetworkError() async throws {
        defer { Container.shared.reset() }
        // Test logic...
    }
    
    @Test("Rejects invalid input") func testValidationError() async throws {
        defer { Container.shared.reset() }
        // Test logic...
    }
    
    // MARK: - Constants
    
    struct TestValue: Codable, Equatable {
        let id: String
        let name: String
    }
    
    // MARK: - Variables
    
    // (Shared test state if needed - use `let` for immutable state)
    
    // MARK: - Helpers
    
    private func makeTestRequest() -> TestRequest {
        TestRequest(/* ... */)
    }
}
```

**Key principles:**
- Use `struct` (not `class`) for test suites - simpler and no reference counting
- Each test gets a fresh copy of the suite
- Test sections come first
- Then Constants, Variables, Helpers
- Group related tests under descriptive MARK comments
- **No `deinit`** - use `defer { Container.shared.reset() }` in each test instead

---

## Phase 2: Execute & Fix

**Triggered by:** Advance intent after Phase 1

**On advance intent:** marks Phase 1 complete in work document, then begins Phase 2.

### Process

**What to do:**

1. **Run the tests** using standard process (see "Running Tests" section below)
   - Use the project's test command from `docs/skai/integration.md`
   - Save full output to `working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt`

**CRITICAL VALIDATION - DO NOT SKIP:**

2. **Verify the test actually ran** - IMMEDIATELY after running the command:
   - **Check output file size**: `ls -lh working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt`
     - If file is 0 bytes or very small (< 1KB): **TESTS DID NOT RUN**
   - **Read the output file**: `cat working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt`
     - Look for test execution indicators (✔, ✘, Test Suite, etc.)
     - If you see no test execution output: **TESTS DID NOT RUN**
   - Confirm any additional expected artifacts described in `docs/skai/integration.md` were produced.
   
   **If tests did not run:**
   - **STOP**: Do NOT proceed to analyze code or make guesses
   - **Investigate why**: Read the FULL output file to find the actual error
   - Common causes:
     - Build failures (compilation errors, missing dependencies)
     - Simulator not available
     - Invalid test target/class/method name
     - Package resolution failures
   - Document the actual error in work document
   - Fix the error
   - Re-run the test command
   - Repeat validation until test actually runs
   - **If you cannot determine why tests didn't run**: Document what you tried and consult with human

   **Only proceed to step 3 when you can confirm:**
   - Output file is substantial (several KB minimum)
   - Output file contains test execution indicators

3. **Document results** in work document
   - Command used
   - Output file location
   - Output file size (proves test ran)
   - Exit code
   - Pass/fail status

4. **If all tests pass:**
   - Document success
   - STOP at checkpoint (present results and wait for advance intent)
   - After advance intent: remove 🟡 from Phase 2 in the work document, from passing test functions, and from section MARK comment (if all tests in section pass)
   - Section complete!

5. **If tests fail:**
   - Keep all 🟡 markers in place (do not remove from passing tests mid-iteration)
   - Document failures in work document
   - Follow maintenance workflow (see below)
   - Re-run after fixes
   - Repeat until all pass

---

### Running Tests

See `docs/skai/integration.md` for project-specific test execution commands.

**Standard test execution process:**

1. **Create output directory** (per `Guides/Core/working-doc-conventions.md`):
```bash
   mkdir -p working-docs/<branch-path>/testing/<suite-name>
   ```

2. **Run tests with full output logging**:
   - **CRITICAL: Run ONLY the tests in the current section** - Do NOT run the entire test file (noise from unimplemented tests).
   - Use the project's test command from `docs/skai/integration.md` and follow its recommended way to scope to specific tests.
   - Persist full output to `working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt`.
   - Persist any additional artifacts described in `docs/skai/integration.md`.

3. **Analyze failures**:
   - **Primary**: Extract structured assertion failure details using the process in `docs/skai/integration.md`
   - **Secondary**: Search output file for additional context:
```bash
     grep -A 10 "failed\|error" working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt
```

4. **Next test run** overwrites the same files (keep paths stable per section)

**Key points:**
- Always scope runs to the current section's tests (see `docs/skai/integration.md`)
- Full output enables multiple analyses without re-running tests
- Branch-based organization keeps test outputs organized
- See `docs/skai/integration.md` for the stack-specific runner setup and artifact extraction

---

### When Tests Fail

**Investigation → Fix → Verify (repeat until passing)**

When resolving test failures, follow the project's Debugging / Problem-Resolution Guide (installed with `skai`):
- See `Guides/Core/debugging-guide.md` for the evidence-first loop and approval gates.
- Select a debugging tactic (e.g., partitioning, minimal working implementation, bisect) and state *why* it's the best next step.
- Prefer experiments that produce discriminating evidence over "guessing fixes."
- Use explicit stop conditions: if you need runtime output you can't access, pause and ask the human to run tests and provide the output/logs.

**Process:**
1. **Extract structured assertion failure details** - This is CRITICAL for understanding what failed
   - See `docs/skai/integration.md` for the command/process to extract assertion failures in this project/stack
   - Look for:
     - Expected vs actual values in assertion failures
     - Error messages and thrown errors
     - Specific line numbers where tests failed
   
2. **Add debug logging if needed**:
   - Use `print("AIDEV: ...")` (or the project's chosen debug prefix) for test debug output
   - The `AIDEV:` prefix allows easy filtering: `grep "AIDEV" <output-file>`
   - Example:
     ```swift
     print("AIDEV: Published values: \(publishedValues)")
     print("AIDEV: Expected: \(expected), Actual: \(actual)")
     ```
   
3. **Search test output file** for additional context:
   ```bash
   grep -A 10 "failed\|error" working-docs/<branch-path>/testing/<suite-name>/<section-name>-test-output.txt
   ```
   
4. **Document the failure** in work document:
   - Which assertion failed (line number)
   - Expected value
   - Actual value
   - Error message if thrown
   
5. **Investigate root cause**:
   - **Start with production code** - Review the logic under test FIRST
   - Read the test code and understand what it's verifying
   - Compare expected vs actual values from assertion failure
   - Check for common patterns below
6. **Propose fix** - Document the proposed solution in work document and STOP
7. **Wait for approval / advance intent** - Human reviews and approves before you apply the fix (end checkpoint output with the standard gate line; see `Guides/Core/process-flow.md`)
8. **Apply fix** - Make the approved changes
9. **Re-run test** - Verify it passes
10. **Repeat if needed** - If still failing, return to step 1 (extract assertion details)

**Critical debugging principles:**
- **Production code first** - Check the code under test before assuming test infrastructure is broken
- **Trust isolation results** - If a test fails in complete isolation with consistent values, the bug is in production code
- **Simple explanations first** - Logic bugs are more likely than framework issues
- **Listen to evidence** - When user provides contradictory evidence, stop and reconsider your hypothesis
- **Facts vs guesses** - Never state assumptions as facts. Clearly label: Facts (what logs/code show), Hypothesis (what you think), Unknown (what needs verification)

**When to ask for human determination:**
- **Business rules/domain questions**: Ask human about product behavior, UX decisions, timing rules, notification preferences
- **Obvious bugs**: Fix without asking - logic errors, wrong variables, crashes, off-by-one errors

**How to report issues requiring human determination:**

Document in work document with:
1. Brief context (what scenario you're testing)
2. What the production code does vs what the test expects
3. Simple question asking which is correct

**Example:**
```markdown
### Decision Required

**Issue:** I'm testing the scenario where `processCompleted()` sends a completion 
notification after data is processed. The production code sends this notification 
with `option: .includeMetadata` (line 194), but the tests expect `.excludeMetadata`. Which is correct?

**Waiting for determination before proceeding.**
```

**Common failure patterns:**
- **Logic errors in production code**: Incorrect calculations, wrong variable used, off-by-one errors
- **Race conditions**: Data races in concurrent code → Add synchronization (actors, locks)
- **Stub misconfiguration**: Not enough responses queued → Check stub response configuration
- **State pollution**: Test passes alone, fails in suite → Verify `defer { Container.shared.reset() }` in each test
- **Wrong assumptions**: Test expects behavior that doesn't exist → Update test or fix production code

**Update test vs production code:**
- Update production code if test reveals a real bug or recent changes broke functionality
- Update test if assumptions are wrong, behavior intentionally changed, or test is too brittle
- Always discuss with human before deciding which to change

---

## Testing Principles

### Test Suite Structure & Isolation

See `docs/skai/integration.md` for:
- Testing framework requirements
- Test suite declaration patterns
- Serialization requirements
- Dependency injection patterns

**CRITICAL: Use `@Suite(.serialized) @MainActor struct`**

```swift
import Testing
import Factory
@testable import ModuleName

@Suite(.serialized) @MainActor
struct MyComponentTests {
    @Test func testSomething() async throws {
        // Clean up at end of test
        defer { Container.shared.reset() }
        
        // Register stubs per-test
        Container.shared.someService.register { StubService() }
        
        // Use #expect() for assertions
        #expect(value == expected)
    }
}
```

**Key Points:**
- Use `struct` (not `class`) - simpler, value semantics, no reference counting
- Must use `@MainActor` to ensure all test code runs on main actor
- Must use `.serialized` to prevent DI conflicts between tests
- **Never use `deinit`** - it's not actor-isolated and causes data races with reactive code
- Use `defer { Container.shared.reset() }` at the start of each test for cleanup
- `defer` guarantees cleanup even if test throws
- Per-test registration for isolation
- See README for complete details

---

### Dependency Injection in Tests

See `docs/skai/integration.md` for:
- Factory framework usage
- Container registration patterns
- Test-specific DI patterns

**Per-test registration pattern:**
```swift
@Test func testSomething() async throws {
    // Clean up at end of test
    defer { Container.shared.reset() }
    
    // Arrange - Create and register stubs locally
    let stubService = StubService()
    let stubRepository = StubRepository()
    
    Container.shared.service.register { stubService }
    Container.shared.repository.register { stubRepository }
    
    // Configure stubs
    stubService.currentState = .active
    stubRepository.responses = [.success(data)]
    
    // Act & Assert
    let sut = SystemUnderTest()
    // ...
}
```

**Key points:**
- Add `defer { Container.shared.reset() }` at the start of each test
- Create and register stubs within each test
- No global stub instances
- `defer` ensures cleanup even if test throws

---

## Common Patterns

### Testing Error Paths

**Use `#expect(throws:)` for error assertions:**

```swift
/// Verifies that errors are properly propagated
@Test func testHandlesError() async throws {
    defer { Container.shared.reset() }
    
    let stubComponent = ComponentStub()
    stubComponent.responses = [
        .failure(ComponentError.operationFailed)
    ]
    
    Container.shared.component.register { stubComponent }
    
    #expect(throws: ComponentError.self) {
        try await sut.performOperation(with: parameters)
    }
}
```

### Testing State Transitions

```swift
/// Verifies that state changes trigger UI updates
@Test func testStateTransition() async throws {
    defer { Container.shared.reset() }
    
    let spy = PublisherSpy(viewModel.$isActive)
    
    // Initial state
    #expect(viewModel.isActive == false)
    
    // Trigger state change
    let finalValue = try await spy.execute {
        await viewModel.performAction()
    }
    
    // Verify transition
    #expect(finalValue == true)
}
```

### Testing Concurrent Requests

```swift
/// Verifies that concurrent operations don't interfere with each other
@Test func testConcurrentOperations() async throws {
    defer { Container.shared.reset() }
    
    let stubComponent = ComponentStub()
    stubComponent.responses = [
        .success(firstResult),
        .success(secondResult)
    ]
    
    Container.shared.component.register { stubComponent }
    let sut = ComponentService()
    
    let task1 = Task { try await sut.performOperation(id: "1") }
    let task2 = Task { try await sut.performOperation(id: "2") }
    
    let result1 = try await task1.value
    let result2 = try await task2.value
    
    #expect(result1.id == "1")
    #expect(result2.id == "2")
    
    let requests = await stubComponent.requests
    #expect(requests.count == 2)
}
```

---

## Checklist Quick Reference

**Work Document Structure:**

Full path: `working-docs/<branch-path>/testing/<suite-name>/<section-name>-writing.md` (per `Guides/Core/working-doc-conventions.md`)

```markdown
# [Suite Name] - [Section Name] Tests - Writing & Execution

## Context
This document tracks the writing and execution work for implementing unit tests.

**Source:** [path to source file]
**Tests:** [path to test file with line range]
**Infrastructure:** [link to infrastructure work document]

## Checklist

- Phase 1: Write Tests 🟡
- Phase 2: Execute & Fix 🟡

## Phase 1: Write Tests

[Implementation details]

## Phase 2: Execute & Fix

[Test results, failures, investigations, fixes]
```

**Phase 1: Write Tests**
- Create work document
- Review available infrastructure
- Implement test logic (Arrange → Act → Assert)
- **Keep 🟡 on test functions** (removed in Phase 2 when they pass)
- Add supporting code if needed
- Ensure test isolation
- Document in work document
- When approved: remove 🟡 from "Phase 1: Write Tests" in the work document

**Phase 2: Execute & Fix**
- Remove 🟡 from Phase 1 in the work document (if not already done)
- Run tests scoped to the current section's tests (per `docs/skai/integration.md`)
- **CRITICAL: Verify test actually ran** (check file size + output contains test execution indicators)
- Document results
- If failures:
  - Extract structured assertion details (CRITICAL) (see `docs/skai/integration.md`)
  - Document expected vs actual values
  - Investigate and fix (see maintenance workflow)
- Re-run until all pass
- When all pass: STOP at checkpoint (present results, wait for advance intent)
- After advance intent: remove 🟡 from test functions, section MARK, and Phase 2 in work document
- If previously passing test fails: Restore 🟡 to that test function

---

## Quick Reference

**Test Structure:**
- Arrange → Act → Assert
- Use stubs and fixtures
- Ensure isolation

**Test Execution:**

See `docs/skai/integration.md` for project-specific test execution commands.

**Standard process:**
- Create suite output directory: `working-docs/<branch-path>/testing/<suite-name>/` (per `Guides/Core/working-doc-conventions.md`, e.g., `working-docs/work/step-refactor/testing/TemplateRenderer/`)
- Run tests scoped to ONLY the current section's tests (per `docs/skai/integration.md`)
- Persist artifacts/logs as described in `docs/skai/integration.md`
- **Extract structured assertion failures** - CRITICAL first step (see `docs/skai/integration.md`)
- Search output file for additional context
- Overwrite both files on subsequent runs

**Key Principles:**
- Use `struct` for test suites (not `class`) - simpler, value semantics
- Mark with `@Suite(.serialized) @MainActor`
- **Never use `deinit`** - it's not actor-isolated and causes data races
- Use `defer { Container.shared.reset() }` at start of each test for cleanup
- Register stubs per-test
- Wait for advance intent before running tests

**Test execution validation:**
- After running test command, IMMEDIATELY verify test actually ran
- Check output file size (should be several KB) and output contains test indicators (plus any expected artifacts from `docs/skai/integration.md`)
- If test didn't run: STOP, investigate why, fix, re-run (do NOT guess or analyze code)

**When tests fail:**
1. Extract structured assertion details (CRITICAL) (see `docs/skai/integration.md`)
2. Document expected vs actual values
3. Investigation → Fix → Verify (repeat until passing)
- See "When Tests Fail" section above for complete process
