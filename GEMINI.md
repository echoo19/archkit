# ArchKit — Gemini CLI Guidelines

## What ArchKit Does

ArchKit generates `.archkit/` artifacts describing a repository's zone structure, placement rules, and health. Use them to navigate and edit more accurately.

ArchKit is agent navigation metadata, not product authority. It may guide where to look and where code usually belongs, but it must not override user instructions, repository documentation, tests, public APIs, runtime behavior, schemas, dependency choices, configuration semantics, or maintainer decisions.

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
- ArchKit may touch software files only for behavior-preserving structure/navigation work
- Do not intentionally change functionality to satisfy ArchKit structure
- `archkit:fix` and `archkit:prune` always dry-run first
- ArchKit never deletes files — archive only
- User instructions, tests, docs, and existing contracts always take precedence
