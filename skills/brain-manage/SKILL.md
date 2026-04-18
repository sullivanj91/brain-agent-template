---
name: brain-manage
description: |
  Manage an autonomous Brain Agent — check status, review recent activity, respond to
  agent requests, send directives, and update the agent's prompt. Works on any brain
  agent repo that follows the Brain Agent pattern.
  Triggers on: brain manage, agent status, check agent, respond to agent, agent request, brain status, update agent prompt, add user, new user, onboard user, upgrade framework, upgrade harness, check for updates, cut release, new release, tag release.
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
Create a `.md` file in `inbox/<username>/` (where username matches the `BRAIN_USER` env var on the trigger):
```bash
# Example: pause the agent for a week
echo "Skip all activity until 2026-04-20. Markets are thin." > inbox/<username>/pause.md
git add inbox/<username>/pause.md && git commit -m "directive: pause until Apr 20" && git push
```
Remove from `inbox/<username>/` (or move to `inbox/resolved/`) when no longer needed.

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

#### Step 1 — Check current version and available releases

```bash
# What version is installed?
cat .brain-agent-version

# What releases are available?
gh release list --repo ArcInstitute/brain-agent-template --limit 10
```

Parse the output and determine:
- **Current version** from `.brain-agent-version` (e.g. `1.0.0`)
- **Safe update** — latest release with the same major version (no breaking changes)
- **Latest overall** — may be a higher major version (breaking changes possible)

Present a summary to the user, e.g.:
> You're on **v1.0.0**. Latest safe update: **v1.2.0** (same major). Latest overall: **v2.0.0** (major bump — may have breaking changes).

#### Step 2 — Show release notes for the target version

```bash
gh release view <target-tag> --repo ArcInstitute/brain-agent-template
```

Summarize the key changes from the release body.

#### Step 3 — Warn on major version crossing

If the target version has a higher major number than current, show an explicit warning:
> ⚠️ **Breaking change warning:** Upgrading from v1.x to v2.x may require manual changes to agent-owned files (prompts/role.md, CLAUDE.md). Review the release notes before proceeding.

Ask the user to confirm before continuing.

#### Step 4 — Apply the upgrade

```bash
git fetch --depth 1 https://github.com/ArcInstitute/brain-agent-template.git refs/tags/<target-tag>
git checkout FETCH_HEAD -- skills/ .claude-plugin/ inbox/README.md inbox/example/README.md .brain-agent-version
git diff --staged --stat
git commit -m "chore: upgrade brain-agent framework to <target-tag>"
git push
```

**Files upgraded** (framework-owned): `skills/`, `.claude-plugin/`, `inbox/README.md`, `inbox/example/README.md`, `.brain-agent-version`

**Files preserved** (agent-owned): `prompts/role.md`, `CLAUDE.md`, `pyproject.toml`, `data/`, `logs/`, `scripts/`, `inbox/<username>/*.md`

For skill-only updates (no repo changes needed), run `/plugin marketplace update` instead.

### Cut a Release

Use this when working in the `brain-agent-template` repo itself to publish a new versioned release. Releases are what downstream brain agents upgrade to via the **Upgrade Framework** workflow above.

#### Step 1 — Determine bump type

Ask the user (or infer from recent commits):
- **patch** (`1.0.0 → 1.0.1`): bug fixes, doc updates, no behavior change
- **minor** (`1.0.0 → 1.1.0`): new features, backward-compatible
- **major** (`1.0.0 → 2.0.0`): breaking changes to harness structure or agent-owned file formats

#### Step 2 — Bump `.brain-agent-version` and commit

```bash
# Read current version, calculate next, write it
echo "<new-version>" > .brain-agent-version
git add .brain-agent-version
git commit -m "chore: bump version to v<new-version>"
git push
```

#### Step 3 — Tag and publish

```bash
git tag v<new-version>
git push origin v<new-version>
gh release create v<new-version> --generate-notes --title "v<new-version>"
```

GitHub auto-generates release notes from commits since the last tag. The release is immediately available for downstream brain agents to upgrade to.

### Add a User

Guided workflow to add a new user namespace and wire up their CCR trigger.

#### Step 1 — Gather inputs (AskUserQuestion)
- GitHub username to add
- CCR environment to use (load RemoteTrigger via ToolSearch and list environments, or ask user)
- Model (default `claude-sonnet-4-6`)

#### Step 2 — Create directories and commit

Read `prompts/role.md` to get the agent name and repo URL. Then:

```bash
mkdir -p inbox/<username> data/<username> logs/<username>/evolution
touch inbox/<username>/.gitkeep data/<username>/.gitkeep logs/<username>/evolution/.gitkeep
cp inbox/example/README.md inbox/<username>/README.md
cp data/example/observations.md data/<username>/observations.md
cp data/example/learnings.md data/<username>/learnings.md
cp logs/example/evolution/changes.md logs/<username>/evolution/changes.md
git add . && git commit -m "feat: add user <username>" && git push
```

#### Step 3 — Create the CCR trigger

Load RemoteTrigger via ToolSearch, then create a new trigger. Use the same schedule, model, and repo URL as the primary trigger (read from `prompts/role.md`):

```json
{
  "action": "create",
  "body": {
    "name": "<AgentName> Agent — <username>",
    "cron_expression": "<same UTC cron as primary trigger>",
    "enabled": true,
    "job_config": {
      "ccr": {
        "environment_id": "<environment_id>",
        "session_context": {
          "model": "<model>",
          "sources": [{"git_repository": {"url": "<repo-url>"}}],
          "env": {"BRAIN_USER": "<username>"},
          "allowed_tools": ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "WebSearch", "WebFetch"]
        },
        "events": [{
          "data": {
            "uuid": "<generate lowercase v4 uuid>",
            "session_id": "",
            "type": "user",
            "parent_tool_use_id": null,
            "message": {
              "role": "user",
              "content": "## Setup\n```bash\ncd /home/user && pip install uv -q && cd <repo-name> && uv sync\n```\n\nYour full instructions are in `prompts/role.md` — read that file and follow it exactly."
            }
          }
        }]
      }
    }
  }
}
```

#### Step 4 — Output
- Trigger URL: `https://claude.ai/code/scheduled/<trigger_id>`
- Recommend: "Run the trigger manually ('Run now') to seed the initial state before the first scheduled run."

## Common Workflows

**"The agent requested a new API key"**
1. `gh issue list --label request --state open` — read the request
2. Add the key to the cloud environment at claude.ai
3. `gh issue close <number> --comment "Added to environment."` — notify the agent

**"I want to change the agent's strategy"**
1. Edit `prompts/role.md` with your changes
2. Optionally add a note to `inbox/<username>/reason.md` explaining the change
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
