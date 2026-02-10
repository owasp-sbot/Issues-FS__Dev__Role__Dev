# Debrief: Double-Path Bug Fix (P0)

**Date:** 2026-02-10
**Role:** Dev
**Severity:** P0 (Critical)
**Status:** Resolved

## Summary

Fixed the critical double-path prefix bug in `Path__Handler__Graph_Node` that caused all CLI commands to fail on real repositories. The bug made the CLI unable to see any existing issues, node types, link types, or config files in `.issues/` directories.

## Root Cause

`Path__Handler__Graph_Node` had a default `base_path = '.issues'` and all path methods (14 total) prepended this to their return values when `has_base_path()` was true. However, `Storage_FS__Local_Disk` was already initialized with `root_path` pointing to the `.issues/` directory. This caused all file lookups to resolve to `.issues/.issues/...` instead of `.issues/...`.

**Example:** Looking for `config/node-types.json`
- Storage root: `/repo/.issues/`
- Path handler returned: `.issues/config/node-types.json`
- Effective lookup: `/repo/.issues/.issues/config/node-types.json` (does not exist)
- Correct lookup: `/repo/.issues/config/node-types.json`

## Why Tests Passed Despite the Bug

Tests used round-trip operations (write then read) through the same doubled path. Both writes and reads went through the path handler, so data written to `.issues/.issues/...` was also read from `.issues/.issues/...`. The bug cancelled itself out in tests but broke all operations on real repos with pre-existing data.

## The Fix

Removed the `if self.has_base_path()` branch from all 14 path methods in `Path__Handler__Graph_Node`. Each method now returns a relative path (e.g., `data/bug/Bug-1/issue.json`, `config/node-types.json`) and the storage backend handles the root path.

### Methods Fixed (14 total)
1. `path_for_issue_json` (fixed by Dinis, commit 1058c245)
2. `path_for_node_json`
3. `path_for_node_folder`
4. `path_for_root_issue`
5. `path_for_issues_folder`
6. `path_for_attachment`
7. `path_for_attachments_folder`
8. `path_for_type_index`
9. `path_for_global_index`
10. `path_for_type_folder`
11. `path_for_node_types`
12. `path_for_link_types`
13. `path_for_settings`
14. `path_for_config_folder`

### Tests Updated
- **test_Bug__Double_Path_Prefix.py**: 20 bug tests converted to regression tests verifying the fix
- **test_Path__Handler__Graph_Node.py**: 18 assertions updated to expect relative paths
- **test_Path__Handler__Graph_Node__Phase_1.py**: 11 assertions updated
- **test_Root__Issue__Service.py**: 2 path assertions updated
- **test_Root__Selection__Service.py**: 1 test updated
- **test_cli__init.py** (CLI project): 2 tests updated (`.issues/.issues/config/` -> `.issues/config/`)

### Test Results
- Issues-FS core: **475 passed**
- Issues-FS CLI: **92 passed**

## Verification on Real Repos

Tested `issues-fs list` on all ecosystem repos with real issues:

| Repo | Issues on Disk | CLI `list` Result |
|---|---|---|
| Librarian | 4 | 4 found, all commands work |
| Issues-FS | 24 | 24 found, all commands work |
| Issues-FS__CLI | 2 | Fails (separate `Obj_Id` validation bug) |
| Issues-FS__Service__UI | 44 | Fails (separate `Obj_Id` validation bug) |

## Follow-Up Bug Discovered

The CLI and UI repos have a separate data validation issue: `Schema__Node.created_by` uses `Obj_Id` (hex-only validator) but some issues have human-readable strings like `conductor-agent` or `system`. This causes a `ValueError` when deserializing. This is tracked as a new bug, not related to the double-path fix.

## Commits

- **Issues-FS** (branch `claude/add-bug-tests`): `e4b85ac` - Fix all 13 remaining methods + regression tests
- **Issues-FS__CLI** (branch `claude/fix-double-path-all-methods-rcalB`): `444e436` - Fix CLI test assertions
- **Issues-FS__Dev** (branch `claude/check-gh-submodule-token-rcalB`): `2153548` - Update submodule refs

## Timeline

1. **2026-02-09**: CLI assessment identified the P0 bug during brief execution
2. **2026-02-09**: Dev wrote 20 bug tests documenting the broken behavior from 8 angles
3. **2026-02-09**: Dinis identified root cause and fixed `path_for_issue_json`
4. **2026-02-10**: Dev applied the same fix to all 13 remaining methods
5. **2026-02-10**: All tests converted to regression tests, full suite green
6. **2026-02-10**: Verified on 4 real repos - 2 fully working, 2 blocked by unrelated bug
