# Codex Instructions

This repo uses `skai` for reusable agentic coding workflows and policies.

Before running a `skai` workflow:

- Read `docs/skai/integration.md` for project-specific build, test, lint, artifact, and evidence-capture details.
- Use the installed skills under `.agents/skills/skai-*/` when the user's request matches one of their descriptions.
- Follow `Submodules/skai/Policies/safe-operations.md` and `Submodules/skai/Policies/universal-stop-conditions.md` for safety boundaries.
- Prefer the canonical guides under `Submodules/skai/Guides/` over older copied guidance elsewhere in the repo.

If project-specific instructions conflict with `skai` defaults, ask the human before proceeding.
