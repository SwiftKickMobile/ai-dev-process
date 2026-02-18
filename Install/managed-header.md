# Managed File Header (Required)

Any file generated/maintained by `ai-dev-process` in a host project must start with this header block.

## Header format

The first lines of the file must be:

```
Managed-By: ai-dev-process
Managed-Id: <asset-id>
Managed-Source: <repo-relative-source> OR <submodule-path>/<repo-relative-source>
Managed-Adapter: <adapter-id>
Managed-Updated-At: <yyyy-mm-dd>
```

Rules:
- Header must appear at the very top of the file.
- Header keys and casing must match exactly.
- `Managed-Id` must match an entry in `assets.manifest.json`.
- Installer/update overwrites a file only when this header is present (or the destination does not exist).
- `Managed-Updated-At` should only change when the file content actually changes. If the source template is unchanged and the destination already has the managed header, **skip the file** -- do not rewrite it just to bump the date. This avoids unnecessary diffs during submodule updates.

## Determining today's date

Whenever a date is needed (for `Managed-Updated-At`, CHANGELOG entries, `install-state.json`, or any other purpose), **always run `date +%Y-%m-%d` in the terminal** to get the current date. Do not rely on dates from the system prompt or conversation context -- they may be stale or in a different timezone than the human.

## Notes

- For file formats that require comment prefixes, the header should still be present as plain text at the top of the file unless that breaks the format. If it breaks the format, adapt by prefixing each line with the file's comment marker while preserving the same keys.

## Skill files (`.cursor/skills/**/SKILL.md`, `.claude/skills/**/SKILL.md`)

Skill files require YAML frontmatter at the top of the file, so the standard managed header cannot appear as the literal first lines.

For these files, treat a skill file as managed if it contains a managed marker comment **immediately after the YAML frontmatter**, for example:

```markdown
---
name: ai-dev-process-debugging
description: ...
---
<!-- Managed-By: ai-dev-process | Managed-Id: skill.ai-dev-process-debugging | Managed-Source: Submodules/ai-dev-process/Templates/skills/ai-dev-process-debugging/SKILL.md | Managed-Adapter: cursor | Managed-Updated-At: 2026-02-17 -->
```

The shared skill templates at `Templates/skills/*/SKILL.md` do **not** contain the managed marker. Each installer stamps it at copy time with the appropriate `Managed-Adapter` value (`cursor` or `claude-code`).

Rules:
- Installers may overwrite a skill file only when this marker is present (or when the destination does not exist).
- `Managed-Id` must match an entry in `assets.manifest.json`.
- Previously installed skills may have adapter-prefixed IDs (e.g., `cursor-skill.ai-dev-process-debugging`); treat these as managed (the marker is present) and overwrite with the current ID format.

## Symlinks

Symlinks cannot "contain" a managed header. For symlinked installs, treat a host path as managed if it is a symlink pointing at the expected `ai-dev-process` target path.

## Ignore files (`.cursorignore`, `.claudeignore`)

Ignore files are typically gitignore-style and are often project-owned. Installers must not overwrite them wholesale.

Instead, installers may manage a delimited block using comment markers, for example:

```
# BEGIN Managed-By: ai-dev-process
... patterns ...
# END Managed-By: ai-dev-process
```

The installer may create the file if it does not exist, and may update only the block if it exists.
