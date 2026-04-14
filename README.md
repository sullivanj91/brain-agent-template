# Brain Agent Template

A template for autonomous, self-evolving Claude Code brain agents. Clone this to create an agent that wakes on a schedule, does useful work, learns over time, and communicates with its owner.

## The Pattern

Each brain agent is a Claude Code cloud scheduled task paired with a GitHub repo as persistent memory. The agent:
- Clones this repo at the start of every session
- Reads its instructions from `prompts/role.md`
- Does work, logs results, evolves its own strategy
- Pushes all changes back to the repo
- Communicates with you via GitHub Issues

## Quick Start

### Option A: Use the `/new-brain-agent` skill (recommended)

If you have this repo (or any brain agent repo) open in Claude Code:
```
/new-brain-agent
```
The skill will ask a few questions and set everything up automatically.

### Option B: Manual setup

1. Click **"Use this template"** on GitHub to create your brain repo
2. Create a cloud environment at claude.ai with your agent's API keys
3. Create a scheduled trigger at claude.ai/code/scheduled with:
   - **Repo:** your new brain repo
   - **Prompt:**
     ```
     cd /home/user && pip install uv -q && cd <your-repo-name> && uv sync
     Your full instructions are in prompts/role.md — read that file and follow it exactly.
     ```
4. Edit `prompts/role.md` to describe what your agent should do
5. Run the trigger manually to seed the initial state

## Repo Structure

```
prompts/
  role.md          ← agent reads + evolves its own instructions here
logs/
  evolution/       ← changelog of what the agent has changed and why
data/
  observations.md  ← ongoing patterns and findings
  learnings.md     ← what worked, what didn't
inbox/
  owner/           ← you write directives here; agent reads them every session
  resolved/        ← move handled items here
scripts/           ← agent-created analysis and execution code
CLAUDE.md          ← context loaded automatically by every Claude Code session
```

## 2-Way Communication

**Owner → Agent (standing directives):** Write `.md` files in `inbox/owner/`. The agent reads all files there at the start of every session. Move them to `inbox/resolved/` when done.

**Owner → Agent (one-time instructions):** Create a GitHub Issue labeled `directive`. The agent reads open directive issues each session, acts on them, comments to acknowledge, and closes them. You can create these from the GitHub UI or via `gh issue create --label directive`.

**Agent → Owner:** The agent creates GitHub Issues labeled `request` when it needs something (an API key, a new data source, a permission). You get notified by email. Close the issue once resolved.

## Skills (auto-loaded when this repo is open in Claude Code)

| Skill | Description |
|---|---|
| `/new-brain-agent` | Scaffold a new brain agent from this template |
| `/brain-manage` | Check agent status, review logs, respond to issues, update prompts |

## Dependency Management

Always use `uv add <package>` — never bare `pip install`. This installs the package and updates `pyproject.toml` so all future agent sessions pick it up via `uv sync`.
