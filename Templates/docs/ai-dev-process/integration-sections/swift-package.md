## Swift Package Manager

Provide `swift`-based commands for packages that use `Package.swift` (not `xcodebuild`).

### Build (CLI, non-interactive)

```bash
swift build \
  --package-path <PackagePath> \
  2>&1 | tee <output-path>.txt
```

### Run unit tests (all)

```bash
swift test \
  --package-path <PackagePath> \
  --xunit-output <results-path>.xml \
  2>&1 | tee <output-path>.txt
```

### Run unit tests (subset)

Entire test suite (class/struct):

```bash
swift test \
  --package-path <PackagePath> \
  --xunit-output <results-path>.xml \
  --filter <TestClass> \
  2>&1 | tee <output-path>.txt
```

Single test method:

```bash
swift test \
  --package-path <PackagePath> \
  --xunit-output <results-path>.xml \
  --filter '<TestClass>/<testMethod>' \
  2>&1 | tee <output-path>.txt
```

Multiple specific tests (pipe-separated):

```bash
swift test \
  --package-path <PackagePath> \
  --xunit-output <results-path>.xml \
  --filter '<TestClass>/<testMethod1>|<TestClass>/<testMethod2>' \
  2>&1 | tee <output-path>.txt
```

### Mandatory: filtered console output (while still saving full output)

Rule: always keep the full output file via `tee`. Filtering is only for terminal display.

```bash
... 2>&1 | tee <output-path>.txt | grep -E "passed|failed|error:|Test run"
```

### Mandatory when tests fail: extract assertion failures

`swift test` does not produce `.xcresult` bundles. Assertion details (expected vs actual values, error messages) are available in two places:

1. **XUnit XML** (`<results-path>.xml`): structured test results with failure messages.
2. **Console output** (`<output-path>.txt`): full output including assertion context.

To extract failure details from console output:

```bash
grep -A 5 "failed\|Expectation failed\|error:" <output-path>.txt
```

Evidence contract when tests fail — provide all of:
- `<output-path>.txt` (full output)
- `<results-path>.xml` (structured results)

### Destination discovery / fallback

`swift test` builds and runs for the host platform by default. No destination selection is needed.
