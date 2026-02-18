# AI Dev Process Integration

This document is the project-owned source of truth for operational details required by `ai-dev-process` workflows.

Do not store secrets here.

<!-- Human-owned: edit freely. -->
## Special instructions / overrides

- 🟡 **Project-specific override rules**
  {human response}
  - INSTRUCTION: Add any project-specific override rules here that should influence how the installer manages the blocks below. Examples: preferred default destinations/devices; scheme/test-plan mapping rules; required flags (extra `xcodebuild` options, Gradle flags); evidence-capture expectations/limitations.
  - INSTRUCTION: If you have no overrides, delete the 🟡 marker and delete these instruction lines.

## Integration sections

<!-- BEGIN Managed-By: ai-dev-process | Section: required-values -->
## Required project integration values

### Repositories and tooling

- **Primary IDE(s)**: {human response}
- **Languages/frameworks**: {human response}
- 🟡 **CI**: {human response}
  - INSTRUCTION: Name the CI system if you have one (GitHub Actions, Buildkite, etc.), or write "none".

### Destinations (multi-platform supported)

- 🟡 **Default destinations** (project-wide constants; used by Xcode command templates)
  {human response}
  - INSTRUCTION: List one destination per platform you support (iOS / macOS / visionOS / etc.). The agent selects `<DefaultDestination>` based on task context.
  - Format: copy/pasteable `xcodebuild -destination` string.
  - Example:
    - iOS: `platform=iOS Simulator,name=<Device>,OS=latest`
    - macOS: `platform=macOS`
    - visionOS: `platform=visionOS Simulator,name=<Device>,OS=latest`

### Xcode container + scheme/test-plan conventions (if applicable)

- 🟡 **Xcode container**
  {human response}
  - INSTRUCTION: Specify the canonical build container as either a project or workspace path (choose one):
    - project: `<ProjectDir>/<Project>.xcodeproj`
    - workspace: `<Workspace>.xcworkspace`

- 🟡 **Scheme selection rule**
  {human response}
  - INSTRUCTION: Describe how to pick the scheme for "the module under work". If it's "scheme == module name", say so.

- 🟡 **Test plan rule** (if applicable)
  {human response}
  - INSTRUCTION: If you use test plans, state the rule (often "test plan name == scheme name"). Otherwise write "not used".

- **Mapping table** (optional; include only if rules aren't enough)
  - Example row: `Module: API` → `Scheme: AI` → `Test plan: AI` → `Test target: AITests`

### UI / instrumentation tests (if applicable)

- 🟡 **How to run UI/instrumentation tests**
  {human response}
  - INSTRUCTION: Provide the canonical command(s) or write "not used".

- 🟡 **Device/simulator requirements**
  {human response}
  - INSTRUCTION: State required devices/OS versions or special setup, or write "none".

### Output and evidence collection (human-in-the-loop)

Purpose: define the project-specific "evidence contract" so agents can debug deterministically when they cannot directly observe runtime behavior.

- 🟡 **Where command output is captured**
  {human response}
  - INSTRUCTION: Provide the canonical paths/naming (and whether the human should paste full content vs paste the path to the captured file).

- 🟡 **Test artifacts** (if applicable)
  {human response}
  - INSTRUCTION: Provide artifact path conventions for `.xcresult` and extracted JSON (or platform equivalents).

- 🟡 **Runtime/UI evidence**
  {human response}
  - INSTRUCTION: Where logs are found and what artifacts to provide (screenshots, recordings, crash reports).

- 🟡 **Known limitations**
  {human response}
  - INSTRUCTION: List any tooling limitations that affect evidence collection (e.g., missing test log visibility) and the workaround.

### Project structure notes

- Main targets/modules:
  {human response}
- 🟡 Test targets/modules:
  {human response}
  - INSTRUCTION: List test targets/modules so the agent can form `-only-testing:` selectors and choose schemes correctly.
- 🟡 Any special conventions needed for imports, fixtures, or test discovery:
  {human response}
  - INSTRUCTION: Document any repo-specific conventions that affect tests.

<!-- END Managed-By: ai-dev-process | Section: required-values -->

<!-- BEGIN Managed-By: ai-dev-process | Section: xcode -->
<!-- END Managed-By: ai-dev-process | Section: xcode -->

<!-- BEGIN Managed-By: ai-dev-process | Section: swift-package -->
<!-- END Managed-By: ai-dev-process | Section: swift-package -->

<!-- BEGIN Managed-By: ai-dev-process | Section: android-gradle -->
<!-- END Managed-By: ai-dev-process | Section: android-gradle -->

