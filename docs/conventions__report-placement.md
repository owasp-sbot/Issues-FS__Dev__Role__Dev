# Convention: Dev Role Report Placement

**Document identifier:** conventions__report-placement
**Version:** v0.1.0
**Date:** 2026-02-10
**Status:** Active

---

## Report Location

All Dev role research reports, experiment logs, and investigation outputs go into:

```
roles/Issues-FS__Dev__Role__Dev/docs/reports/
```

## Naming Convention

```
v{version}__{type}__{topic}.md
```

Where:
- **version** matches the Dev role version at time of creation (e.g., `v0.1.4`)
- **type** is one of: `report`, `experiment`, `investigation`, `poc`
- **topic** is a kebab-case description (e.g., `osbot-playwright-research`)

### Examples

```
v0.1.4__report__osbot-playwright-research.md
v0.1.5__experiment__memory-fs-sqlite-backend.md
v0.1.6__investigation__type-safe-migration-gaps.md
v0.1.7__poc__graph-based-test-runner.md
```

## Report Structure

Reports should include at minimum:

1. **Header** with date, researcher role(s), and status
2. **What** was investigated and why
3. **What was tried** (exact commands, code, outputs)
4. **What worked** and **what didn't** (be honest about failures)
5. **Issues found** (code-level or environment-level)
6. **Recommendations** for the ecosystem

## Rationale

Reports are versioned alongside the Dev role to maintain traceability. They live in the Dev role repo (not the parent orchestration repo) because:
- They are Dev role artifacts
- They can be found by reading the Dev role repo
- They don't clutter the orchestration repo
