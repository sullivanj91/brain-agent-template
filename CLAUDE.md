# {{AGENT_NAME}} Brain

Autonomous agent persistent state. All files are created and evolved by the agent — do not edit unless overriding agent decisions.

## What This Agent Does

{{AGENT_PURPOSE}}

## Repo Structure

- `prompts/role.md` — agent's instructions; agent evolves this over time
- `logs/<username>/evolution/` — per-user changelog of strategic decisions and self-modifications
- `data/<username>/` — per-user observations and learnings across sessions
- `inbox/<username>/` — write directives here; agent reads every session (scoped to `$BRAIN_USER`)
- `inbox/resolved/` — move handled inbox items here
- `scripts/` — agent-created analysis and execution code

## Multi-User Support

State is isolated per user via the `BRAIN_USER` environment variable set in each CCR trigger.
- One trigger per user, each with `BRAIN_USER=<username>` in the environment
- All state paths are automatically prefixed by `$BRAIN_USER`
- To add a user: create their directories (`inbox/<username>/`, `data/<username>/`, `logs/<username>/evolution/`), commit, and create a new CCR trigger with `BRAIN_USER=<username>`

## Scheduled Triggers

| Trigger | Model | Schedule |
|---------|-------|----------|
| {{TRIGGER_NAME}} | {{MODEL}} | {{SCHEDULE}} |

Manage triggers at claude.ai/code/scheduled.

## Dependency Management

Always use `uv add <package>` — this installs and updates `pyproject.toml` automatically. Never use bare `pip install`.

## Git Workflow

For this repo (brain state, logs, data, scripts): always commit and push directly to `main`.
For any other repos the agent works in: use feature branches and open PRs for human review.

## 2-Way Communication

- **Owner → agent:** Write `.md` files in `inbox/<username>/` (matches `$BRAIN_USER`)
- **Agent → owner:** GitHub Issues labeled `request`
