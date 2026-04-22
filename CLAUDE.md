# ArchKit — Claude Code Guidelines

## What ArchKit Is

ArchKit is a repository optimization plugin. It generates `.archkit/` artifacts that describe a repo's zone structure, placement rules, and health. These artifacts improve how you navigate, route, and edit inside codebases.

## How to Use Skills

Use the `Skill` tool to invoke ArchKit skills. Do not read skill files directly with the `Read` tool.

```
archkit:using-archkit   Introduction and skill map
archkit:init            Initialize .archkit/ for a repo
archkit:route           Route a task to the right files
archkit:audit           Detect dead / duplicate / misplaced files
archkit:fix             Repair misplacements (dry-run by default)
archkit:prune           Archive dead files (dry-run by default)
archkit:pack            Regenerate agent-context.md
archkit:refresh         Rescan repo and update artifacts
```

## Behavior Rules

1. **Invoke `archkit:route` before choosing files** for any non-trivial task if `.archkit/zones.json` exists.
2. **Route results are starting points.** Always expand scope when dependencies require it.
3. **`AVOID` paths are deprioritized, not forbidden.** You may access them if needed.
4. **Never delete files.** `archkit:prune` archives to `.archkit/archive/`.
5. **Dry-run first.** `archkit:fix` and `archkit:prune` always show a plan before executing.
6. **User instructions take precedence** over all ArchKit skill guidance.

## Contributing to ArchKit

See README.md for contribution guidelines. Skill changes require testing against real agent behavior before submission.
