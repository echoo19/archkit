---
name: archkit-audit
description: Use when the repo feels noisy or structurally drifted, before adding new code to a messy area, or when the user asks to find dead, duplicate, or misplaced files
---

# ArchKit Audit

Detect dead files, duplicates, and misplacements in the repository. Output structured candidates for fix and prune.

**Core principle:** All candidates are probabilistic. Never act on them automatically. Always present them for user review. Risk level governs urgency, not permission.

**Authority boundary:** Audit is structural evidence gathering, not a product decision. Findings must not be used to change runtime behavior, public APIs, schemas, dependency choices, configuration semantics, or existing contracts. If imports, tests, docs, or user instructions disagree with an audit candidate, the candidate loses.

## Process

### Step 1 — Load artifacts

```bash
cat .archkit/zones.json
cat .archkit/map.json
```

If `.archkit/` doesn't exist, run `archkit:init` first.

### Step 2 — Dead file detection

A file is a **dead candidate** if it meets multiple of these criteria:

| Signal | Weight |
|--------|--------|
| Zero inbound imports (nothing imports this file) | High |
| Last modified > 90 days ago | Medium |
| Filename contains `old`, `backup`, `copy`, `tmp`, `unused`, `deprecated` | High |
| Extension is `.bak`, `.orig`, `.old` | High |
| File is in a `legacy/`, `old/`, `deprecated/`, `archive/` directory | Medium |
| Test file with no corresponding source file | Medium |
| Config file that references a removed feature | Low |

**Scan for inbound imports:**
```bash
# For each candidate file, check if anything imports it:
grep -r "from.*<filename-without-ext>" src/ --include="*.ts" --include="*.js" -l 2>/dev/null
grep -r "require.*<filename-without-ext>" src/ --include="*.ts" --include="*.js" -l 2>/dev/null
```

**Risk levels:**
- **LOW risk** = safe to archive (multiple high signals, no imports found)
- **MEDIUM risk** = review before archiving (some signals, uncertain)
- **HIGH risk** = do NOT recommend archiving (conflicting signals, dynamic imports possible)

Only recommend `action: ARCHIVE` for LOW risk, HIGH confidence candidates whose removal from active locations is expected to preserve behavior.

### Step 3 — Duplicate detection

A file is a **duplicate candidate** if:
- Two files share nearly identical names in different directories (e.g., `utils/format.ts` and `helpers/format.ts`)
- Two files export the same primary identifier (function or class name)
- Two files contain > 70% similar logic (estimate from reading both)

```bash
# Find same-named files in different directories:
find . -type f -name "*.ts" -not -path "*/node_modules/*" \
  | awk -F/ '{print $NF}' | sort | uniq -d
```

For each duplicate name found, identify all paths containing it.

### Step 4 — Misplacement detection

A file is a **misplaced candidate** if its content doesn't match its zone:

| Symptom | Example | Suggested zone |
|---------|---------|----------------|
| Business logic in a component file | `components/userService.ts` exports a class with DB calls | `services` |
| Route handler outside routes zone | `utils/authHandler.ts` registers an Express route | `api` |
| Config values in source code | `src/api/config.ts` contains hardcoded URLs | `config` |
| Test file outside test zone | `src/features/user.test.helpers.ts` | `tests` |
| Utility function in feature folder | `src/checkout/dateUtils.ts` used across 5+ features | `utils` |
| Type definitions mixed with logic | `src/routes/types.ts` only contains interfaces | `types` |

For each candidate, read the file briefly to verify the mismatch is real.

### Step 5 — Write report.json

```json
{
  "auditedAt": "<ISO timestamp>",
  "deadCandidates": [
    {
      "path": "src/utils/formatDate.old.ts",
      "reason": "no inbound imports; filename contains .old; last modified 200 days ago",
      "risk": "LOW",
      "action": "ARCHIVE",
      "confidence": 0.91
    }
  ],
  "duplicateCandidates": [
    {
      "paths": ["src/utils/format.ts", "src/helpers/format.ts"],
      "reason": "identical filenames; both export formatDate and formatCurrency",
      "recommendation": "consolidate into src/utils/format.ts; update imports",
      "confidence": 0.85
    }
  ],
  "misplacementCandidates": [
    {
      "path": "src/components/userService.ts",
      "currentZone": "components",
      "suggestedZone": "services",
      "reason": "exports UserService class with database calls; no JSX or UI logic",
      "confidence": 0.88
    }
  ]
}
```

### Step 6 — Output summary

```
ArchKit Audit Complete
──────────────────────
Dead candidates:        <N> (LOW risk: X, MEDIUM: Y, HIGH: Z)
Duplicate candidates:   <N>
Misplacement candidates: <N>

Top issues:
  [DEAD]       src/utils/formatDate.old.ts — no imports, .old extension (confidence: 0.91)
  [DUPLICATE]  src/utils/format.ts ↔ src/helpers/format.ts — same exports (0.85)
  [MISPLACED]  src/components/userService.ts → should be src/services/ (0.88)

Next steps:
  - Run /archkit-fix to repair misplacements (dry-run by default)
  - Run /archkit-prune to archive dead files (dry-run by default)
  - Review duplicates manually before consolidating
  - Do not change runtime behavior to satisfy audit findings
```

## Red Flags — You Are Auditing Wrong

- Recommending ARCHIVE for HIGH-risk candidates
- Flagging files without checking for inbound imports
- Misplacement detection based on filename alone (read the file)
- Presenting candidates as confirmed issues rather than candidates
- Recommending deleting anything (ArchKit never deletes)
- Treating structural cleanliness as more important than functionality, tests, docs, or user instructions
