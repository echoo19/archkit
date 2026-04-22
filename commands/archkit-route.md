---
description: "Route a task description to the most relevant files — outputs PRIMARY, SECONDARY, and AVOID paths with confidence"
---

Invoke the `archkit:route` skill to find the right starting files for the current task.

Pass the task description as context. For example: `/archkit-route add password reset endpoint` or just `/archkit-route` to route based on the current conversation context.

Outputs PRIMARY paths (start here), SECONDARY paths (expand if needed), AVOID paths (deprioritized, not forbidden), a new-code placement suggestion, and a confidence level.
