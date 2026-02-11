# AI Dev Process Integration

This document is the project-owned source of truth for operational details required by `ai-dev-process` workflows.

Do not store secrets here.

Conventions:
- Use 🟡 to mark any missing information that must be filled in for agents to run workflows deterministically.
- Prefer **non-interactive, command-line** commands over GUI instructions so:
  - agents can run them autonomously when they have terminal access
  - if a human must run them, output/artifacts can be captured and pasted back reliably

## Installer setup tasks (do these first)

1. 🟡 **Confirm the default build/test destination** (if applicable)
   - The installer should propose a destination string and ask the human to confirm (do not invent device models from memory).
   - Fill in “Default destination” under “Repositories and tooling” with a copy/pasteable destination string (used by all `xcodebuild` commands).

2. 🟡 **Fill in canonical build + test commands (copy/pasteable)**
   - The installer should infer these from CI/docs when possible, otherwise ask the human for the minimal missing values.
   - Fill in “Build / compile” and “Unit tests” with the exact commands the agent should run (or the human should run when asked), including:
     - full output capture (output file path)
     - result bundle paths (e.g., `.xcresult`) and any extraction steps (e.g., `xcresulttool`)

## Repositories and tooling

- **Primary IDE(s)**:
- **Languages/frameworks**:
- **CI**:
- 🟡 **Default destinations** (project-wide constants used by command templates)
  - Purpose: avoid the agent guessing destinations (and avoid “works on my machine” drift).
  - Provide one default destination per supported platform you care about (iOS / macOS / visionOS / etc.).
  - The agent should pick the correct default destination based on the task’s platform context.
  - Format: copy/pasteable `xcodebuild -destination` string.
  - Example:
    - iOS: `platform=iOS Simulator,name=<Device>,OS=latest`
    - macOS: `platform=macOS`
    - visionOS: `platform=visionOS Simulator,name=<Device>,OS=latest`

## Command template variables (agent-filled)

The command templates below use placeholders like `<Scheme>` and `<TestTarget>`.

- These are **variables**, not “project TODOs”.
- The agent should usually infer them from context (what module/code is under test, where the tests live, existing CI scripts), then proceed.
- If any variable is unclear, the agent must ask the human for clarification before running commands.

Common variables:
- `<DefaultDestination>`: selected from “Default destinations” above for the platform under work (then referenced by command templates).
- `<Scheme>`: the scheme that builds/tests the module under work.
- `<TestPlan>` (if used): the test plan to run (often matches the scheme/module name).
- `<TestTarget>`, `<TestClass>`, `<testMethod>`: used for `-only-testing:` selectors.
- `<ProjectDir>/<Project>.xcodeproj` or `<Workspace>.xcworkspace`: the build container.
- `<output-path>.txt`, `<result-bundle>.xcresult`, `<results-json-path>.json`: artifact paths (follow the process docs; document any project-specific overrides below).

## Scheme / test plan conventions (Xcode, if applicable)

If the project uses multiple schemes and/or test plans, define the convention here so agents don’t guess.

- 🟡 **Scheme selection rule** (e.g., “scheme name == module name”; “API module uses scheme `AI`”):
- 🟡 **Test plan rule** (if applicable) (e.g., “test plan name == scheme name”):
- 🟡 **Mapping table** (optional; include only if rules aren’t enough):
  - Example row: `Module: API` → `Scheme: AI` → `Test plan: AI` → `Test target: AITests`

## Build / compile

- 🟡 **Build/compile command(s)** (must be copy/pasteable)

## Unit tests

- 🟡 **How to run all unit tests** (command)
- 🟡 **How to run a single test / subset** (command)
- 🟡 **How to capture full output** (exact command + artifact paths, e.g. output file + result bundle)

## UI / instrumentation tests (if applicable)

- **How to run**:
- **Device/simulator requirements**:

## Destination discovery / fallback (if applicable)

Purpose: if the default destination isn’t available on the machine (or you need a different platform), use `showdestinations` to pick an available destination, then update “Default destinations” above.

- **How to list destinations** (standard `xcodebuild` procedure):
  - `xcodebuild -showdestinations -scheme <Scheme> -project <ProjectDir>/<Project>.xcodeproj`
  - or `xcodebuild -showdestinations -scheme <Scheme> -workspace <Workspace>.xcworkspace`

## Xcode/Swift (if applicable)

If this is an Xcode project, provide `xcodebuild`-based commands (not “open Xcode” steps).

- **List schemes**:
  - `xcodebuild -list -project <path/to/App.xcodeproj>`
  - or `xcodebuild -list -workspace <path/to/App.xcworkspace>`

- **Build (CLI, non-interactive)**:
  - Pattern:

    ```bash
    xcodebuild build \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      -destination '<DefaultDestination>' \
      2>&1 | tee <output-path>.txt
    ```

- **Run unit tests (all)** (always capture full output + `.xcresult`) :
  - Pattern:

    ```bash
    rm -rf <result-bundle>.xcresult && \
    xcodebuild test \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      # If you use test plans, add: -testPlan <TestPlan> \
      -destination '<DefaultDestination>' \
      -resultBundlePath <result-bundle>.xcresult \
      2>&1 | tee <output-path>.txt
    ```

- **Run unit tests (subset)** (`-only-testing:`; always capture output + `.xcresult`) :
  - Entire class:

    ```bash
    rm -rf <result-bundle>.xcresult && \
    xcodebuild test \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      -destination '<DefaultDestination>' \
      -resultBundlePath <result-bundle>.xcresult \
      -only-testing:<TestTarget>/<TestClass> \
      2>&1 | tee <output-path>.txt
    ```

  - Single test method (quote + include `()` when required by the test framework):

    ```bash
    rm -rf <result-bundle>.xcresult && \
    xcodebuild test \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      -destination '<DefaultDestination>' \
      -resultBundlePath <result-bundle>.xcresult \
      -only-testing:'<TestTarget>/<TestClass>/<testMethod>()' \
      2>&1 | tee <output-path>.txt
    ```

**Mandatory: filtered console output (while still saving full output)**

- Rule: always keep the full output file via `tee`. Filtering is only for terminal display.
- Pattern:

    ```bash
    ... 2>&1 | tee <output-path>.txt | grep -E "^(✔|✘|•)|Assertion|failed|error:|threw"
    ```

**Mandatory when tests fail: extract assertion failures from `.xcresult`**

- Action: after any failing test run that produced `<result-bundle>.xcresult`, run this command to extract the detailed failure messages (expected vs actual, line numbers, thrown errors) into a JSON file.
- Pattern:

    ```bash
    xcrun xcresulttool get test-results tests \
      --path <result-bundle>.xcresult \
      --format json > <results-json-path>.json
    ```

  - Evidence contract: when tests fail, provide:
    - `<output-path>.txt` (full output)
    - `<result-bundle>.xcresult` path
    - `<results-json-path>.json` (required)

## Output and evidence collection (human-in-the-loop)

Purpose: define the project-specific “evidence contract” so agents can debug deterministically when they cannot directly observe runtime behavior.

When the agent asks for evidence, provide the items below in the format/locations your project uses:

- 🟡 **Where command output is captured**:
  - e.g., `work/<branch>/.../*.txt` paths, naming conventions, and whether to paste content vs paste a file path

- 🟡 **Test artifacts** (if applicable):
  - `.xcresult` bundle path conventions
  - `xcresulttool` extraction JSON path conventions

- 🟡 **Runtime/UI evidence** (when the agent can’t run/interact with the app):
  - where to find logs (Xcode console, OSLog export, device logs)
  - what to provide (screenshot, screen recording, repro steps, crash report)

Known limitations:
- 🟡 (e.g., missing test log visibility; where to find output instead)

## Project structure notes

- Main targets/modules:
- Test targets/modules:
- Any special conventions needed for imports, fixtures, or test discovery:

