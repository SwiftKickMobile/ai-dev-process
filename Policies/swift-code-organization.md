# Swift Code Organization (Swift-only)

This is a stack-specific policy intended for Swift codebases.

## General organization

These principles apply to all code. Type organization rules (below) supersede these when applicable.

### Priority ordering

Higher priority code above lower priority code.

- Functional statements before logging/diagnostics
- Core logic before edge case handling
- Return paths before cleanup code

### Subordination ordering

Superior code above subordinate code.

- Primary types before helper types (e.g., a view before its subviews)
- Callers before callees (if function A calls function B, A appears before B)
- Public API before private implementation

### Access level ordering

Non-private code above private code.

- Public/internal types before private/fileprivate types
- Public/internal extensions before private extensions

---

## Type organization

Within each type, use `MARK` comments to organize code into sections.

### Standard sections (required in order, even if empty)

```swift
// MARK: - API
// MARK: - Constants
// MARK: - Variables
```

1. API is for the type's interface members--properties, methods, initializers, nested types, etc. that callers interact with (regardless of access level, but excludes private implementation details and protocol conformances)
2. Constants is for private items determined at compile time, constant properties, nested types, typealiases, etc.
3. Variables is for properties determined at runtime, including `let` reference types, which are mutable.

### Optional sections (preferred order)

```swift
// MARK: - Lifecycle      // Overridden methods (viewDidLoad, deinit, etc.)
// MARK: - Configuration  // Setup code (Combine chains, observers, etc.)
// MARK: - <functionality> // Private implementation grouped by purpose
// MARK: - <ProtocolName>  // Protocol conformance implementations
```

### Enforcement rules

- All standard sections required even if empty
- 100% API types exempt from sections
- `MARK` comments must have blank line after
- Use `// MARK: - <name>` format exactly

