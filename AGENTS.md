# ArchKit — Agent Guidelines

## What ArchKit Does

ArchKit generates `.archkit/` artifacts that describe a repository's structure: zones, entry points, placement rules, and health. Use these artifacts to navigate and edit more accurately.

## Skills

| Skill | Trigger |
|-------|---------|
| `archkit:init` | `.archkit/` does not exist |
| `archkit:route` | Before choosing files for any task |
| `archkit:audit` | Repo feels noisy; duplicates suspected |
| `archkit:fix` | Misplacements identified by audit |
| `archkit:prune` | Dead files identified by audit |
| `archkit:pack` | After structural changes |
| `archkit:refresh` | Repo changed significantly |

## Platform Tool Names

| Claude Code | Copilot CLI | Codex | Gemini CLI |
|-------------|-------------|-------|-----------|
| `Skill` tool | `skill` tool | `skill` tool | `activate_skill` |

## Core Rules

- Route results are **suggestions** (PRIMARY / SECONDARY / AVOID), not hard limits
- `AVOID` paths are **deprioritized**, not forbidden
- Always follow dependencies outside suggested scope when needed
- `archkit:fix` and `archkit:prune` are **dry-run by default**
- ArchKit **never deletes files** — archive only

## Artifacts Reference

| File | Purpose |
|------|---------|
| `.archkit/zones.json` | Zone map for routing |
| `.archkit/placement-rules.json` | Where new code belongs |
| `.archkit/report.json` | Audit results |
| `.archkit/agent-context.md` | Compact repo briefing (injected by hook) |
| `.archkit/subagent-template.md` | Subagent briefing template |
| `.archkit/archive/` | Archived files (never delete from here) |
