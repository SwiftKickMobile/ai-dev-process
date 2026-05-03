# 🟡 Product / system name

🟡 Short paragraph describing what this product or system does, its primary users, and what problem it solves. (1-3 sentences.)

INSTRUCTION: The install runbook should gather this content via discussion at install time. If the human defers, leave the 🟡 markers and this INSTRUCTION line in place; a future install/update will surface them again.

## Scopes

This PRD is organized by scope. (See `Submodules/skai/Guides/Core/prd-guide.md` for the full taxonomy.)

- [`platform/`](platform/_platform.md) -- system-wide invariants (protocol-level, format-level, infrastructure-level rules).
- [`domains/`](domains/_domains.md) -- business / domain rules (entities, invariants, state transitions).
- [`features/`](features/_features.md) -- user-facing capabilities reusable across apps.
- [`apps/`](apps/_apps.md) -- per-app behavior.

For hybrid repos: some scopes may be routed to a shared submodule. See `docs/skai/integration.md` (Requirements section) for the routing.

## Glossary

Domain terms used across requirements: [`glossary.md`](glossary.md).
