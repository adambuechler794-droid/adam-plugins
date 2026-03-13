---
name: create-issues
description: Create Vibe Kanban issues from a finalized Superpowers plan document. Reads the plan, proposes issue hierarchy, and creates issues via VK MCP server with full context preserved.
---

Use the `plan-to-vk-issues` skill to translate the current finalized plan into Vibe Kanban issues. If no plan path is specified, look for the most recent file in `docs/plans/`. Confirm the issue hierarchy with the user before creating anything.
