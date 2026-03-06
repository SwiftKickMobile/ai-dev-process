Managed-By: skai
Managed-Id: policy.universal-stop-conditions
Managed-Source: Policies/universal-stop-conditions.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-03-06

# Universal STOP Conditions (Agent Policy)

These conditions override `auto` and any other advance intent. When any condition is true, STOP immediately and wait for the human.

## Conditions

1. **Missing/broken required tool or integration.** A tool, service, or integration required by the current workflow is unavailable or returning errors. Do not attempt workarounds -- report what is missing and wait.

2. **Ambiguity requiring product intent.** The next step depends on a product, design, or business decision that is not established in the spec, guide, or integration doc. Do not guess -- state the ambiguity and wait.

3. **About to deviate from an explicit plan/spec/instruction.** The spec/plan says X but you are about to do not-X (skip a step, change an approach, weaken a requirement). Do not proceed -- explain the deviation and wait.

4. **About to run a destructive/irreversible operation.** Any operation that discards work, destroys data, or is difficult to reverse (see `Policies/safe-operations.md` for examples). Do not run it -- describe scope, what could be lost, and wait for explicit approval.

5. **Required evidence/artifacts are unavailable or new evidence contradicts the current hypothesis.** You need output, logs, or test results you cannot access, or new information invalidates your working theory. Do not continue on stale assumptions -- report what changed and wait.
