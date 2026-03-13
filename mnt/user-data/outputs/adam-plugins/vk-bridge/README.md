# vk-bridge

A Claude Code plugin that bridges Superpowers' planning workflow to Vibe Kanban's orchestration layer. Eliminates detail loss when translating plans into issues, and standardizes subagent execution within VK workspaces.

## The Problem

Superpowers produces rich, detailed plans through its brainstorm → write-plan workflow. Vibe Kanban orchestrates parallel agent execution across isolated workspaces. But the handoff between them — turning a plan into board issues — is ad hoc. Details get lost, acceptance criteria get flattened, and the executing agent in each workspace doesn't know how to structure its work.

## What's Inside

### Skills (auto-discovered by Claude Code)

| Skill | Activates When | What It Does |
|-------|---------------|-------------|
| `plan-to-vk-issues` | After `superpowers:writing-plans` completes, or user says "create issues" / "send to VK" | Reads finalized plan doc, creates VK issues via MCP with full context, hierarchy, acceptance criteria, and dependencies preserved |
| `vk-subagent-execution` | Workspace starts on a plan-to-vk-issues issue, or issue description contains "subagent-driven development" | Orchestrates implementation by delegating subtasks to subagents with TDD enforcement, reviewing output, integrating results |

### Commands

| Command | Description |
|---------|-------------|
| `/vk-bridge:create-issues` | Explicitly invoke plan-to-issues translation |
| `/vk-bridge:execute-issue` | Explicitly invoke subagent execution on current issue |

## Workflow

```
User describes feature
        ↓
superpowers:brainstorming (Socratic refinement)
        ↓
superpowers:writing-plans (structured plan document)
        ↓
★ /vk-bridge:create-issues (creates VK board issues from plan)
        ↓
User reviews board, starts workspaces
        ↓
★ /vk-bridge:execute-issue (orchestrated subagent implementation)
        ↓
superpowers:requesting-code-review (quality gate)
        ↓
Merge / PR
```

## Installation

### Local install (for testing and iteration)

```bash
claude --plugin-dir /path/to/vk-bridge
```

Or add to your Claude Code settings:

```json
{
  "plugins": [
    { "type": "local", "path": "/path/to/vk-bridge" }
  ]
}
```

### From a marketplace (once published)

```
/plugin marketplace add your-username/your-marketplace
/plugin install vk-bridge@your-marketplace
```

## Requirements

- **Superpowers plugin** installed in Claude Code
- **Vibe Kanban MCP server** connected (`create_issue` tool available)
- A VK project with at least one repository configured

## Customization

### Project-specific conventions

Create a `conventions.md` alongside your project's CLAUDE.md:

```markdown
# Project Conventions
- All API routes in src/routes/
- Tailwind only, no custom CSS
- Tests co-located with source as *.test.ts
```

The plan-to-vk-issues skill will include these in issue descriptions when the conventions file is referenced in the plan.

## Directory Structure

```
vk-bridge/
├── .claude-plugin/
│   └── plugin.json          ← Plugin manifest (only this goes here)
├── skills/                   ← Auto-discovered by Claude Code
│   ├── plan-to-vk-issues/
│   │   └── SKILL.md
│   └── vk-subagent-execution/
│       └── SKILL.md
├── commands/                 ← Slash commands
│   ├── create-issues.md
│   └── execute-issue.md
└── README.md
```

## License

MIT
