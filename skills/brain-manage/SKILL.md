---
name: brain-manage
description: |
  Manage an autonomous Brain Agent — check status, review recent activity, respond to
  agent requests, send directives, and update the agent's prompt. Works on any brain
  agent repo that follows the Brain Agent pattern.
  Triggers on: brain manage, agent status, check agent, respond to agent, agent request, brain status, update agent prompt.
---

# Brain Manage

Manage a running brain agent from within its repo.

## Available Actions

### Check Status
Review what the agent has been doing:
```bash
# Recent commits
git log --oneline -20

# Open agent requests (issues labeled 'request')
gh issue list --label request --state open

# Today's activity
git log --since="24 hours ago" --oneline
```
Read `logs/evolution/changes.md` for strategic decisions. Read `data/observations.md` for recent findings.

### Respond to Agent Requests
View open requests:
```bash
gh issue list --label request --state open
```
To fulfill a request: handle it, then close the issue:
```bash
gh issue close <number> --comment "Done — added POLYGON_API_KEY to the environment."
```

### Send a Directive
**One-time directive** (agent will act + close):
```bash
gh issue create --title "Directive: <action>" --body "<instructions>" --label directive
```

**Standing directive** (persists across sessions):
Create a `.md` file in `inbox/owner/`:
```bash
# Example: pause the agent for a week
echo "Skip all activity until 2026-04-20. Markets are thin." > inbox/owner/pause.md
git add inbox/owner/pause.md && git commit -m "directive: pause until Apr 20" && git push
```
Remove from `inbox/owner/` (or move to `inbox/resolved/`) when no longer needed.

### Update Agent Prompt
Edit `prompts/role.md` directly — the agent will read the updated instructions on its next session:
```
open prompts/role.md
# or use the Edit tool
```
Commit and push after editing.

### View Trigger Schedule
```bash
# List all triggers (requires RemoteTrigger tool)
# Or visit: https://claude.ai/code/scheduled
```

### Run Agent Now
Visit `https://claude.ai/code/scheduled` and click "Run now" on the relevant trigger.
Or use the RemoteTrigger tool:
```
Load RemoteTrigger via ToolSearch, then: RemoteTrigger({action: "run", trigger_id: "<id>"})
```
The trigger ID is in `prompts/role.md` at the top.

### Upgrade Framework
Update the brain-agent harness (skills, plugin config, inbox structure) without touching your agent's data or prompts.

Check current version:
```bash
cat .brain-agent-version
```

Apply upgrade (fetches framework files from the template, leaves agent files untouched):
```bash
git fetch --depth 1 https://github.com/sullivanj91/brain-agent-template.git main
git checkout FETCH_HEAD -- skills/ .claude-plugin/ inbox/README.md inbox/owner/README.md .brain-agent-version
git diff --staged --stat
git commit -m "chore: upgrade brain-agent framework to $(cat .brain-agent-version)"
git push
```

**Files upgraded** (framework-owned): `skills/`, `.claude-plugin/`, `inbox/README.md`, `inbox/owner/README.md`, `.brain-agent-version`

**Files preserved** (agent-owned): `prompts/role.md`, `CLAUDE.md`, `pyproject.toml`, `data/`, `logs/`, `scripts/`, `inbox/owner/*.md`

For skill-only updates (no repo changes needed), run `/plugin marketplace update` instead.

## Common Workflows

**"The agent requested a new API key"**
1. `gh issue list --label request --state open` — read the request
2. Add the key to the cloud environment at claude.ai
3. `gh issue close <number> --comment "Added to environment."` — notify the agent

**"I want to change the agent's strategy"**
1. Edit `prompts/role.md` with your changes
2. Optionally add a note to `inbox/owner/reason.md` explaining the change
3. Commit + push — takes effect next session

**"The agent seems stuck / made a bad decision"**
1. `git log --oneline -10` — see what happened
2. Edit `prompts/role.md` to correct course
3. Optionally revert bad commits: `git revert <hash>`
4. Add a `directive` issue explaining what to do differently

**"I want to upgrade the framework"**
1. `cat .brain-agent-version` — see current version
2. Run the upgrade commands in **Upgrade Framework** above
3. Review the staged diff before committing
