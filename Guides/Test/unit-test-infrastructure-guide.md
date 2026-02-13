Managed-By: ai-dev-process
Managed-Id: guide.unit-test-infrastructure
Managed-Source: Guides/Test/unit-test-infrastructure-guide.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-08

# Unit Test Infrastructure Guide

## Purpose

Defines the process for identifying, implementing, and compiling test infrastructure.

**Infrastructure includes:**
- Production code abstractions (protocols/wrappers for testability)
- Test doubles (stubs implementing protocols)
- Test data (fixtures)
- Test framework utilities (shared helpers)

---

## Commands

### Next Command

**Definition:** Any of `"begin"`, `"next"`, or `"continue"` — these are synonymous.

**Behavior:** Context determines the action:
- If waiting to proceed → remove 🟡 from the current phase (where applicable), advance to next phase
- If stopped due to ambiguities or unexpected challenges → resume where you left off

---

## Process Overview

**Three phases:**

1. **Phase 1: Identify** - Research and document infrastructure needs
2. **Phase 2: Implement** - Create the infrastructure (if needed)
3. **Phase 3: Compile** - Ensure everything builds (if implemented)

**Progress tracking:** Work document checklist uses 🟡 for TODO phases, and no marker for completed phases (remove 🟡 when complete)

**Flow:**
1. Phase 1 (Identify) → STOP at checkpoint
2. Next Command → **Branching:**
   - If no infrastructure needed → STOP at checkpoint (no document needed)
   - If infrastructure exists → STOP at checkpoint (no document needed)
   - If new infrastructure needed → Create work document → Execute Phase 2 (Implement)
3. After Phase 2 → Automatically execute Phase 3 (Compile)
4. After Phase 3 → Remove 🟡 from Phase 2 & 3 (in the work document) → STOP at checkpoint

**Possible outcomes:**
- No infrastructure needed → STOP at checkpoint after Phase 1 (no document)
- Infrastructure exists → STOP at checkpoint after Phase 1 (no document)
- New infrastructure needed → STOP at checkpoint after Phase 3 (document tracks work)

---

## Phases

### Phase 1: Identify

**Goal:** Determine exactly what infrastructure is needed for the tests

**Checkpoint:** STOP after this phase and wait for Next Command

### Phase 2: Implement

**Runs only if:** Infrastructure changes needed (determined in Phase 1)

### Phase 3: Compile

**Runs only if:** Phase 2 ran

**Checkpoint:** STOP after this phase and wait for Next Command

---

## Phase 1: Identify

### Critical Mindset

**Your role is to enable tests, NOT decide whether to test:**

❌ **Wrong thinking:**
- "SSL pinning can't be unit tested, should we skip these tests?"
- "This requires too much refactoring, let's remove these tests"
- "This needs integration tests, not unit tests"

✅ **Correct thinking:**
- "SSL pinning requires abstracting certificate validation. Here are 3 approaches..."
- "Testing this requires refactoring ProductionClass to inject DependencyX. Here's the proposed change..."
- "Current architecture makes this hard to test. Should we: A) Add abstraction layer, or B) Refactor to dependency injection?"

**Always propose HOW to test, never whether to test.**

---

### Process

**What to do:**

1. **START WITH STRATEGY**
   - Create "Overview / Strategy" section explaining the testing approach
   - Describe high-level plan before listing details

2. **SEARCH FIRST - MANDATORY**
   - **NEVER assume infrastructure exists**
   - Use `grep`, `glob_file_search`, or `codebase_search` to find files
   - Use `read_file` to verify capabilities of found files
   - **Document search commands used and results found**
   - If file not found: Mark as "NEW (does not exist)"
   - If file found: Mark as "Already exists at [path]" and list its current capabilities

3. **CHECK FOR EXISTING TESTS**
   - Review existing infrastructure already in use
   - Prefer reusing over creating new

4. **ANALYZE REQUIREMENTS**
   - What dependencies does the component inject?
   - Are there hard dependencies (like URLSession) that need abstraction?
   - What test data is needed?
   - Are there repetitive patterns for test framework?

5. **BE DEFINITIVE - NO ASSUMPTIONS**
   - Never say "new or existing" or "if available"
   - You must KNOW which it is through verified search
   - Every “Already exists” claim must be backed by a search command that found the file
   - **Every "NEW" must be backed by a search command that returned no results**

6. **BE SPECIFIC**
   - For existing items: List exact new methods/properties needed
   - For new items: List exact API surface
   - Include file paths and current capabilities

7. **DOCUMENT EVERYTHING**
   - Write analysis in work document
   - Organize by infrastructure category
   - **Include search commands and results in work document as proof**

**Then:**
- **If no infrastructure needed**: Document "No infrastructure changes required"
- **If infrastructure needed**: Document all needed changes

**Checkpoint:** Present analysis and STOP. Wait for Next Command.

---

### Infrastructure Categories

#### 1. Production Code Abstractions

**Purpose:** Make production code testable

**Examples:**
- Protocol wrappers around Apple APIs (URLSession → NetworkSession)
- Dependency injection refactoring
- Certificate validation abstractions

**When needed:**
- Testing code with hard dependencies on frameworks
- Code that directly uses Apple APIs
- Code tightly coupled to specific implementations

#### 2. Test Doubles (Stubs)

**Purpose:** Implement protocols for dependency injection in tests

**Requirements:**
- Implement the protocol being stubbed
- Support controllable responses for async operations
- Provide tracking properties for verification (e.g., `var requests: [Request] = []`)
- Support flexible configuration for different test scenarios

**Location and Naming:**

See `docs/ai-dev-process/integration.md` for the project’s test infrastructure locations and naming conventions.

#### 3. Fixtures

**Purpose:** Reusable test data

**Requirements:**
- Static properties on a struct
- Meaningful, realistic data
- Complete objects when possible

**Location and Naming:**

See `docs/ai-dev-process/integration.md` for the project’s fixture locations and naming conventions.

#### 4. Test Framework Utilities

**Purpose:** Shared testing helpers that reduce boilerplate

**Requirements:**
- Only depends on Core module
- Well-documented with usage examples
- Generic and flexible where appropriate

**Location and Examples:**

See `docs/ai-dev-process/integration.md` for the project’s test utility locations and available utilities.

**When to propose:**
- Pattern appears in multiple test suites
- Reduces significant boilerplate
- Enables consistent testing approach
- Solves a common testing problem

**Always propose to human before implementing.**

---

### Example: Good Infrastructure Analysis

```markdown
## Phase 1: Identify

### Overview / Strategy

To test `ComponentUnderTest` without making real external calls, we need to abstract 
third-party dependencies behind our own protocol that supports dependency injection.

**Approach**: Protocol Wrapper Pattern
- Create `ComponentProtocol` protocol that mirrors the third-party API we use
- Make the third-party type conform to it (production code continues using real implementation)
- Inject `ComponentProtocol` into `ComponentUnderTest` via dependency injection
- Create `ComponentStub` for tests with controllable responses

This allows us to control responses, simulate errors, and verify behavior without 
touching external dependencies.

---

### Production Code Abstractions

**Search performed:**
```bash
grep -r "protocol ComponentProtocol" path/to/source/
# Result: No matches found
```

**1. ComponentProtocol protocol** - **NEW (does not exist)**
- Location: `Module/Component/ComponentProtocol.swift`
- Purpose: Abstraction over third-party dependency to enable dependency injection
- Definition: `func operation(parameters: Parameters) async throws -> Result`

**2. ThirdPartyType extension** - **NEW**
- Same file as ComponentProtocol
- Purpose: Make real third-party type conform so production code works unchanged
- Empty conformance: `extension ThirdPartyType: ComponentProtocol {}`

**3. ComponentUnderTest** - **Modify existing** at `Module/Component/Component.swift`
- Current (line 82): `let dependency = ThirdPartyType(configuration: .default)`
- Add: `@Injected(\.componentDependency) var componentDependency` property
- Use injected dependency instead of creating new one
- Add Factory registration returning default implementation

### Stubs

**Search performed:**
```bash
glob_file_search "**/ComponentStub.swift"
# Result: No files found
```

**1. ComponentStub** - **NEW (does not exist)**
- Location: `Module/PreviewContent/Stubs/ComponentStub.swift`
- Conforms to: `ComponentProtocol` protocol
- Required API:
  - `func operation(parameters: Parameters) async throws -> Result`
  - `var calls: [Parameters] { get }` - tracks all calls made
  - Properties/methods to configure responses for testing

**Search performed:**
```bash
read_file path/to/Mocks/MockService.swift
# Found at specified path
```

**2. MockService** - **Already exists** at `path/to/Mocks/MockService.swift`
- ✅ Has: `state` publisher, `performAction()` method
- ➕ Need to add: tracking properties for retry attempts to support retry flow tests

### Fixtures

**1. Entity fixtures** - **NEW** (test-only helper)
- Location: In test file Constants section or `Module/PreviewContent/Fixtures/EntityFixtures.swift`
- Purpose: Pre-configured entities for testing
- Example: `static let valid = Entity(id: "test-id-123")`

**2. TestRequest** - **NEW** (test-only helper)
- Location: In test file Constants section
- Simple Request implementation for testing
- Uses codable `TestResponse` struct

### Test framework

- **No new utilities needed** - will use existing shared test utilities

---

### Summary

**Infrastructure needed:**
- 1 new protocol (ComponentProtocol)
- 1 protocol conformance (ThirdPartyType)
- 1 modified production class (ComponentUnderTest)
- 1 new stub (ComponentStub)
- 1 modified mock (MockService - add tracking properties)
- 2 new test helpers (EntityFixtures, TestRequest in test file)

**Ready to proceed to Phase 2?**

**Checkpoint:** Present complete analysis and STOP. Wait for Next Command.

**After Next Command:**
- Remove 🟡 from Phase 1 in the work document (if you created one)
- If infrastructure needed: Proceed to Phase 2
- If no infrastructure needed: Infrastructure process complete (no Phase 2/3 work document required)
```

---

## Phase 2: Implement

**This phase only runs if infrastructure changes are needed.**

**Triggered by:** Next Command after Phase 1 (when infrastructure is needed)

### Process

**What to do:**

**0. Create work document:**
   - **File name**: `<branch-name>/testing/<suite-name>/<section-name>-infrastructure.md`
   - `<suite-name>` = test file name without "Tests.swift" (e.g., `TemplateRenderer` from `TemplateRendererTests.swift`)
   - `<section-name>` = section name in kebab-case (e.g., `success-tests`, `error-handling`)
   - **Example**: `work/feature-branch/testing/TemplateRenderer/success-tests-infrastructure.md`
   - Note: `<branch-name>` includes the `work/` prefix (e.g., `work/feature-branch`)
   - **Structure**:
     ```markdown
     # [Suite Name] - [Section Name] Tests - Infrastructure
     
     ## Context
     This document tracks the infrastructure work for implementing unit tests for [description].
     
     **Source:** [path to source file]
     **Tests:** [path to test file with line range]
     
     ## Checklist
     
     - Phase 1: Identify
     - Phase 2: Implement 🟡
     - Phase 3: Compile 🟡
     
     ## Phase 1: Identify
     
     [Copy analysis from Phase 1 output]
     
     ## Phase 2: Implement
     
     [Implementation details will go here]
     ```

**1. Create infrastructure work spec** (if complex, otherwise implement directly):
   - **File name**: `<branch-name>/testing/<suite-name>/<section-name>-infrastructure-spec.md`
   - **Example**: `work/feature-branch/testing/TemplateRenderer/success-tests-infrastructure-spec.md`
   - Note: `<branch-name>` includes the `work/` prefix (e.g., `work/feature-branch`)
   - **Structure**: Motivation, Functional Requirements, Relevant Files, Task List
   - **Content**: Describe all stubs, mocks, fixtures, and production code changes needed
   - **No code**: Describe what to implement, not how

2. **Document in work document** what infrastructure was created

3. **Work through the spec with human:**
   - Human will say "Start task N" or "Proceed to next task"
   - Implement each task following the project’s code organization policy (stack-specific)
   - Remove 🟡 from tasks as you complete them (do not use ✅)
   - Continue until all infrastructure tasks are complete

4. **Proceed to Phase 3** (Compile)

### Infrastructure Locations

See `docs/ai-dev-process/integration.md` for:
- Stub/mock locations and naming
- Fixture locations and naming
- Test utility locations
- Production code abstraction guidelines

---

## Phase 3: Compile

**This phase only runs if infrastructure was implemented in Phase 2.**

### Process

**What to do:**

1. **Create "Compilation" subsection** in work document under infrastructure section

2. **Run the build command** to compile the framework target
   
   See `docs/ai-dev-process/integration.md` for project-specific build commands.

3. **Document compilation results** in work document

4. **If there are errors:**
   - List all compilation errors
   - Fix them
   - Re-run compilation
   - Repeat until clean

5. **If no errors:** Document success with exit code 0

6. **Complete:** Infrastructure ready for use

7. **Mark phases complete:** Remove 🟡 from Phase 2 and Phase 3 in the work document

8. **Done:** Infrastructure process complete

---

## Stub Implementation Guidelines

### Structure

```swift
import Testing  // For test doubles used in Swift Testing tests

public class ComponentStub: ComponentProtocol {
    // MARK: - ComponentProtocol
    
    public func operation(parameters: Parameters) async throws -> Result {
        calls.append(parameters)
        return try await nextResponse()
    }
    
    // MARK: - API
    
    // Public properties for test configuration and verification
    
    // MARK: - Variables
    
    public private(set) var calls: [Parameters] = []
    public var responses: [Result] = []
    private var responseIndex = 0
    
    // MARK: - Lifecycle
    
    public init() {}
    
    // MARK: - Helpers
    
    private func nextResponse() async throws -> Result {
        defer { responseIndex += 1 }
        return responses[responseIndex]
    }
}
```

### Key Principles

- **Track all interactions**: Store parameters in arrays for verification
- **Support controllable responses**: Allow tests to configure results and errors
- **Public configuration**: Allow tests to configure behavior
- **Thread-safe if needed**: Use `actor` for concurrent access
- **Follow code-organization**: Protocol conformance first, then API, then internal

---

## Fixture Implementation Guidelines

### Structure

```swift
import Foundation

public struct EntityFixtures {
    public static let valid = Entity(
        id: "entity-123",
        name: "Valid Entity",
        createdDate: Date(),
        owner: "Owner Name",
        status: .active
    )
    
    public static let inactive = Entity(
        id: "entity-456",
        name: "Inactive Entity",
        createdDate: Date(),
        owner: "Another Owner",
        status: .inactive
    )
}
```

### Key Principles

- **Static properties**: Easy to access in tests
- **Meaningful data**: Realistic and complete
- **Multiple variants**: Different scenarios (valid, invalid, edge cases)
- **Reusable**: Useful across multiple test suites

---

## Common Patterns

### Protocol Wrapper Pattern

**Problem:** Need to test code that uses Apple framework directly (URLSession, FileManager, etc.)

**Solution:**
1. Create protocol matching the API you need
2. Make framework type conform via extension
3. Inject protocol instead of concrete type
4. Create stub implementing protocol for tests

**Example:**
```swift
// 1. Protocol
protocol ComponentProtocol {
    func operation(parameters: Parameters) async throws -> Result
}

// 2. Conformance
extension ThirdPartyType: ComponentProtocol {}

// 3. Injection
class ComponentUnderTest {
    @Injected(\.componentDependency) var componentDependency
}

// 4. Stub
class ComponentStub: ComponentProtocol { /* ... */ }
```

### Controllable Async Responses Pattern

**Problem:** Need controllable async responses in tests

**Solution:** Configure stubs with queued responses

**Example:**
```swift
// Setup
let stubComponent = ComponentStub()
stubComponent.responses = [
    .success(firstResult),
    .success(secondResult),
    .failure(ComponentError.failure)
]

Container.shared.componentDependency.register { stubComponent }
let sut = ComponentUnderTest()

// Test
let result1 = try await sut.performOperation(params: params1)  // Gets first response
let result2 = try await sut.performOperation(params: params2)  // Gets second response
let result3 = try await sut.performOperation(params: params3)  // Gets third response (throws)
```

---

## Checklist Quick Reference

**Work Document Structure** (created in Phase 2 only if infrastructure needed):

File name: `<branch-name>/testing/<suite-name>/<section-name>-infrastructure.md`

```markdown
# [Suite Name] - [Section Name] Tests - Infrastructure

## Context
This document tracks the infrastructure work for implementing unit tests for [description].

**Source:** [path to source file]
**Tests:** [path to test file with line range]

## Checklist

- Phase 1: Identify
- Phase 2: Implement 🟡
- Phase 3: Compile 🟡

## Phase 1: Identify

[Analysis content]

## Phase 2: Implement

[Implementation details]

## Phase 3: Compile

[Compilation results]
```

**Phase 1: Identify**
- Create "Overview / Strategy" section
- **MANDATORY: Search for each infrastructure item**
- **MANDATORY: Verify files exist with `read_file` before marking as existing**
- Analyze what's needed
- Document all requirements by category with search proof
- Present analysis and wait for Next Command

**Phase 2: Implement** (only if infrastructure needed)
- Create work document (only now, not in Phase 1)
- Create infrastructure work spec (if complex)
- Implement all tasks
- Mark tasks complete

**Phase 3: Compile** (if Phase 2 ran)
- Build framework
- Fix compilation errors
- Verify clean build
- Remove 🟡 from Phase 2 & 3

---

## Quick Reference

**Infrastructure Types:**
1. Production Code Abstractions - Make code testable
2. Stubs - Test doubles for DI
3. Fixtures - Reusable test data
4. Test Framework Utilities - Shared helpers

**Locations:**

See `docs/ai-dev-process/integration.md` for all infrastructure locations and naming conventions.

**Command:**
- Next Command - Removes 🟡 from the previously completed phase (where applicable) and proceeds to next phase
- Phase 1 → STOP (checkpoint)
- After approval → Remove 🟡 from Phase 1 (if present in a work document)
- Branch: No infrastructure → Done | Infrastructure needed → Phase 2 & 3
- After Phase 3 → Remove 🟡 from Phase 2 & 3 → Done

**Progress Tracking:**
- Work document checklist: 🟡 = TODO, no marker = complete
- Within content sections: Prefer “Already exists” vs “NEW” (avoid ✅/❌ completion markers)

