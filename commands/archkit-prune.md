---
description: "Archive dead and unused files to .archkit/archive/ — never deletes, always reversible (dry-run by default)"
---

Invoke the `archkit:prune` skill to archive dead files identified by `archkit:audit`.

Runs in **dry-run mode by default** — shows the full archive plan and behavior-preservation checks before doing anything. Execution requires explicit confirmation. ArchKit never deletes files — archived files live in `.archkit/archive/` and can be restored at any time.

Run `/archkit-audit` first if you haven't already.
