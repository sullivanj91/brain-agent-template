---
name: new-brain-agent
description: |
  Scaffold a new autonomous Brain Agent — a self-evolving Claude Code scheduled task
  with a GitHub brain repo, self-modifiable prompts, and GitHub Issues for 2-way
  owner communication. Creates the repo from template, fills in CLAUDE.md and
  prompts/role.md, and creates the CCR scheduled trigger automatically.
  Triggers on: brain agent, new agent, scheduled agent, autonomous agent, create agent, new brain.
---

# New Brain Agent

Scaffold a complete autonomous brain agent: GitHub repo + cloud scheduled trigger, wired up and ready to run.

## What Gets Created

1. **GitHub repo** from `sullivanj91/brain-agent-template` — pre-structured with `prompts/`, `inbox/`, `logs/`, `data/`, `scripts/`
2. **`prompts/role.md`** — agent's instructions, filled in with your purpose and schedule; agent evolves this over time
3. **`CLAUDE.md`** — project context auto-loaded by every Claude Code session
4. **CCR scheduled trigger** — bootstraps with `uv sync` then reads `prompts/role.md`

## Workflow

### Step 1: Gather inputs

Ask the user (use AskUserQuestion):
- Agent name (will become `<name>-brain` repo)
- One-line purpose ("what does this agent do?")
- **GitHub username** of the primary owner (used as `BRAIN_USER` — e.g. `alice`)
- **Session structure** — this shapes how `prompts/role.md` is written:
  - *Single routine*: Every run does the same work (e.g. "check metrics and post a Slack summary")
  - *Daily + weekly*: Daily runs execute the work; one run per week the agent reviews performance and self-evolves its own instructions in `prompts/role.md` (recommended for research, trading, monitoring agents)
  - Describe what the daily work is, and (if weekly) what the agent should review and evolve
- Schedule (human-readable, e.g. "every weekday at 9am ET") — convert to UTC cron
- Model: default `claude-sonnet-4-6`; suggest Opus for complex reasoning/strategy sessions, Haiku for lightweight reporting
- GitHub visibility: public or private
- CCR environment to use (list available environments via RemoteTrigger list, or ask user)
- Environment variables needed (names only — user adds values in the environment settings)

### Step 2: Create and populate the repo

```bash
# Create from template
gh repo create <name>-brain --template ArcInstitute/brain-agent-template --public  # or --private

# Clone locally
git clone https://github.com/<owner>/<name>-brain /tmp/<name>-brain
cd /tmp/<name>-brain
```

Fill in `CLAUDE.md` — replace placeholders:
- `{{AGENT_NAME}}` → agent name
- `{{AGENT_PURPOSE}}` → one-line purpose
- `{{TRIGGER_NAME}}` → agent name + "Agent"
- `{{MODEL}}` → chosen model
- `{{SCHEDULE}}` → human-readable schedule

Create user-scoped directories for the primary owner:
```bash
cd /tmp/<name>-brain
mkdir -p inbox/<username> data/<username> logs/<username>/evolution
touch inbox/<username>/.gitkeep data/<username>/.gitkeep logs/<username>/evolution/.gitkeep
# Move the example state files into the user's namespace
mv data/example/observations.md data/<username>/observations.md
mv data/example/learnings.md data/<username>/learnings.md
mv logs/example/evolution/changes.md logs/<username>/evolution/changes.md
# Rename the inbox template dir
mv inbox/example inbox/<username>
```

Fill in `prompts/role.md` — replace placeholders:
- `{{AGENT_NAME}}` → agent name
- `{{MODEL}}` → chosen model
- `{{SCHEDULE}}` → human-readable schedule
- `{{AGENT_TASK_DESCRIPTION}}` → structured session instructions based on the user's session structure answer:

  **If single routine:** Write numbered steps for what the agent does every run. Be specific about data sources, outputs, and where to write results.

  **If daily + weekly:** Structure as two named sections:
  ```
  ## Daily Work
  1. [specific steps for the regular work]
  2. Read data/observations.md and data/learnings.md for context from past sessions
  3. [execute tasks, write outputs to appropriate files]

  ## Weekly Self-Evolution (run once per week, e.g. on [day])
  1. Read logs/evolution/changes.md and data/learnings.md to understand your history
  2. Review recent data/observations.md entries — what patterns are emerging?
  3. Evaluate what's working and what isn't
  4. Update this file (prompts/role.md) with improved instructions — refine your daily steps, add new approaches, remove what isn't useful
  5. Log what you changed and why in logs/evolution/changes.md
  ```

```bash
cd /tmp/<name>-brain
git add -A
git commit -m "init: scaffold from brain-agent-template"
git push origin main
```

### Step 3: Create the CCR trigger

Use the `RemoteTrigger` tool (load with ToolSearch first):

```json
{
  "action": "create",
  "body": {
    "name": "<AgentName> Agent",
    "cron_expression": "<UTC cron>",
    "enabled": true,
    "job_config": {
      "ccr": {
        "environment_id": "<environment_id>",
        "session_context": {
          "model": "<model>",
          "sources": [
            {"git_repository": {"url": "https://github.com/<owner>/<name>-brain"}}
          ],
          "env": {"BRAIN_USER": "<github-username>"},
          "allowed_tools": ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "WebSearch", "WebFetch"]
        },
        "events": [
          {
            "data": {
              "uuid": "<generate lowercase v4 uuid>",
              "session_id": "",
              "type": "user",
              "parent_tool_use_id": null,
              "message": {
                "role": "user",
                "content": "## Setup\n```bash\ncd /home/user && pip install uv -q && cd <name>-brain && uv sync\n```\n\nYour full instructions are in `prompts/role.md` — read that file and follow it exactly."
              }
            }
          }
        ]
      }
    }
  }
}
```

After creating the trigger, update `prompts/role.md` with the trigger ID and push.

### Step 4: Output

Report back:
- GitHub repo URL
- Trigger URL: `https://claude.ai/code/scheduled/<trigger_id>`
- Recommended next step: "Run the trigger manually ('Run now') to seed the initial state before the first scheduled run."

## Key Rules

- Always use `uv add <package>` in the agent prompt, never bare pip install
- Always use `uv run python` in the agent prompt, never bare python3
- Cron expressions are in UTC — convert from user's local time and confirm
- Generate a fresh lowercase UUID v4 for the trigger event
- If the agent needs multiple roles/schedules (like Trader + Reporter), create separate triggers pointing at the same repo with different prompt files (e.g. `prompts/trader.md`, `prompts/reporter.md`)

## Reference Files

- [Brain Pattern overview](references/pattern.md)
- [Example prompts/role.md](references/prompt-template.md)
