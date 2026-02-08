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

## Notes

- For file formats that require comment prefixes, the header should still be present as plain text at the top of the file unless that breaks the format. If it breaks the format, adapt by prefixing each line with the file’s comment marker while preserving the same keys.

## Symlinks

Symlinks cannot “contain” a managed header. For symlinked installs, treat a host path as managed if it is a symlink pointing at the expected `ai-dev-process` target path.
