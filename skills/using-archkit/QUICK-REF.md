# ArchKit — Quick Reference

ArchKit is agent navigation metadata. It tells you where to START — it does not restrict where you go, and it never overrides user instructions, tests, docs, imports, or existing contracts.

## Skills

| Skill | When to invoke |
|-------|----------------|
| `archkit:init` | `.archkit/` doesn't exist — run this first |
| `archkit:route` | Before choosing files for any task |
| `archkit:audit` | Repo feels noisy or structurally drifted |
| `archkit:fix` | After audit — move misplaced files (dry-run first) |
| `archkit:prune` | After audit — archive dead files (dry-run first) |
| `archkit:pack` | After fixes — regenerate agent-context.md |
| `archkit:refresh` | After major repo changes |

## Rules

- If `.archkit/` doesn't exist, invoke `archkit:init` before starting any non-trivial task
- Invoke `archkit:route` before choosing files — results are suggestions, not hard limits
- PRIMARY → SECONDARY → AVOID (AVOID is deprioritized, not forbidden)
- Always expand scope when dependencies require it
- `archkit:fix` and `archkit:prune` are dry-run by default — never execute without user confirmation
- ArchKit never deletes files — archive only

**Claude Code:** `Skill` tool (e.g., `archkit:route`) | **Gemini CLI:** `activate_skill` | **Other:** `skills/<name>/SKILL.md`
