# Stack addendum: Xcode / Swift

This file is read by the Claude Code install runbook (`install-update-claudecode.md`) when the detected stack is Xcode/Swift.

## Experimental: Xcode MCP-hosted Claude Code

This stack addendum covers Xcode (26.3+) MCP-hosted Claude Code. The exact Xcode+MCP conventions may differ by environment. If any convention below doesn't match the host repo's setup, STOP and ask the human what to use.

## Integration doc guidance

When filling or updating the Integration doc (`docs/skai/integration.md`):

- Prefer non-interactive command-line commands (e.g., `xcodebuild ...`) over GUI instructions ("open Xcode..."). If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.
- **Never invent a simulator/device model.** If a canonical `xcodebuild -destination` string is not already established in-repo, propose one and ask the human to confirm before writing it.
- `xcodebuild -destination` strings must be canonical (from in-repo evidence or human-confirmed).
