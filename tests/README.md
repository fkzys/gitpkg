# Tests

## Overview

| File | Scope | Root | What it tests |
|------|-------|------|---------------|
| `test.sh` | Unit | No | `common.sh` + `sandbox.sh` — path validation, filelist/symlink checks, protected dirs, URL dedup, commit reading, escape stripping, Makefile target detection, system package detection |
| `test_integration.sh` | Integration | Yes | Full lifecycle: install → update → remove via real `gitpkg` commands with a local git repo, bubblewrap sandbox, and file deployment |

## Running

```bash
# Unit tests (no root needed)
make test

# Integration tests (requires root + bubblewrap)
sudo bash tests/test_integration.sh
```

## How they work

### Unit tests (`test.sh`)

Sources `common.sh` and `sandbox.sh` directly, then exercises pure functions
against temporary files. No network, no root, no real packages installed.

Covers:
- `_validate_path`: absolute paths, traversal (`..`), newlines, empty strings
- `_validate_filelist`: valid/invalid/empty file lists
- `_validate_symlinks`: safe symlinks, traversal symlinks, regular files
- `is_protected_dir`: system dirs protected, non-system dirs allowed
- `_dedup_urls`: deduplication, order preservation, empty line filtering
- `_safe_read_commit`: normal/dirty/missing/empty commit files, truncation
- `_strip_escape_sequences`: ANSI color codes, bold, plain text
- `_has_make_target`: present/absent targets, whitespace before colon
- `_is_system_managed`: real package manager query (system-dependent)

### Integration tests (`test_integration.sh`)

Creates a local bare git repo, clones it, and runs `gitpkg install`, `update`,
`remove` against real files. Tests:

- v1 install simulation + standalone update to v2
- Metadata sync (`depends`, `backup` copied to dbdir)
- Already up-to-date detection
- Dry-run (nothing changes on disk)
- Real update to v3
- Update after source cache deleted (re-clone from recorded URL)
- Clean removal (binary + db entry deleted)

Requires: root, bubblewrap, git, make.
Self-installs gitpkg from source tree if not already installed, cleans up on exit.

## Test environment

- Unit tests create a temporary directory (`mktemp -d`) cleaned up via `trap EXIT`
- No root privileges required for unit tests
- Integration tests install/remove real files under `/usr/bin`, `/var/lib/gitpkg`
- CI runs unit tests on every push; integration tests run with `sudo` on Ubuntu (see `.github/workflows/ci.yml`)
