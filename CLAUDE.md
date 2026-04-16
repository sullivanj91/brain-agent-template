# {{AGENT_NAME}} Brain

Autonomous agent persistent state. All files are created and evolved by the agent — do not edit unless overriding agent decisions.

## What This Agent Does

{{AGENT_PURPOSE}}

## Repo Structure

- `prompts/role.md` — agent's instructions; agent evolves this over time
- `logs/evolution/` — changelog of strategic decisions and self-modifications
- `data/` — observations and learnings across sessions
- `inbox/owner/` — write directives here; agent reads every session
- `inbox/resolved/` — move handled inbox items here
- `scripts/` — agent-created analysis and execution code

## Scheduled Triggers

| Trigger | Model | Schedule |
|---------|-------|----------|
| {{TRIGGER_NAME}} | {{MODEL}} | {{SCHEDULE}} |

Manage triggers at claude.ai/code/scheduled.

## Dependency Management

Always use `uv add <package>` — this installs and updates `pyproject.toml` automatically. Never use bare `pip install`.

## Git Workflow

Always commit and push directly to `main`. Never create feature branches.

## 2-Way Communication

- **Owner → agent:** Write `.md` files in `inbox/owner/`
- **Agent → owner:** GitHub Issues labeled `request`
