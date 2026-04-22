---
name: archkit-pack
description: Use after structural changes (fix, prune, major refactor) to regenerate agent-context.md and subagent-template.md so future sessions have accurate repo context
---

# ArchKit Pack

Regenerate the agent-facing artifacts: `agent-context.md` and `subagent-template.md`. These are the files the session hook injects, so keeping them current is what makes ArchKit passive.

**Core principle:** Pack is what converts raw artifacts into agent-optimized context. Run it after any structural change to keep future sessions accurate.

## When to Run

- After `archkit:fix` moves files
- After `archkit:prune` archives dead files
- After a large manual refactor
- After `archkit:refresh` rescans the repo
- Whenever `agent-context.md` feels stale

## Process

### Step 1 — Load current artifacts

```bash
cat .archkit/canon.json
cat .archkit/zones.json
cat .archkit/placement-rules.json
cat .archkit/avoid.json
cat .archkit/map.json
```

### Step 2 — Regenerate agent-context.md

Write `.archkit/agent-context.md` as a compact, scannable briefing. This file must be:
- Under 80 lines
- Machine-readable (structured with headers and tables)
- Free of narrative prose
- Accurate to the current state of `.archkit/`

Template:

```markdown
# ArchKit Repository Context
Generated: <ISO timestamp>
Repo: <name>

## Type
<repoType> — <framework>

## Zone Map
| Zone | Paths | Entry Points | Key Terms |
|------|-------|--------------|-----------|
| api | src/routes/** | src/routes/index.ts | route, endpoint, handler, controller |
| services | src/services/** | src/services/index.ts | service, business logic |
| models | src/models/** | — | model, entity, schema, database |
| tests | tests/**, src/**/*.test.ts | — | test, spec, coverage |
| utils | src/utils/** | — | utility, helper, shared, format |
| config | config/**, src/config/** | config/default.ts | config, setting, environment |

## Placement Rules
- New API route → `src/routes/` — file pattern: `{feature}.route.ts`
- New service → `src/services/` — file pattern: `{feature}.service.ts`
- New model → `src/models/` — file pattern: `{Feature}.model.ts`
- New utility (used in 2+ zones) → `src/utils/`
- Tests → co-located as `{file}.test.ts` or in `tests/`

## Key Entry Points
- `src/index.ts` — application root
- `src/routes/index.ts` — route registry
- `src/app.ts` — Express app setup

## Avoid Paths
- `src/legacy/` — deprecated, deprioritize not forbid
- `.archkit/archive/` — pruned files, ignore
- `dist/` — compiled output, do not edit

## Routing Instruction
Route results are starting points. Always expand scope when:
- A dependency goes outside the suggested zone
- Shared logic appears in another zone
- Multiple candidate files exist
```

### Step 3 — Regenerate subagent-template.md

Write `.archkit/subagent-template.md`:

```markdown
# ArchKit Subagent Briefing Template
Use this when dispatching subagents for implementation tasks.

---
You are working in a {{REPO_TYPE}} repository ({{FRAMEWORK}}).

**Your task:** {{TASK_DESCRIPTION}}

**Architecture zones:**
{{ZONE_TABLE}}

**Start with these files (PRIMARY):**
{{PRIMARY_PATHS}}

**Expand to these if dependencies require (SECONDARY):**
{{SECONDARY_PATHS}}

**Deprioritize (AVOID — not forbidden):**
{{AVOID_PATHS}}

**If creating new code, place it at:**
{{NEW_CODE_PLACEMENT}} — file pattern: {{FILE_PATTERN}}

**Expansion rule:** If imports, dependencies, or shared logic exist outside the paths above, follow them. Do not stay restricted to primary paths if the task requires broader access.

**Safety:** Do not edit files in dist/, build/, or .archkit/archive/.
---
```

### Step 4 — Confirm

```
ArchKit Pack — Complete
────────────────────────
agent-context.md regenerated (<N> lines)
subagent-template.md regenerated

Future sessions will reflect the updated repo structure.
```
