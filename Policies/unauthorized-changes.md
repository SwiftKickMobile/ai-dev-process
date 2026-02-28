Managed-By: ai-dev-process
Managed-Id: policy.unauthorized-changes
Managed-Source: Policies/unauthorized-changes.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-28

# Do NOT Make Unauthorized Changes

Do not make code changes during discussion or troubleshooting unless explicitly authorized by the user. This rule applies especially when:

- Diagnosing issues or problems
- Responding to questions
- Explaining potential solutions
- Adding debug/logging statements

When identifying a solution, explain it and wait for authorization before implementing.

Exception: when the user explicitly requests the change (e.g., "make this change", "fix this", "implement X").

## Explicit authorization (bright line)

"Explicit authorization" means the human clearly instructs the agent to make code changes now (an imperative request to implement/modify), not merely to analyze, explain, or suggest.

If the user intent could plausibly be interpreted as discussion/troubleshooting, treat it as NOT authorized and STOP to ask: "Do you want me to implement this change?"

If operating inside an `ai-dev-process` workflow with checkpoints: treat authorization to proceed as valid only at a checkpoint, with the standard `⏳ GATE:` line (see `Guides/Core/process-flow.md`). Do not infer execution approval from collaborative phrasing outside a gate.

### Authorized examples (code changes allowed)

- "Fix this."
- "Implement X."
- "Make the change."
- "Go ahead and apply that fix."
- "Update the code to do Y."
- "Proceed with the changes."

### Not authorized examples (discussion/troubleshooting; no code changes)

- "Why is this happening?"
- "What do you think is wrong?"
- "How would you fix this?"
- "What are my options?"
- "Can you take a look?"
- "What does this code do?"

### Ambiguous examples (MUST STOP and ask)

- "Can you try X?"
- "Maybe change Y?"
- "Should we do Z?"
- "Let's do X next." (without a clear "implement" instruction)
- "We should do X."
- "We'll knock this out."

### Special case: debugging/logging and other "small" edits

Adding/removing debug logs, changing test assertions, and "quick refactors" are still code changes and require explicit authorization when in discussion/troubleshooting mode.

## Literal instruction compliance (execution-level invariant)

If the human gives an explicit instruction about how to perform a change, follow it literally.

If you believe the instruction is suboptimal, STOP and ask before acting -- do not substitute your preferred approach.

