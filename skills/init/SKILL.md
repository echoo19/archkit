---
name: archkit-init
description: Use when .archkit/ does not exist in the repo root, or when the user asks to initialize or reinitialize ArchKit for this repository
---

# ArchKit Init

Scan the repository, infer its architecture, and write `.archkit/` optimization artifacts that all other ArchKit skills depend on.

**Core principle:** You are the agent doing this work using your own tools (Bash, Read, Write). No external binary is required.

## Process

### Step 1 — Explore repo structure

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
  | sort | head -300
```

Also get the top-level directory shape:

```bash
ls -la .
find . -maxdepth 2 -type d \
  -not -path "*/.git*" \
  -not -path "*/node_modules*" \
  -not -path "*/dist*" \
  -not -path "*/build*" \
  -not -path "*/__pycache__*" \
  | sort
```

### Step 2 — Read manifests

Read whichever of these exist: `package.json`, `tsconfig.json`, `pyproject.toml`, `setup.py`, `Cargo.toml`, `go.mod`, `Gemfile`, `pom.xml`, `build.gradle`, `composer.json`.

### Step 3 — Detect repo type

Assign one of: `node-typescript`, `node-javascript`, `python`, `go`, `rust`, `ruby`, `java`, `php`, `monorepo`, `unknown`.

Evidence: manifest files found, language distribution in file extensions.

### Step 4 — Infer zones

Map directories to architecture zones using this table:

| Directory pattern | Zone name | Role |
|---|---|---|
| `src/routes/`, `src/api/`, `routes/`, `app/routes/` | `api` | HTTP handlers / route definitions |
| `src/services/`, `services/`, `app/services/` | `services` | Business logic |
| `src/models/`, `models/`, `src/entities/`, `src/schemas/` | `models` | Data models / ORM entities |
| `src/components/`, `components/`, `app/components/` | `components` | UI components |
| `src/pages/`, `pages/`, `app/pages/`, `views/` | `pages` | Page-level components / views |
| `src/utils/`, `utils/`, `src/helpers/`, `helpers/`, `src/lib/`, `lib/` | `utils` | Shared utilities |
| `src/config/`, `config/`, `src/settings/` | `config` | Configuration |
| `tests/`, `test/`, `__tests__/`, `spec/`, `src/**/*.test.*`, `src/**/*.spec.*` | `tests` | Tests |
| `scripts/`, `tools/`, `bin/` | `scripts` | Build / utility scripts |
| `docs/`, `documentation/` | `docs` | Documentation |
| `migrations/`, `db/migrations/` | `migrations` | Database migrations |
| `middleware/`, `src/middleware/` | `middleware` | Request middleware |
| `hooks/`, `src/hooks/` | `hooks` | Framework hooks |
| `store/`, `src/store/`, `src/state/` | `state` | State management |
| `src/types/`, `types/`, `src/@types/` | `types` | Type definitions |

For each zone found, infer:
- `paths`: glob patterns that belong to it
- `entryPoints`: index files, main entry files
- `relatedTerms`: keywords a task description might contain that should route here
- `confidence`: 0.0–1.0 based on how clearly the directory matches the role

### Step 5 — Identify avoid paths

Paths to deprioritize (not forbid):
- Any directory named `legacy`, `old`, `deprecated`, `archive`, `backup`, `tmp`, `temp`
- `.archkit/archive/`
- `dist/`, `build/`, `out/`, `__pycache__/`, `.venv/`, `vendor/`, `node_modules/`

### Step 6 — Derive placement rules

For each zone, create a rule: "If a task description contains any of these terms, new code should be placed in this zone's primary path."

### Step 7 — Write .archkit/ artifacts

Create the directory and write all files:

```bash
mkdir -p .archkit
```

**canon.json** — architecture contract:
```json
{
  "version": "1.0.0",
  "repoType": "<detected>",
  "scannedAt": "<ISO timestamp>",
  "sourceRoot": "<src/ or . or lib/>",
  "entryFile": "<main entry if found>",
  "conventions": {
    "language": "<primary language>",
    "framework": "<if detectable>",
    "testPattern": "<*.test.ts or test_*.py etc>"
  },
  "zones": [ ...ArchitectureZone objects... ],
  "placementRules": [ ...PlacementRule objects... ]
}
```

**zones.json** — compact zone list for fast routing:
```json
{
  "zones": [
    {
      "name": "api",
      "role": "HTTP route handlers",
      "paths": ["src/routes/**"],
      "entryPoints": ["src/routes/index.ts"],
      "relatedTerms": ["route", "endpoint", "handler", "controller", "REST", "API"],
      "avoidPaths": [],
      "confidence": 0.95
    }
  ]
}
```

**map.json** — file inventory:
```json
{
  "scannedAt": "<ISO timestamp>",
  "fileCount": 0,
  "files": [
    {
      "path": "src/routes/auth.ts",
      "zone": "api",
      "size": 1240,
      "entryPoint": false
    }
  ]
}
```

**avoid.json** — deprioritized paths:
```json
{
  "avoidPaths": [
    { "path": "src/legacy/", "reason": "deprecated code", "forbidden": false },
    { "path": ".archkit/archive/", "reason": "pruned files", "forbidden": false }
  ]
}
```

**placement-rules.json**:
```json
{
  "rules": [
    {
      "pattern": "new API route / endpoint / handler",
      "targetPath": "src/routes/",
      "filePattern": "{feature}.route.ts",
      "confidence": "HIGH"
    }
  ]
}
```

**report.json** — empty until audit runs:
```json
{
  "auditedAt": null,
  "deadCandidates": [],
  "duplicateCandidates": [],
  "misplacementCandidates": []
}
```

**routes.json** — empty until route queries run:
```json
{ "recentRoutes": [] }
```

### Step 8 — Generate agent-context.md

This is the most important artifact. It gets injected into every future session by the hook.

Write `.archkit/agent-context.md` in this format:

```markdown
# ArchKit Repository Context
Generated: <ISO timestamp>
Repo: <name from package.json / directory name>

## Type
<repoType> — <framework if known>

## Zone Map
| Zone | Paths | Entry Points | Key Terms |
|------|-------|--------------|-----------|
| api | src/routes/** | src/routes/index.ts | route, endpoint, handler |
| services | src/services/** | src/services/index.ts | service, business logic |
...

## Placement Rules
- New API route → `src/routes/` as `{feature}.route.ts`
- New service → `src/services/` as `{feature}.service.ts`
...

## Key Entry Points
- `<main entry file>` — application root
- `<route index>` — route registry
...

## Avoid Paths
- `src/legacy/` — deprecated, deprioritize
- `.archkit/archive/` — pruned files, ignore

## Routing Notes
Always expand scope when dependencies go outside the suggested zone.
Route results are starting points, not hard limits.
```

### Step 9 — Generate subagent-template.md

Write `.archkit/subagent-template.md`:

```markdown
# Subagent Briefing Template

Use this template when dispatching a subagent for repo work.

---
You are working in a {{REPO_TYPE}} repository.

**Your task:** {{TASK_DESCRIPTION}}

**Start with these files (PRIMARY):**
{{PRIMARY_PATHS}}

**Expand to these if dependencies require (SECONDARY):**
{{SECONDARY_PATHS}}

**Deprioritize (AVOID — not forbidden):**
{{AVOID_PATHS}}

**If creating new code, place it at:**
{{NEW_CODE_PLACEMENT}}

**Expansion rule:** If imports, dependencies, or shared logic go outside the paths above, follow them. Do not stay restricted to primary paths if the task requires broader access.
---
```

## Completion

After writing all files, output a brief summary:

```
ArchKit initialized.
Repo type: <type>
Zones found: <count> (<names>)
Files indexed: <count>
.archkit/ written: canon.json, zones.json, map.json, avoid.json, placement-rules.json, report.json, routes.json, agent-context.md, subagent-template.md

Run /archkit-route "<task>" to get file routing for your next task.
```
