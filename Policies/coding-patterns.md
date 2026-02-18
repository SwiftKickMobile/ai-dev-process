# Coding Patterns (Core)

This is a language-agnostic set of coding patterns intended to keep APIs simple, code readable, and changes easy to review.

## Avoid one-time-use variables

Don't introduce local variables used only once unless the expression is complex or the name meaningfully improves readability.

## Organize by priority and call hierarchy

Prefer a consistent ordering that helps readers understand the "main story" first.

- Higher-priority functions above lower-priority/supporting functions
- Callers above callees (if A calls B, A appears above B)
- Core business logic before analytics/logging/cleanup
- Primary state changes before secondary side effects

## Don't make callers think

Design APIs so callers don't have to replicate state checks at every call site.

Prefer:
- Idempotent operations (safe to call even if already in the desired state)
- "No-op if not applicable" behavior when appropriate

Avoid:
- Requiring callers to check internal state before invoking an API unless there's a strong reason

