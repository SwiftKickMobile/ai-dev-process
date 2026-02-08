# Maintain `ai-dev-process` (LLM maintainer runbook)

This file is instructions for an LLM making changes **to the `ai-dev-process` repo itself** (not installing into a host project).

## Safety defaults

- Do not commit unless explicitly asked.
- Do not delete files unless explicitly asked.
- Do not add dependencies unless explicitly asked.
- Keep edits minimal and reversible.

## Before you change anything

1. Read `README.md` to understand the intended system.
2. Identify what kind of change this is:
   - guide/policy content change
   - installer/runbook change
   - template change
   - repo metadata change (manifest/changelog/readme)
3. Propose a short plan and then proceed.

## Required bookkeeping (most common misses)

When you change assets, keep these in sync:
- `assets.manifest.json`
  - add/move/rename assets here
  - keep `id` stable when possible
- `README.md`
  - update the file-level Asset inventory for developer-facing guides
- `CHANGELOG.md`
  - add an entry under “Unreleased” for user-visible changes

## Content rules (project goals)

- Prefer IDE-neutral `.md` sources in this repo.
- IDE-specific outputs (e.g., Cursor `.mdc`) are generated into host repos by installer runbooks under `Install/`.
- Keep examples project-agnostic (use the shared fictional theme, currently “LumenNotes”).

## How to guide an LLM (prompt template)

Use this as a starting prompt when maintaining this repo:

> You are updating the `ai-dev-process` repo.
> Follow `README.md` and `maintain-ai-dev-process.md`.
> Make minimal changes.
> Update `assets.manifest.json`, `README.md` (asset inventory), and `CHANGELOG.md` as needed.
> Do not commit or delete files unless I explicitly ask.
> After edits, summarize exactly what changed and list touched files.

