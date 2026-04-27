---
name: archkit-fix
description: Use after archkit:audit has identified misplaced files to safely move them and update all import references; always dry-run first
---

# ArchKit Fix

Repair misplaced files by moving them to the correct zone and updating all import references.

**Core principle:** Dry-run by default. Always show the full plan and get user confirmation before executing. Never move a file you cannot fully update the imports for.

**Authority boundary:** Fix may touch software files only for behavior-preserving structural moves. It must not intentionally change runtime behavior, public APIs, data schemas, dependency choices, configuration semantics, or product decisions. If a move requires semantic changes, skip it and report why.

## Process

### Step 1 — Load candidates

```bash
cat .archkit/report.json
```

Filter to `misplacementCandidates` with confidence ≥ 0.75. Skip anything below that threshold.

Also check for duplicateCandidates — those require manual consolidation, not automated moves.

### Step 2 — Build move plan

For each misplacement candidate:

1. **Verify the candidate** — read the file briefly to confirm the mismatch is real
2. **Determine target path** — use `zones.json` to find the correct zone's primary path
3. **Determine new filename** — keep filename unless it's misleading (e.g., `userService.ts` in components → `src/services/userService.ts`)
4. **Find all import references** — search the codebase for every file that imports this path:

```bash
# Find all files that import the candidate (adjust extension as needed):
grep -r "from.*components/userService" . \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" -l

grep -r "require.*components/userService" . \
  --include="*.ts" --include="*.js" -l
```

5. **Check for path alias complexity** — read `tsconfig.json` or `jsconfig.json` for path aliases (e.g., `@/`, `~/`). Note any that affect the move.
6. **Check behavior preservation** — skip the move if dynamic imports, generated references, framework conventions, public module paths, package exports, tests, docs, or config make the move unsafe or ambiguous.

### Step 3 — Output dry-run plan

Present the full plan before doing anything:

```
ArchKit Fix — Dry Run
──────────────────────
Planned moves:

  1. src/components/userService.ts → src/services/userService.ts
     Import updates required in:
       - src/pages/UserProfile.tsx (line 3)
       - src/routes/user.route.ts (line 1)
       - tests/user.test.ts (line 2)
     Confidence: 0.88
     Behavior-preserving check: all references resolved; no public export/config/test blockers found

  2. src/utils/authHandler.ts → src/routes/authHandler.ts
     Import updates required in:
       - src/index.ts (line 5)
     Confidence: 0.76
     Behavior-preserving check: all references resolved; no public export/config/test blockers found

Files that will NOT be moved (confidence < 0.75):
  - src/lib/config.ts (confidence: 0.61 — too ambiguous)

Files that will NOT be moved (behavior preservation unclear):
  - src/api/publicClient.ts — exported package path may be public API

To execute: confirm with "yes, run archkit fix" or invoke /archkit-fix --execute
```

**Do NOT proceed past this point without explicit user confirmation.**

### Step 4 — Execute (only after confirmation)

For each planned move:

1. Copy the file to the new path
2. Update all import references found in Step 2
3. Verify the file at the new path exists and looks correct
4. Run the relevant available verification command for the affected project area when one exists
5. Delete the original file only after the moved file, import updates, and verification are acceptable

Import update pattern (TypeScript/JavaScript):
```bash
# Replace old path with new path in each affected file
# Use exact string replacement — do not regex-replace carelessly
sed -i '' "s|from '../../components/userService'|from '../../services/userService'|g" src/pages/UserProfile.tsx
```

For each file update, read it after editing to verify the replacement was correct.

### Step 5 — Update map.json

After all moves complete, update `.archkit/map.json` to reflect the new file locations and zone assignments.

### Step 6 — Output completion report

```
ArchKit Fix — Complete
──────────────────────
Moved: 2 files
Import references updated: 5 files
Skipped (low confidence): 1 file
Skipped (behavior preservation unclear): 1 file

Run /archkit-pack to regenerate agent-context.md with the updated structure.
```

## Safety Rules

- **Never move** a file if you cannot find all its import references
- **Never execute** without explicit user confirmation of the dry-run plan
- **Never move** a file when behavior preservation is unclear
- **Never change** runtime semantics, public APIs, schemas, dependency versions, or config meaning to make a move work
- **Never use** broad regex patterns for import updates — use exact path strings
- **If a move fails mid-way**, report the partial state immediately and stop
- **Duplicate candidates** are excluded — consolidation requires human judgment

## Red Flags

- Executing without showing dry-run first
- Moving a file with dynamic imports (`require(variable)`) — skip it
- Updating imports in generated files (`dist/`, `build/`) — skip them
- Presenting this as reversible without noting the original path
- Treating cleaner structure as a reason to alter behavior or public contracts
