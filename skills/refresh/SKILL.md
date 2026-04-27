---
name: archkit-refresh
description: Use when the repo has changed significantly (new directories, major refactor, dependency changes) to rescan and update all .archkit/ artifacts
---

# ArchKit Refresh

Rescan the repository and update all `.archkit/` artifacts to reflect the current state.

**Core principle:** Refresh is a lightweight re-init that preserves existing configuration while updating stale data. It does not overwrite manually edited canon.json zone definitions unless the directory structure has materially changed.

**Authority boundary:** Refresh updates agent navigation metadata only. It must not change product behavior, public APIs, schemas, dependency choices, configuration semantics, or code files. Regenerated context must preserve the ArchKit authority boundary.

## When to Run

- After adding or removing major directories
- After a large refactor that moved many files
- After updating dependencies (new frameworks, removed packages)
- When `archkit:route` results feel wrong or stale
- Before a big feature sprint where accuracy matters

## Process

### Step 1 — Load existing artifacts

```bash
cat .archkit/canon.json
cat .archkit/zones.json
```

Note which zones currently exist and their confidence scores.

### Step 2 — Rescan file structure

```bash
find . -maxdepth 4 -type f \
  -not -path "*/.git/*" \
  -not -path "*/node_modules/*" \
  -not -path "*/dist/*" \
  -not -path "*/build/*" \
  -not -path "*/__pycache__/*" \
  -not -path "*/.venv/*" \
  -not -path "*/vendor/*" \
  -not -path "*/.archkit/*" \
  | sort
```

### Step 3 — Diff against map.json

Compare the current file list to `map.json`:
- **New files**: add to map, assign to zones
- **Deleted files**: remove from map (check if they were moved vs genuinely removed)
- **New directories**: check if they represent new zones

### Step 4 — Update zones

For each existing zone: verify its paths still exist. Update `confidence` if evidence has changed.

For new directories found: apply the zone inference from `archkit:init` Step 4. Add new zones if found.

For removed directories: mark the zone as inactive (don't delete it — history matters).

### Step 5 — Update artifacts

Rewrite these files with updated data:
- `map.json` — updated file inventory
- `zones.json` — updated zone list
- `canon.json` — updated `scannedAt` timestamp, updated zones

Do NOT overwrite `report.json` or `routes.json` — those are audit and routing history.

### Step 6 — Regenerate agent-context.md

Run `archkit:pack` to regenerate the session-facing artifacts from the updated data.

### Step 7 — Output summary

```
ArchKit Refresh — Complete
───────────────────────────
Files scanned: <N> (was <N-prev>)
New files: <N>
Removed files: <N>
Zone updates: <list any new/removed/changed zones>
map.json updated
zones.json updated
canon.json updated
agent-context.md regenerated
```
