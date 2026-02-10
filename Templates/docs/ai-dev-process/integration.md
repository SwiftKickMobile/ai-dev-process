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
- 🟡 **Build** (`xcodebuild ... build`):
- 🟡 **Run unit tests (all)** (`xcodebuild ... test`):
- 🟡 **Run unit tests (subset)** (`-only-testing:...`):
- 🟡 **Result bundle + full output capture** (`-resultBundlePath ...` + redirect/tee output):

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

