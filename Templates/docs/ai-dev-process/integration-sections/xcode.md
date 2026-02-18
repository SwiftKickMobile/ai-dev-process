## Xcode/Swift

Provide `xcodebuild`-based commands (not "open Xcode" steps).

### List schemes

- `xcodebuild -list -project <path/to/App.xcodeproj>`
- or `xcodebuild -list -workspace <path/to/App.xcworkspace>`

### Build (CLI, non-interactive)

```bash
xcodebuild build \
  -scheme <Scheme> \
  -project <ProjectDir>/<Project>.xcodeproj \
  -destination '<DefaultDestination>' \
  2>&1 | tee <output-path>.txt
```

### Run unit tests (all)

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

### Run unit tests (subset)

Entire class:

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

Single test method (quote + include `()` when required by the test framework):

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

### Mandatory: filtered console output (while still saving full output)

Rule: always keep the full output file via `tee`. Filtering is only for terminal display.

```bash
... 2>&1 | tee <output-path>.txt | grep -E "^(✔|✘|•)|Assertion|failed|error:|threw"
```

### Mandatory when tests fail: extract assertion failures from `.xcresult`

Action: after any failing test run that produced `<result-bundle>.xcresult`, run this command to extract the detailed failure messages into a JSON file.

```bash
xcrun xcresulttool get test-results tests \
  --path <result-bundle>.xcresult \
  --format json > <results-json-path>.json
```

Evidence contract: when tests fail, provide:
- `<output-path>.txt` (full output)
- `<result-bundle>.xcresult` path
- `<results-json-path>.json` (required)

