# ArchKit — Gemini CLI Guidelines

## What ArchKit Does

ArchKit generates `.archkit/` artifacts describing a repository's zone structure, placement rules, and health. Use them to navigate and edit more accurately.

## Using Skills in Gemini CLI

Use `activate_skill` to load ArchKit skills:

```
activate_skill archkit:init       # Initialize .archkit/ for this repo
activate_skill archkit:route      # Route a task to the right files
activate_skill archkit:audit      # Detect dead / duplicate / misplaced files
activate_skill archkit:fix        # Repair misplacements (dry-run by default)
activate_skill archkit:prune      # Archive dead files (dry-run by default)
activate_skill archkit:pack       # Regenerate agent-context.md
activate_skill archkit:refresh    # Rescan repo and update artifacts
```

## Behavior Rules

- Invoke `archkit:route` before choosing files for any non-trivial task
- Route results are starting points — always expand when dependencies require it
- `AVOID` paths are deprioritized, not forbidden
- `archkit:fix` and `archkit:prune` always dry-run first
- ArchKit never deletes files — archive only
- User instructions always take precedence
