---
name: using-archkit
description: Use at the start of any session in a code repository to understand how ArchKit optimizes agent navigation, routing, and structural decisions
---

<SUBAGENT-STOP>
If you were dispatched as a subagent for a specific implementation task, skip this skill unless your task explicitly involves repo structure or file routing.
</SUBAGENT-STOP>

# ArchKit

ArchKit is a repository optimization layer. It improves how you navigate, route, and edit inside a codebase by maintaining machine-readable architecture artifacts in `.archkit/`.

**Core principle:** ArchKit is a prioritization system, not a restriction system. It tells you where to START, not where you MUST stay.

## Skill Map

| Skill | When to use |
|-------|-------------|
| `archkit:init` | `.archkit/` doesn't exist yet, or repo was substantially restructured |
| `archkit:route` | Before editing files for a task — find the right starting point |
| `archkit:audit` | Repo feels noisy, bloated, or structurally drifted |
| `archkit:fix` | Audit found misplaced files; safe, import-aware repair |
| `archkit:prune` | Audit found dead/duplicate files; archive them safely |
| `archkit:pack` | After major fixes — regenerate agent-context.md |
| `archkit:refresh` | Repo changed significantly; rescan and update all artifacts |

## When to Invoke Skills

**Always invoke `archkit:route`** before starting a new feature, bug fix, or refactor. It surfaces the right files immediately.

**Invoke `archkit:audit`** when:
- You're about to add code and the area feels messy
- You notice duplicate files or functions
- You're unsure which of several similar files to edit

**Invoke `archkit:init`** when:
- `.archkit/` does not exist in the repo root
- The user asks to initialize or set up ArchKit

**Invoke `archkit:fix` or `archkit:prune`** only after `archkit:audit` has run and produced candidates.

## Safety Rules (Always)

- Route results are **suggestions**, not hard limits
- `AVOID` paths are deprioritized, not forbidden
- Always expand scope when dependencies or shared logic require it
- Never archive or move files without confirming with the user first
- `archkit:fix` and `archkit:prune` run in **dry-run mode by default**

## Accessing Skills

**Claude Code:** Use the `Skill` tool — `archkit:init`, `archkit:route`, etc.
**Copilot CLI / Codex:** Use the `skill` tool with the same names.
**Gemini CLI:** Use `activate_skill`.
**Other agents:** Read `skills/<name>/SKILL.md` directly.
