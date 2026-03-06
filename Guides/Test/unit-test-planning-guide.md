Managed-By: skai
Managed-Id: guide.unit-test-planning
Managed-Source: Guides/Test/unit-test-planning-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-04

# Unit Test Planning Guide

## Purpose

Defines the process for creating complete test plans with ALL test stubs and documentation across ALL sections before any implementation begins.

## Checkpoints

This guide follows the shared process-flow mechanics in `Guides/Core/process-flow.md` (checkpoints, advance intent, `auto`, and the standard gate line).

Workflow-specific gate points (this guide must STOP and wait at these checkpoints):
- After planning is complete (all files/sections/tests are stubbed and marked 🟡).
- When ambiguities block planning (missing product intent, unclear expected behavior, unclear public API to test).

At checkpoints, end checkpoint output with the standard gate line (see `Guides/Core/process-flow.md`).

---

## Advance intent

Advance intent (and `auto`) semantics are defined in `Guides/Core/process-flow.md`.

**Behavior:** Context determines the action:
- If waiting to proceed → mark planning complete, return to parent process
- If stopped due to ambiguities or unexpected challenges → resume where you left off

---

## The Planning Process

**Goal:** Create test plan with stubs and detailed doc comments

**Output:** One or more test files with sections and tests marked 🟡 to indicate TODO

**Four scenarios:**

1. **New Test Suite** - Creating test file from scratch
   - Create test file with suite declaration
   - Define ALL MARK sections
   - **Add 🟡 emoji to all section MARK comments** (marking sections as TODO)
   - Add ALL test stubs across ALL sections
   - **Add 🟡 emoji to each test function** (marking individual tests as TODO)
   - Add detailed doc comments for every test

2. **Adding New Sections** - Adding new sections to existing test file
   - Open existing test file
   - Define new MARK sections being added
   - **Add 🟡 emoji to new section MARK comments** (marking sections as TODO)
   - Add all test stubs for the new sections only
   - **Add 🟡 emoji to each new test function** (marking individual tests as TODO)
   - Add detailed doc comments for every new test
   - Existing sections and tests remain unchanged

3. **Adding Tests to Existing Section** - Adding tests within an existing section
   - Open existing test file
   - Locate the existing MARK section
   - **Add 🟡 emoji to section MARK comment if not already present** (marking section as TODO)
   - Add new test stubs within the section
   - **Add 🟡 emoji to each new test function** (marking individual tests as TODO)
   - Add detailed doc comments for every new test
   - Existing tests in section remain unchanged

4. **Marking Failing Tests** - Identifying existing tests that need fixing
   - Open existing test file
   - Locate the failing test functions
   - **Add 🟡 emoji to each failing test function** (marking them as needing fixes)
   - **Add 🟡 emoji to section MARK comment if not already present** (marking section as TODO)
   - Present marked tests for approval

**Multi-File Planning:**
When planning tests for multiple types:
- Create ALL test files in a single planning phase
- Each file follows the same structure (sections with 🟡, test stubs with 🟡)
- Present complete list of all planned files at checkpoint
- Implementation proceeds file-by-file after planning approval

**CRITICAL:**
- Do all planning upfront (all files, all sections, all tests)
- Mark sections with 🟡 to indicate they contain TODO work
- Mark individual test functions with 🟡 to indicate they need implementation or fixes
- Do NOT create new work documents during planning
- **Only make changes that qualify as "test planning"** - nothing else

**What qualifies as test planning:**
- ✅ Create new test file with suite declaration (new suite only)
- ✅ Add new MARK sections with 🟡
- ✅ Add new test stubs with 🟡 and `Issue.record()`
- ✅ Add 🟡 to existing section MARK if adding tests to it
- ✅ Add doc comments to new test stubs

**What does NOT qualify as test planning:**
- ❌ Renaming existing tests
- ❌ Refactoring existing test code
- ❌ Changing `struct` to `class` or other structural changes
- ❌ Modifying cleanup patterns
- ❌ Reorganizing existing MARK sections
- ❌ Any change to existing code

**Key principle:** Test stubs with detailed doc comments ARE the plan

### Creating the Complete Test Plan

**CRITICAL:** The doc comments in the code ARE the test plan. They must be sufficient for review and approval.

**Planning creates:**

**For New Test Suite:**
1. Test file with ALL test stubs across ALL sections
2. ALL MARK sections defined **with 🟡 emoji** (marking sections as TODO)
3. ALL test functions **with 🟡 emoji** (marking individual tests as TODO)
4. Detailed doc comments for every test
5. Complete picture of test coverage

**For Adding New Sections:**
1. New MARK sections in existing test file **with 🟡 emoji** (marking sections as TODO)
2. Test stubs for new sections only **with 🟡 emoji** (marking individual tests as TODO)
3. Detailed doc comments for every new test
4. Existing sections and tests remain unchanged

**For Adding Tests to Existing Section:**
1. New test stubs in existing section **with 🟡 emoji** (marking individual tests as TODO)
2. Section MARK comment **with 🟡 emoji if not already present** (marking section as TODO)
3. Detailed doc comments for every new test
4. Existing tests in section remain unchanged

**For Marking Failing Tests:**
1. Failing test functions **marked with 🟡 emoji** (indicating need for fixes)
2. Section MARK comment **with 🟡 emoji if not already present** (marking section as TODO)


**All tests require:**
- Descriptive camelCase name
- **`@Test("...")` display name** -- concise behavior sentence for test navigator and CI output (not a duplicate of the function name)
- **Concise, but sufficient doc comment** that explains **what** the test verifies and **how** it verifies it (enough for human review/approval).
  - Prefer 1-3 short lines; add more only when needed for clarity.
- **Body with `Issue.record("Test not yet implemented")` to mark as failing TODO**

**Doc Comment Guidelines:**
- Keep it concise, but sufficient to describe what the test verifies and how it verifies it
- Be specific about the behavior being validated; include key setup only if it is not obvious
- Make it clear enough for human review and approval

**Creating Complete Test Plan:**

**For New Test Suite:**
1. Create test file with suite declaration
2. Define ALL MARK sections for logical grouping
3. **Add 🟡 emoji to all MARK sections** (marking sections as TODO)
4. Add ALL test stub functions across ALL sections
5. **Add 🟡 emoji to each test function** (marking individual tests as TODO)
6. Add detailed doc comments for every test
7. Add standard sections: Constants, Variables, Helpers

**For Adding New Sections:**
1. Open existing test file
2. Define new MARK sections for new functionality
3. **Add 🟡 emoji to new MARK sections** (marking sections as TODO)
4. Add test stub functions for new sections only
5. **Add 🟡 emoji to each new test function** (marking individual tests as TODO)
6. Add detailed doc comments for every new test
7. Existing sections and helpers remain unchanged

**For Adding Tests to Existing Section:**
1. Open existing test file
2. Locate the existing MARK section
3. **Add 🟡 emoji to section MARK comment if not already present** (marking section as TODO)
4. Add new test stub functions within the section
5. **Add 🟡 emoji to each new test function** (marking individual tests as TODO)
6. Add detailed doc comments for every new test
7. Existing tests in section remain unchanged

**For Marking Failing Tests:**
1. Open existing test file
2. Locate the failing test functions
3. **Add 🟡 emoji to each failing test function** (marking them as needing fixes)
4. **Add 🟡 emoji to section MARK comment if not already present** (marking section as TODO)

**Example (Swift Testing framework):**
```swift
import Testing
import Factory
@testable import YourModule

@Suite(.serialized) @MainActor
struct ComponentTests {

    // MARK: - Success Tests 🟡
    
    /// Verifies that a successful operation returns the expected value.
    /// Stubs the dependency to return a success response.
    /// Ensures the component properly processes the response.
    @Test("Returns expected value on success") func testOperationReturnsSuccessResponse() async throws { // 🟡
        Issue.record("Test not yet implemented")
    }
    
    /// Verifies that required parameters are passed correctly.
    /// Creates an operation with specific parameters and inspects the captured values.
    /// Validates that all parameters are included correctly.
    @Test("Passes required parameters to dependency") func testOperationPassesParameters() async throws { // 🟡
        Issue.record("Test not yet implemented")
    }
    
    // MARK: - Error Handling Tests 🟡
    
    /// Verifies that error conditions throw appropriate errors.
    /// Stubs dependency to return error condition. Ensures component throws appropriate error.
    @Test("Throws appropriate error on failure") func testOperationThrowsOnError() async throws { // 🟡
        Issue.record("Test not yet implemented")
    }
    
    // MARK: - Constants
    
    // MARK: - Variables
    
    // MARK: - Helpers
}
```

**Assertions - Use `#expect()` (not XCTest macros):**
```swift
#expect(value == expected)       // Equality
#expect(value != unexpected)      // Inequality  
#expect(value > 0)                // Comparison
#expect(optionalValue == nil)     // Nil check
#expect(throws: MyError.self) {   // Error throwing
    try somethingThatThrows()
}
```

---

## Testing Principles

### Test Public APIs, Not Private Implementation

**CRITICAL: Unit tests should test behavior through PUBLIC interfaces**

**Rules:**
1. **Only test public and internal methods** - Never create tests for `private` methods
2. **Private methods are tested indirectly** - They're exercised through the public methods that call them
3. **Focus on behavior, not implementation** - Test what the code does, not how it does it
4. **Don't expose internals for testing by default** - In most cases, indirect testing through public APIs is sufficient
5. **Exception: Request visibility change when needed** - If testing a private method through public APIs is problematic, you can request it be made `internal` for testing

**Why:**
- Tests should validate contracts and behavior, not implementation details
- Private methods can be refactored freely if tests focus on public APIs
- Indirect testing ensures private methods work in their actual usage context
- Exposing internals for testing creates artificial coupling

**Planning Check:**
Before creating test stubs for a method, verify:
1. ✅ Is it `public` or `internal`? → Create tests
2. ❌ Is it `private`? → Check if it's tested through public APIs
   - If adequately tested indirectly → Do NOT create tests
   - If testing through public APIs is problematic → Request it be made `internal` for testing

**When to request visibility change:**
- Complex logic that's difficult to trigger through public APIs
- Need for comprehensive edge case testing that's impractical via public methods
- Method has critical business logic that warrants direct testing
- **Document the reason** in your work document when requesting the change

**Example:**
```swift
class ComponentUnderTest {
    // ✅ Test this directly - it's the public API
    public func processData() async throws -> Result {
        let validated = validateInput()  // private helper
        return try await transform(validated)  // private helper
    }
    
    // ❌ Don't test these directly - they're private
    private func validateInput() -> ValidationResult { ... }
    private func transform(_ data: ValidationResult) async throws -> Result { ... }
}

// Tests should only call processData()
// validateInput() and transform() are tested indirectly through processData()
```

**During Planning:**
- When reviewing the source code to plan tests, check method visibility
- Only create test stubs for public/internal methods
- Trust that private methods will be adequately tested through public APIs

---

### Avoid Frivolous and Redundant Tests

**CRITICAL: Every test must validate actual logic, not language features or redundancy**

**Frivolous tests to avoid:**

1. **Testing protocol conformance** - The compiler guarantees this
   ```swift
   // ❌ BAD: Testing that a protocol method returns the required type
   @Test func testErrorConvertibleReturnsAIError() async throws {
       let error = CustomError.failure
       #expect(error.asAIError is AIError)  // Compiler already guarantees this
   }
   ```

2. **Testing trivial property accessors** - No logic to test
   ```swift
   // ❌ BAD: Testing a property that's just optional chaining
   var currentOut: Output? { currentVersionedOut?.output }
   
   @Test func testCurrentOutMatchesVersionedOut() {
       // This just tests Swift's optional chaining - no actual logic
   }
   ```

3. **Testing standard library/framework behavior** - Already tested by Apple/Swift
   ```swift
   // ❌ BAD: Testing that Combine's .map() works
   var versions: Publisher { versionedOut.map { $0?.versions } }
   
   @Test func testVersionsPublisherExtractsVersions() {
       // This just tests Combine's .map() - no actual logic
   }
   ```

4. **Testing implementation details** - Tests private internals, not behavior
   ```swift
   // ❌ BAD: Testing internal state cleanup
   @Test func testContinuationCleanedUpAfterEmission() {
       // versionedOutContinuation is a private implementation detail
   }
   ```

5. **Testing the inverse of another test** - Redundant coverage
   ```swift
   // ❌ BAD: Testing "not stale" when you already test "is stale"
   @Test func testIsStaleDetectsChangedInputs() { ... }    // ✅ Good
   @Test func testIsStaleIsFalseWhenUnchanged() { ... }    // ❌ Redundant
   ```

6. **Testing already-covered behavior** - Duplicate coverage
   ```swift
   // ❌ BAD: Both test version incrementing
   @Test func testPerformIncrementsVersion() {
       // Verifies version 0, 1, 2
   }
   @Test func testVersionNumberStartsCorrectly() {
       // Verifies version starts at 0 - already covered above
   }
   ```

**What TO test:**

✅ **Business logic** - Actual decisions and transformations
✅ **State management** - Complex state transitions and side effects
✅ **Error handling** - Error capture, propagation, and recovery
✅ **Integration behavior** - How components work together
✅ **Edge cases** - Boundary conditions and unusual inputs
✅ **Observable behavior** - What callers can see and rely on

**Planning checklist to avoid frivolous tests:**

Before creating a test stub, ask:
1. ❓ **Does this test actual logic?** If it's just calling a getter, probably no.
2. ❓ **Could this test fail if I changed the implementation?** If only breaking changes would fail it, it's testing the type system.
3. ❓ **Is this already covered by another test?** Check for redundancy.
4. ❓ **Am I testing my code or the framework?** Don't test Swift/Combine/Foundation.
5. ❓ **Is this testing observable behavior or implementation details?** Stick to public contracts.

**Examples of good vs bad tests:**

```swift
// ✅ GOOD: Tests actual lock behavior logic
@Test func testLockedReturnsExistingOutput() {
    // Tests that .locked prevents re-execution - real business logic
}

// ❌ BAD: Tests trivial accessor
@Test func testCurrentOutReturnsOutputFromVersionedOut() {
    // currentOut is just: currentVersionedOut?.output
    // No logic - just optional chaining
}

// ✅ GOOD: Tests error capture and storage logic
@Test func testPerformCapturesError() {
    // Tests actual error handling behavior
}

// ❌ BAD: Tests protocol conformance
@Test func testCustomErrorConvertsToAIError() {
    // Protocol conformance is a compile-time guarantee
}

// ✅ GOOD: Tests version incrementing logic
@Test func testPerformIncrementsVersion() {
    // Verifies versions: 0, 1, 2 - tests actual logic
}

// ❌ BAD: Redundant with above
@Test func testVersionNumberStartsCorrectly() {
    // Just checks version 0 - already covered above
}
```

---

### Test Suite Declaration

**Use Swift Testing framework (not XCTest) for all new test suites**

**Framework imports:**
```swift
import Testing  // ✅ Use Swift Testing
@testable import ModuleName

@Suite(.serialized) @MainActor
struct MyComponentTests {
    @Test func testSomething() async throws {
        defer { Container.shared.reset() }
        // Use #expect() for assertions
    }
}
```

**Key points:**
- Use `struct` for test suites
- Use `defer { Container.shared.reset() }` at the start of each test that registers dependencies
- `.serialized` prevents DI override conflicts between tests
- `@MainActor` ensures tests run on main actor when needed

---

## Test Organization

### File Structure

See `docs/skai/integration.md` for:
- Test target organization
- File naming conventions
- Test file structure

### Within Test Files
Use MARK comments to organize (following code-organization pattern):

```swift
// MARK: - Send Tests (or just "Tests", or multiple sections for finer grouping)
// MARK: - Constants
// MARK: - Variables  
// MARK: - Helpers
```

**Note**: Test sections come FIRST, before Constants/Variables/Helpers

### Test Naming
- **Function names**: camelCase (e.g., `testOperationReturnsSuccessResponse`)
- **Display names**: `@Test("...")` with a concise behavior sentence (e.g., `@Test("Returns expected value on success")`)
- **Documentation**: Plain English doc comment explaining the test
- **Example**:
  ```swift
  /// Verifies that a successful operation returns the expected value
  @Test("Returns expected value on success") func testOperationReturnsSuccessResponse() async throws {
      // TODO
  }
  ```

### Test Scope
- Flexible - whatever makes sense for the scenario
- Can be one assertion or multiple related assertions
- Can be small focused tests or longer scenario/integration tests

---

## Tips for Good Test Plans

### Coverage Considerations
- Think about happy paths AND error paths
- Consider edge cases and boundary conditions
- Don't duplicate coverage - each test should verify something unique
- Group related tests into logical MARK sections

### Naming Conventions
- Use `test` prefix
- Use descriptive verb + object pattern: `testOperationReturnsSuccessResponse`
- Be specific: `testErrorConditionTriggersRetry` not `testHandlesError`

### Documentation
- Doc comments should explain what the test verifies (and briefly how it verifies it if not obvious)
- Keep it concise, but sufficient for human review/approval
- Focus on behavior, not implementation details

---

## Checklist Quick Reference

**Creating Complete Test Plan:**
- Identify all functional areas to test
- Define MARK sections for logical grouping
- Create stub functions with descriptive names for ALL tests
- Add detailed doc comments for every test
- Leave all bodies empty or with `// TODO`
- Follow MARK organization pattern
- Present complete plan to human
- Wait for human to choose which section to start

---

## Quick Reference

**Planning Output:**
- **New Suite:** Create ALL test stubs across ALL sections with 🟡 emoji on sections and tests, `Issue.record()` in body
- **Adding New Sections:** Create test stubs for new sections only with 🟡 emoji on sections and tests, `Issue.record()` in body
- **Adding Tests to Section:** Create test stubs within existing section with 🟡 emoji on tests (and section if needed), `Issue.record()` in body
- **Marking Failing Tests:** Mark existing failing tests with 🟡 emoji (and section if needed)
- Detailed doc comments for every test
- Organized by MARK sections
- `Issue.record("Test not yet implemented")` in stub bodies to mark as failing TODO

**Test Suite Structure (new files only):**
- Use `@Suite(.serialized) @MainActor struct`
- Use `defer { Container.shared.reset() }` in tests that register dependencies
- Test sections first, then Constants, Variables, Helpers

**Only make changes that qualify as "test planning":**
- Adding new sections and test stubs = planning ✅
- Renaming, refactoring, restructuring = NOT planning ❌
- Structural changes to existing tests = NOT planning ❌
