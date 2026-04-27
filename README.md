# ArchKit

**A skill pack for coding agents that maps your repo structure, routes tasks to the right files, and flags dead or misplaced code before it causes problems.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/echoo19/archkit/releases)

---

## Why This Exists

Coding agents navigate your repo by reading file names, directory structures, and import graphs. When a repo is clean and well-organized, agents find the right files on the first try. When it isn't, they:

- Edit the wrong file because two similar ones exist
- Waste context exploring dead directories
- Put new code in the wrong place because the placement signal is ambiguous
- Miss shared logic that lives outside their initial scope
- Duplicate implementations that already exist elsewhere

These aren't hallucination problems. They're navigation problems. The agent is doing its best with an ambiguous map.

**Dead files, duplicates, and misplacements actively degrade agent performance.** ArchKit fixes the map.

---

## How It Works

ArchKit is a plugin for coding agents (Claude Code, Gemini CLI, Cursor, Codex, Copilot CLI). Install it once and it runs passively.

ArchKit is **agent navigation metadata**, not a product authority. It can suggest where agents should look, where new code usually belongs, and which files deserve structural review. It must not override user instructions, repository docs, tests, public APIs, runtime behavior, data schemas, dependency choices, configuration semantics, or other product decisions.

On first use, it scans your repository and generates a `.archkit/` directory containing:

- A **zone map** describing what each part of the repo is for
- **Placement rules** suggesting where new code usually belongs
- An **agent-context file** injected into every session via a hook
- A **subagent briefing template** for dispatching focused subagents

From that point on, every agent session starts with repo navigation context already loaded. The agent gets hints about where the API layer is, where the service layer is, where tests live, where not to look first, and where new code is usually placed.

When you ask it to work on something, it routes to likely starting files immediately. Agents still follow imports, tests, docs, and existing contracts wherever they lead.

ArchKit also provides **audit, fix, and prune** skills that detect and repair structural problems on demand. These skills may touch software files only when the action is behavior-preserving and mechanically safe. Fixes require confirmation, complete reference coverage, and post-change verification. Pruning moves only low-risk, high-confidence dead files to `.archkit/archive/` and never deletes anything.

---

## Installation

### Claude Code

**Option 1 — One command:**
```bash
/plugin install archkit
```

**Option 2 — Manual clone:**
```bash
git clone https://github.com/echoo19/archkit ~/.claude/plugins/archkit
```
Then reload Claude Code.

**Option 3 — Ask Claude:**
```
Download and install https://github.com/echoo19/archkit as a plugin
```

### Gemini CLI

```bash
gemini extensions install https://github.com/echoo19/archkit
```

To update:
```bash
gemini extensions update archkit
```

### Cursor

In Cursor Agent chat:
```
/add-plugin archkit
```

### OpenAI Codex CLI

```bash
/plugins
```
Search for `archkit` and select Install.

### GitHub Copilot CLI

```bash
copilot plugin install archkit
```

### Manual (any agent)

Clone anywhere and point your agent at it:
```bash
git clone https://github.com/echoo19/archkit
```
Then reference the `skills/` and `hooks/` directories in your agent's configuration.

---

## Quick Start

### 1. Initialize ArchKit for your repo

Inside any code repository:

```
/archkit-init
```

Or ask the agent: *"Initialize ArchKit for this repo"*

ArchKit scans your directory structure, detects your repo type and framework, infers architecture zones, and writes `.archkit/` with all artifacts. This runs once and takes 20–60 seconds depending on repo size.

### 2. Route tasks to the right files

Before starting any feature or bug fix:

```
/archkit-route add password reset endpoint
```

Output:
```
TASK: add password reset endpoint

PRIMARY (start here):
  - src/routes/auth.ts
  - src/services/auth.service.ts

SECONDARY (expand if needed):
  - src/models/User.ts
  - src/middleware/validate.ts

AVOID (deprioritized, not forbidden):
  - src/legacy/
  - src/routes/v1/

NEW CODE PLACEMENT: src/routes/auth.ts
  File pattern: auth.route.ts

CONFIDENCE: HIGH

EXPANSION RULE: If dependencies go outside the paths above, follow them.
```

### 3. Audit for structural issues

```
/archkit-audit
```

Output:
```
Dead candidates:         3 (LOW risk: 2, MEDIUM: 1)
Duplicate candidates:    1
Misplacement candidates: 2

Top issues:
  [DEAD]       src/utils/formatDate.old.ts — no imports, .old extension (0.93)
  [DUPLICATE]  src/utils/format.ts <-> src/helpers/format.ts — same exports (0.85)
  [MISPLACED]  src/components/userService.ts -> should be src/services/ (0.88)
```

### 4. Fix misplacements

```
/archkit-fix
```

Dry-run by default. Shows the full move plan (files to move, imports to update) and waits for confirmation before executing.

### 5. Archive dead files

```
/archkit-prune
```

Dry-run by default. Archives to `.archkit/archive/` — never deletes. Always reversible.

### 6. Keep it current

After major changes:
```
/archkit-refresh
```

After fixes or structural changes:
```
/archkit-pack
```

Regenerates `agent-context.md` so the next session reflects the current structure.

---

## Skills

| Skill | Slash Command | What It Does |
|-------|---------------|-------------|
| `archkit:init` | `/archkit-init` | Scan repo and generate `.archkit/` artifacts |
| `archkit:route` | `/archkit-route` | Route a task to the right files |
| `archkit:audit` | `/archkit-audit` | Detect dead, duplicate, and misplaced files |
| `archkit:fix` | `/archkit-fix` | Move misplaced files with import updates |
| `archkit:prune` | `/archkit-prune` | Archive dead files safely |
| `archkit:pack` | — | Regenerate agent-context.md |
| `archkit:refresh` | `/archkit-refresh` | Rescan repo and update all artifacts |
| `archkit:using-archkit` | — | Introduction and skill map |

---

## The `.archkit/` Directory

| File | Purpose |
|------|---------|
| `canon.json` | Architecture guidance: zones, rules, conventions |
| `zones.json` | Compact zone list for fast routing |
| `map.json` | File inventory with zone assignments |
| `placement-rules.json` | Suggested locations for new code per pattern |
| `avoid.json` | Deprioritized paths with reasons |
| `report.json` | Latest audit results |
| `routes.json` | Recent routing result cache |
| `agent-context.md` | Compact repo briefing injected by hook every session |
| `subagent-template.md` | Template for briefing subagents |
| `archive/` | Archived files from prune — never deleted |
| `archive/manifest.json` | Record of every archived file |

Commit `.archkit/` to version control (except `archive/` if you prefer to keep it local).

---

## Design Principles

**Prioritization, not restriction.** ArchKit tells agents where to start, not where they must stay. Every route result includes PRIMARY paths, SECONDARY paths, and an explicit expansion instruction. Agents always follow dependencies wherever they lead.

**Behavior-preserving only.** ArchKit may move or archive software files only to improve agent navigation, structure, or maintainability, and only when the action is highly likely not to change functionality. It must never intentionally change runtime behavior, public contracts, schemas, dependency versions, build outputs, or configuration meaning.

**Evidence over authority.** Audit findings, placement rules, and route results are candidates and hints. If tests, imports, documentation, or user instructions conflict with ArchKit, those sources win.

**Passive by default.** The session hook injects repo context into every conversation automatically. You don't change your workflow.

**Safe by default.** `archkit:fix` and `archkit:prune` always dry-run first. Pruning archives to `.archkit/archive/` and never deletes. Fix only moves files where all import references are resolvable, dynamic references are not a blocker, and verification can confirm behavior-preserving edits.

**Zero dependencies.** Pure markdown and bash. No npm install, no build step, no external services. Works offline.

---

## Examples

### New feature in a Node/Express repo

```
You: Add a user preferences endpoint

ArchKit (via route skill):
  PRIMARY: src/routes/user.ts, src/services/user.service.ts
  SECONDARY: src/models/User.ts, src/middleware/auth.ts
  PLACEMENT: src/routes/user.ts -> add GET /preferences handler

Agent: [opens src/routes/user.ts, adds handler, follows import to user.service.ts]
```

### Cleaning up a messy repo

```
You: /archkit-audit

ArchKit:
  [DEAD]       tests/auth.test.old.ts — no corresponding source, .old extension
  [MISPLACED]  src/components/apiClient.ts -> should be src/utils/ (no JSX)
  [DUPLICATE]  src/utils/date.ts <-> src/helpers/date.ts — identical exports

You: /archkit-fix
ArchKit: [dry-run] Move src/components/apiClient.ts -> src/utils/apiClient.ts
         Update imports in: src/pages/Dashboard.tsx, src/pages/Settings.tsx
         Confirm?

You: yes
ArchKit: [executes] Moved. 2 import files updated.

You: /archkit-prune
ArchKit: [dry-run] Archive tests/auth.test.old.ts -> .archkit/archive/tests/auth.test.old.ts
         Confirm?

You: yes
ArchKit: [archives] Manifest updated.
```

### Subagent briefing

When dispatching a subagent, ArchKit's `subagent-template.md` gives you a ready-to-use briefing that scopes the subagent to the right zone while always allowing expansion.

---

## Contributing

1. Fork the repository
2. Create a branch for your change
3. Test your skill change against real agent behavior — before and after
4. Submit a PR with the specific agent behavior that motivated the change

Skill changes without evidence of tested agent behavior won't be merged. See `CLAUDE.md` for full guidelines.

---

## License

MIT — see [LICENSE](LICENSE).
