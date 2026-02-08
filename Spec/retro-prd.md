Managed-By: ai-dev-process
Managed-Id: guide.retro-prd
Managed-Source: Spec/retro-prd.md
Managed-Adapter: repo-source
Managed-Updated-At: 2026-02-08

Prerequisite: read `Spec/work-spec.md`.

Based on the reviewed code and discussion above:

1. Infer externally observable or cross-component behavioral contracts from the existing codebase.
2. Compare them to the existing `/requirements/**` library.
3. Add missing requirements and update incorrect/outdated ones.
4. Do NOT introduce implementation details (types, functions, files, initializers).
5. Place each requirement using the platform/domains/features/apps rules.
6. Do NOT add progress markers.

Only update /requirements/**.