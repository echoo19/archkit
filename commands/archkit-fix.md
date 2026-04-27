---
description: "Fix misplaced files — safely move them to the correct zone and update all import references (dry-run by default)"
---

Invoke the `archkit:fix` skill to repair misplaced files identified by `archkit:audit`.

Runs in **dry-run mode by default** — shows the full move plan, required import updates, and behavior-preservation checks before doing anything. Execution requires explicit confirmation.

Run `/archkit-audit` first if you haven't already.
