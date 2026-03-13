---
name: vk-subagent-execution
description: Use when starting work on a Vibe Kanban issue that has subtasks or multiple acceptance criteria. Activates when a workspace session begins on an issue created by plan-to-vk-issues, when the issue description contains "subagent-driven development", or when the user says "execute with subagents".
version: 0.1.0
---

# vk-subagent-execution

Execute Vibe Kanban issues efficiently by breaking the work within a single workspace into subagent-delegated units. The primary agent acts as an **orchestrator** — delegating focused subtasks to subagents, reviewing their output, and integrating the results.

## Prerequisites

- The `Task` tool (subagent spawning) is available
- The issue has a description with requirements and acceptance criteria (created by `plan-to-vk-issues` or equivalent)
- `superpowers:test-driven-development` is understood (subagents MUST follow TDD)

## Iron Rules

1. **Read the issue first.** Before any code, read the full issue description. The description IS your spec. Do not rely on the issue title alone.
2. **Orchestrate, don't implement.** The primary agent plans the subtask breakdown, delegates to subagents, reviews output, and handles integration. The primary agent writes code ONLY for integration glue or when a subtask is trivially small (< 5 lines).
3. **One subtask = one subagent.** Each subagent gets a focused, self-contained unit of work with clear inputs and outputs.
4. **Subagents follow TDD.** Every subagent delegation MUST include the directive: "Use superpowers:test-driven-development. Write the failing test first."
5. **Review before integration.** After each subagent completes, review its output against the acceptance criteria before moving to the next subtask.

## Subtask Delegation Format

When spawning a subagent via the `Task` tool, structure the prompt:

```
## Task
[One-sentence description of what to implement]

## Context
[Relevant context from the parent issue description. Include:
- Which files to modify or create
- Patterns to follow (reference existing code)
- Data structures or API signatures involved]

## Acceptance Criteria
- [ ] [Specific, verifiable criterion from the parent issue]
- [ ] [Tests pass for this specific functionality]

## Constraints
- Use superpowers:test-driven-development. Write the failing test FIRST.
- Do NOT modify files outside the scope listed above.
- Do NOT add dependencies without noting them.
- If you encounter an ambiguity, stop and report it rather than guessing.
```

## Process

1. **Read the issue description completely.** Load all context, requirements, acceptance criteria, and notes.
2. **Decompose into subtasks.** Break the issue into the smallest units that can be independently implemented and tested. Typical decomposition:
   - Data layer / schema changes
   - Core logic / business rules
   - API endpoints or interface changes
   - UI components (if applicable)
   - Integration tests
3. **Announce the plan.** Present the subtask breakdown to the user:
   ```
   Executing [Issue Title] with subagents:
   1. [Subtask 1] — [which files, estimated scope]
   2. [Subtask 2] — [which files, estimated scope]
   3. [Subtask 3] — [which files, estimated scope]
   Integration: [what needs to happen after subtasks complete]
   ```
4. **Execute in dependency order.** Delegate subtasks that have no dependencies first. If two subtasks are independent, they can run in parallel (if the platform supports it).
5. **Review each subtask output.** After a subagent completes:
   - Check that tests were written FIRST (TDD compliance)
   - Check that acceptance criteria are met
   - Check for scope creep (modifications outside stated scope)
   - If issues found: provide specific feedback and re-delegate
6. **Integrate.** After all subtasks pass review:
   - Ensure all tests pass together (not just individually)
   - Handle any integration glue (imports, wiring, configuration)
   - Run the full test suite
7. **Verify acceptance criteria.** Walk through every acceptance criterion from the original issue. Each one must be demonstrably met.
8. **Report completion.** Summarize what was implemented, which tests were added, and any notes for review.

## Handling Failures

- **Subagent produces bad output:** Do NOT fix it yourself. Provide specific feedback and re-delegate. The subagent needs to learn from its mistakes within the TDD cycle.
- **Acceptance criterion can't be met:** Stop. Report to the user with the specific blocker. Do not work around it silently.
- **Scope expansion needed:** If implementation reveals that the issue scope was underestimated, stop and report. Suggest creating a follow-up issue rather than expanding the current one.
- **Three failed attempts on same subtask:** Escalate to user. The issue description may need clarification or the approach may need rethinking. Reference `superpowers:systematic-debugging` if the failure involves a bug rather than a spec problem.

## Anti-Patterns — Do NOT Do These

- **Monolith implementation:** Doing everything in one pass without subagents. The whole point is delegation and review.
- **Vague delegation:** "Implement the auth stuff" → Use "Implement the `validateToken` middleware in `src/middleware/auth.ts` that checks JWT expiry and refreshes tokens within the 7-day window"
- **Skipping review:** Trusting subagent output without checking. Always verify TDD compliance and acceptance criteria.
- **Over-decomposition:** Creating 15 subagents for a task that has 3 acceptance criteria. Match decomposition to actual complexity.
- **Fixing subagent work yourself:** This defeats the purpose. Re-delegate with better instructions instead.
- **Ignoring the issue description:** The issue description was carefully constructed from the brainstorm and plan. It IS the spec. Don't improvise.
