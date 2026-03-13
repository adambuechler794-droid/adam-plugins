---
name: plan-to-vk-issues
description: Use when a Superpowers plan document has been finalized and the user wants to create Vibe Kanban issues from it. Activates after superpowers:writing-plans completes or when user says "create issues", "send to VK", "populate the board", "create tickets", or similar.
version: 0.1.0
---

# plan-to-vk-issues

Translate a finalized Superpowers plan document into well-structured Vibe Kanban issues **without losing granularity or detail**. The plan document is the source of truth. Every task, acceptance criterion, context note, and dependency captured during brainstorming and planning MUST survive the translation.

## Prerequisites

- A finalized plan document exists (typically `docs/plans/YYYY-MM-DD-<feature-name>.md`)
- The Vibe Kanban MCP server is connected and `create_issue` tool is available
- You know the target project (check MCP context or ask the user once)

## Iron Rules

1. **No detail loss.** If the plan says it, the issue captures it. Do not summarize away context, edge cases, or acceptance criteria. The issue description is what the executing agent will see — it has ZERO context from the brainstorm conversation.
2. **No invented scope.** Do not add tasks that aren't in the plan. Do not split or merge tasks unless the user approves.
3. **One plan task = one issue.** Each discrete task in the plan becomes exactly one issue. Do not combine multiple plan tasks into a single issue.
4. **Parent-child structure mirrors plan hierarchy.** The feature-level item is the parent. All implementation tasks are children. If the plan has sub-sections (e.g., "Backend", "Frontend", "Tests"), create intermediate parent issues for each section.
5. **Acceptance criteria are mandatory.** Every issue MUST have acceptance criteria derived from the plan. If the plan task doesn't have explicit criteria, derive them from the task description and the brainstorm context.

## Issue Structure

Each issue created via `create_issue` MUST follow this structure:

### Title
- Action-oriented: starts with a verb ("Implement", "Add", "Create", "Configure", "Write")
- Specific enough that the executing agent knows the scope from the title alone
- Include the component/area prefix if the plan has sections (e.g., "[Backend] Implement JWT rotation")

### Description

```markdown
## Context
[Why this task exists. What problem it solves. Reference to the parent feature.
Include relevant architectural decisions from the brainstorm.]

## Requirements
[Specific requirements from the plan. Bullet points OK here.
Include file paths, API signatures, data structures — anything concrete from the plan.]

## Acceptance Criteria
- [ ] [Criterion 1 — must be verifiable by the executing agent]
- [ ] [Criterion 2]
- [ ] [Criterion N]

## Dependencies
[List any issues that must be completed before this one.
Reference by title since IDs won't exist yet.]

## Notes for Executing Agent
[Any context the executing agent needs that isn't obvious from the requirements.
Patterns to follow, files to reference, traps to avoid.
If Superpowers TDD applies: "Use superpowers:test-driven-development for implementation."]

## Execution Method
This issue should be implemented using subagent-driven development.
Use superpowers:subagent-driven-development for implementation.
Each subtask within this issue should be delegated to a subagent
with the plan context and acceptance criteria provided above.
```

## Process

1. **Read the plan document.** Load the full plan from `docs/plans/`. Do not work from memory of the planning conversation.
2. **Identify the hierarchy.** Map the plan structure to parent/child issue relationships. Announce the proposed structure to the user.
3. **Confirm with user.** Present the issue titles and hierarchy before creating anything. Ask: "Does this breakdown look right? Any tasks to split, merge, or reorder?"
4. **Create parent issue first.** The parent issue title should be the feature name. Description should include the high-level goal and link to the plan document path.
5. **Create child issues in dependency order.** Issues that others depend on come first. Set `parent_issue_id` on each child.
6. **Announce each creation.** After each `create_issue` call, confirm: "Created: [title] → [issue_id]"
7. **Summary.** After all issues are created, present a table:

| Issue | Title | Parent | Dependencies |
|-------|-------|--------|-------------|
| #1    | ...   | —      | —           |
| #2    | ...   | #1     | —           |
| #3    | ...   | #1     | #2          |

8. **Tag the plan document.** Append the issue IDs to the plan document so there's a bidirectional reference.

## Anti-Patterns — Do NOT Do These

- **Vague titles:** "Handle auth stuff" → Use "Implement JWT token rotation with 7-day refresh window"
- **Empty descriptions:** Never create an issue with just a title. The executing agent has no brainstorm context.
- **Lossy summarization:** "Set up the database" when the plan specified schema migrations, seed data, and index creation as separate concerns.
- **Skipping acceptance criteria:** The executing agent cannot verify its own work without them.
- **Flat structure:** If the plan has hierarchy, the issues must have hierarchy.
- **Forgetting dependencies:** If task B requires task A's output, the dependency MUST be noted.
