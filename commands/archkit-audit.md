---
description: "Audit this repository for dead files, duplicates, and misplaced code — outputs candidates for fix and prune"
---

Invoke the `archkit:audit` skill to scan for dead files, duplicate implementations, and misplaced code. Outputs structured candidates to `.archkit/report.json`. Findings are review candidates, not product decisions, and must not override tests, docs, imports, user instructions, or existing behavior.

If `.archkit/` doesn't exist yet, run `/archkit-init` first.
