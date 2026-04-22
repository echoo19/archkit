---
name: archkit-route
description: Use before starting any task in a code repository to identify the right files, avoid wrong-file edits, and get a confident starting point with expansion guidance
---

# ArchKit Route

Route a task description to the most relevant files in the repository.

**Core principle:** Output PRIMARY (start here), SECONDARY (expand if needed), AVOID (deprioritize, not forbidden). Always include expansion instructions. Agents must never be prevented from following dependencies outside the suggested scope.

## Process

### Step 1 — Load zone data

Read `.archkit/zones.json`. If it doesn't exist, invoke `archkit:init` first.

```bash
cat .archkit/zones.json
cat .archkit/placement-rules.json
cat .archkit/avoid.json
```

### Step 2 — Extract task keywords

From the task description, extract:
- **Nouns**: feature names, entity names, component names
- **Verbs**: action words (add, fix, update, remove, refactor)
- **Technical terms**: specific patterns (route, service, model, test, hook, middleware, etc.)
- **Domain terms**: business-specific words that map to zones

### Step 3 — Score zones

For each zone, compute a relevance score:
- **+3** for each direct match between task keyword and `zone.relatedTerms`
- **+2** for each match between task keyword and zone name
- **+1** for each partial / synonym match
- **+0** for no match

Rank zones by score descending.

### Step 4 — Select primary and secondary paths

**PRIMARY** = Top-scoring zone's paths + its entryPoints  
**SECONDARY** = Second and third scoring zones' paths  
**AVOID** = Paths from `avoid.json` + any zone with score 0  

### Step 5 — Determine new code placement

From `placement-rules.json`, find the rule whose pattern best matches the task description. If no rule matches confidently, suggest the primary zone's root path.

### Step 6 — Determine confidence

| Confidence | Condition |
|---|---|
| **HIGH** | Top zone score ≥ 6 AND clear task intent |
| **MEDIUM** | Top zone score 3–5 OR ambiguous task |
| **LOW** | Top zone score < 3 OR no zones loaded |

**With LOW confidence:** Expand primary paths to include 2–3 zones; encourage broad exploration.

### Step 7 — Output RouteResult

```
TASK: <task description>

PRIMARY (start here):
  - <path1>
  - <path2>

SECONDARY (expand if needed):
  - <path3>
  - <path4>

AVOID (deprioritized, not forbidden):
  - <path5>

NEW CODE PLACEMENT: <target path> / <filename pattern>

CONFIDENCE: HIGH | MEDIUM | LOW

EXPANSION RULE: If dependencies, imports, or shared logic go outside the paths above,
follow them. These paths are a starting point, not a hard scope.
```

### Step 8 — Cache result

Append to `.archkit/routes.json`:
```json
{
  "task": "<task>",
  "primaryPaths": [...],
  "secondaryPaths": [...],
  "avoidPaths": [...],
  "newCodePlacement": "<path>",
  "confidence": "HIGH",
  "timestamp": "<ISO>"
}
```
Keep only the 20 most recent entries.

## Low-Confidence Behavior

When confidence is LOW:
- Do NOT present narrow paths as authoritative
- Widen primary paths to cover multiple zones
- Add explicit note: "Scope is broad — follow imports to narrow down"
- Encourage running `archkit:audit` if the repo feels structurally unclear

## Red Flags — You Are Routing Wrong

- You are pointing to only one file when the task clearly spans layers
- You are blocking expansion by saying "only look here"
- You are omitting SECONDARY paths entirely
- You are not including an expansion rule
- AVOID paths are described as forbidden

All of these violate the prioritization principle. Fix before outputting.
