---
name: archkit-prune
description: Use after archkit:audit has identified dead or unused files to safely archive them; always dry-run first and never deletes
---

# ArchKit Prune

Archive dead, unused, and redundant files by moving them to `.archkit/archive/`. Never deletes.

**Core principle:** Archiving is reversible. Deletion is not. ArchKit never deletes files. Everything goes to `.archkit/archive/` with its original path preserved.

**Authority boundary:** Prune may touch software files only when archiving is expected to preserve behavior. It must not remove active code paths, public APIs, schemas, config semantics, dependency choices, or files needed by tests, docs, build tooling, runtime discovery, or framework conventions.

## Process

### Step 1 — Load dead candidates

```bash
cat .archkit/report.json
```

Filter `deadCandidates` to:
- `risk: "LOW"` AND `confidence ≥ 0.85` AND `action: "ARCHIVE"`

**Do NOT archive** `MEDIUM` or `HIGH` risk candidates without explicit user instruction.
**Do NOT archive** any candidate whose behavioral impact is unclear, even if it appears structurally dead.

### Step 2 — Final verification pass

For each candidate, do a final import check before pruning:

```bash
# Confirm zero inbound imports:
grep -r "from.*<basename>" . \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" \
  --include="*.py" -l 2>/dev/null | grep -v ".archkit"

grep -r "require.*<basename>" . \
  --include="*.ts" --include="*.js" -l 2>/dev/null | grep -v ".archkit"
```

If any file imports the candidate, remove it from the archive list and flag it for review.

Also check for non-import references that may affect behavior: package exports, framework file naming conventions, config references, docs that describe public paths, test fixtures, generated manifests, and dynamic loading patterns. If any are found, remove the file from the archive list and flag it for review.

### Step 3 — Output dry-run plan

```
ArchKit Prune — Dry Run
────────────────────────
Candidates to archive:

  src/utils/formatDate.old.ts
    Reason: .old extension; zero inbound imports; last modified 247 days ago
    Confidence: 0.93 | Risk: LOW
    Archive path: .archkit/archive/src/utils/formatDate.old.ts

  src/legacy/v1-auth.ts
    Reason: in legacy/ directory; zero inbound imports
    Confidence: 0.89 | Risk: LOW
    Archive path: .archkit/archive/src/legacy/v1-auth.ts

Excluded (risk too high or confidence too low):
  src/lib/config.ts — MEDIUM risk; possible dynamic reference

Excluded (behavior preservation unclear):
  src/plugins/legacy-auth.ts — possible runtime discovery by filename

To execute: confirm with "yes, run archkit prune" or invoke /archkit-prune --execute
```

**Do NOT proceed without explicit user confirmation.**

### Step 4 — Execute (only after confirmation)

For each approved candidate:

1. Create the archive path:
```bash
mkdir -p ".archkit/archive/$(dirname <original-path>)"
```

2. Move (not copy) the file:
```bash
mv "<original-path>" ".archkit/archive/<original-path>"
```

3. Verify the file exists at the archive path before continuing.

### Step 5 — Write archive manifest

Append to `.archkit/archive/manifest.json` (create if doesn't exist):

```json
{
  "archivedAt": "<ISO timestamp>",
  "entries": [
    {
      "originalPath": "src/utils/formatDate.old.ts",
      "archivePath": ".archkit/archive/src/utils/formatDate.old.ts",
      "reason": ".old extension; zero inbound imports; last modified 247 days ago",
      "confidence": 0.93,
      "archivedAt": "<ISO timestamp>"
    }
  ]
}
```

### Step 6 — Update map.json

Remove archived files from `.archkit/map.json`.

### Step 7 — Output completion report

```
ArchKit Prune — Complete
─────────────────────────
Archived: <N> files
Skipped: <N> files (risk too high)
Skipped: <N> files (behavior preservation unclear)

Archived files are at .archkit/archive/ and can be restored at any time.
Manifest written to .archkit/archive/manifest.json.

Run /archkit-pack to regenerate agent-context.md with the cleaned structure.
```

## Restoring Archived Files

To restore a pruned file:

```bash
mv ".archkit/archive/<original-path>" "<original-path>"
```

## Safety Rules

- **Never delete** — always use `mv`, never `rm`
- **Never archive** MEDIUM or HIGH risk candidates without explicit user instruction
- **Never archive** a file that has any active import (even one)
- **Never archive** files in `src/`, `lib/`, or primary source zones without extra verification
- **Never archive** when runtime discovery, public exports, tests, docs, or config references make behavior preservation unclear
- **Always verify** the archive copy before removing the original

## Red Flags

- Deleting instead of archiving
- Archiving files with active imports
- Archiving without a dry-run
- Using `cp + rm` instead of `mv` (creates window where file is in two places)
- Archiving active software files for structural cleanliness rather than behavior-preserving cleanup
