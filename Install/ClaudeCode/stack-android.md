# Stack addendum: Android / Kotlin

This file is read by the Claude Code install runbook (`install-update-claudecode.md`) when the detected stack is Android/Kotlin.

## Integration doc guidance

When filling or updating the Integration doc (`docs/skai/integration.md`):

- Prefer non-interactive command-line commands (e.g., `./gradlew ...`) over GUI instructions. If you can't produce command-line commands with high confidence, leave 🟡 placeholders and ask.

## Testing stack defaults

Observed defaults from real Android projects (use as initial suggestions when seeding the Integration doc, but confirm with the developer):

- Unit tests: JUnit4 + MockK + kotlinx-coroutines-test (when coroutines are involved).
- Instrumentation tests: AndroidX JUnit + Espresso.
