# Error Handling Policy

This is a project policy describing how errors should be shaped, logged, and reported. It is intentionally implementation-agnostic. Code examples (if any) are illustrative only.

## Principles

### 1) One canonical project error type

Project-defined error types should conform to a single project error protocol/type.

Example naming (fictional theme): `LumenNotesError`.

### 2) Optional user-facing messaging

Error types may optionally provide a UI-facing representation for user messaging.

Example naming (fictional theme): `LumenNotesUIError`.

If no UI-facing message is provided, callers should use a default message appropriate for the UX.

### 3) Wrap third-party errors at the boundary

Third-party/library/framework errors should be wrapped/translated into the project error type at the boundary where they occur.

Use a standard wrapper type when defining a new domain error is not warranted.

### 4) Log at the source

Errors should be logged at the source where they originate (the boundary that wraps/translates them). Higher-level callers should not double-log.

### 5) Notability / reporting

Unexpected or undesirable errors should be marked as "notable" so they are sent to the project's error reporting/telemetry system. Expected errors (e.g., auth timeouts) are typically not notable.

### 6) Taxonomy

Use a consistent taxonomy/category list for error logging/reporting to support filtering and analysis.

Example taxonomy (format only):

- `"Module"`, `"Feature"`, `"Operation"`
- `"Module"`, `"Network"`, `"Request"`

## Project integration points

The project must define (in its own rules/policies):

- The canonical project error protocol/type name
- The UI-facing error/message representation (if used)
- The error reporting sink and what "notable" means
- The taxonomy conventions (roots/casing/allowed tags)
