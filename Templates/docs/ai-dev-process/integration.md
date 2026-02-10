# AI Dev Process Integration

This document is the project-owned source of truth for operational details required by `ai-dev-process` workflows.

Do not store secrets here.

Conventions:
- Use 🟡 to mark any missing information that must be filled in for agents to run workflows deterministically.
- Prefer **non-interactive, command-line** commands over GUI instructions so:
  - humans can paste output back to agents
  - agents can reason deterministically about what ran

## Repositories and tooling

- **Primary IDE(s)**:
- **Languages/frameworks**:
- **CI**:
- 🟡 **Default destination (copy/pasteable `xcodebuild -destination` string, if applicable)**:
  - Example: `platform=iOS Simulator,name=<Device>,OS=latest`

## Build / compile

- 🟡 **Build/compile command(s)** (must be copy/pasteable)
- 🟡 **How to list targets/schemes/modules** (if applicable)
- 🟡 **How to choose a destination** (simulator/device/emulator), if applicable

## Unit tests

- 🟡 **How to run all unit tests** (command)
- 🟡 **How to run a single test / subset** (command)
- 🟡 **How to capture full output** (exact command + artifact paths, e.g. output file + result bundle)

## UI / instrumentation tests (if applicable)

- **How to run**:
- **Device/simulator requirements**:

## Simulator/device selection (if applicable)

- Preferred devices/OS versions:
- How to list/select destinations:

## Xcode/Swift (if applicable)

If this is an Xcode project, provide `xcodebuild`-based commands (not “open Xcode” steps).

- 🟡 **List schemes**:
  - `xcodebuild -list -project <path/to/App.xcodeproj>`
  - or `xcodebuild -list -workspace <path/to/App.xcworkspace>`

- 🟡 **Build (CLI, non-interactive)**:
  - Pattern:

    ```bash
    xcodebuild build \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      -destination '<DefaultDestination>' \
      2>&1 | tee <output-path>.txt
    ```

- 🟡 **Run unit tests (all)** (always capture full output + `.xcresult`) :
  - Pattern:

    ```bash
    rm -rf <result-bundle>.xcresult && \
    xcodebuild test \
      -scheme <Scheme> \
      -project <ProjectDir>/<Project>.xcodeproj \
      -destination '<DefaultDestination>' \
      -resultBundlePath <result-bundle>.xcresult \
      2>&1 | tee <output-path>.txt
    ```

- 🟡 **Run unit tests (subset)** (`-only-testing:`; always capture output + `.xcresult`) :
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

- 🟡 **Optional: noise filtering for human display** (do not lose the full output file):
  - Pattern:

    ```bash
    ... 2>&1 | tee <output-path>.txt | grep -E "^(✔|✘|•)|Assertion|failed|error:|threw"
    ```

- 🟡 **Extract assertion failures from `.xcresult`** (required when tests fail):
  - Pattern:

    ```bash
    xcrun xcresulttool get test-results tests \
      --path <result-bundle>.xcresult \
      --format json > <results-json-path>.json
    ```

  - Evidence contract: when tests fail, the human should provide:
    - `<output-path>.txt` (full output)
    - `<result-bundle>.xcresult` path
    - `<results-json-path>.json` (if extracted)

## Output and evidence collection (human-in-the-loop)

When the agent asks for evidence, provide:
- Full command output (or a path to captured output)
- Test result bundles/artifacts (if applicable)
- Crash logs / screenshots / traces as requested

Known limitations:
- (e.g., missing test log visibility; where to find output instead)

## Project structure notes

- Main targets/modules:
- Test targets/modules:
- Any special conventions needed for imports, fixtures, or test discovery:

## Work document conventions

- Work docs live under `work/<branch>/...`
- Testing work docs and artifacts live under `work/<branch>/testing/<suite>/...`

