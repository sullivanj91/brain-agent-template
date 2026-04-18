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
  role.md               ← agent reads + evolves its own instructions here
logs/
  <username>/
    evolution/           ← per-user changelog of decisions and changes
data/
  <username>/
    observations.md      ← per-user ongoing patterns and findings
    learnings.md         ← per-user what worked, what didn't
inbox/
  <username>/            ← you write directives here; agent reads them every session
  resolved/              ← move handled items here
scripts/                 ← agent-created analysis and execution code
CLAUDE.md                ← context loaded automatically by every Claude Code session
```

`<username>` matches the `BRAIN_USER` env var set in each CCR trigger — this is how multiple users can share one repo with fully isolated state.

## 2-Way Communication

**Owner → Agent (standing directives):** Write `.md` files in `inbox/<username>/` (where `<username>` matches the `BRAIN_USER` env var on the trigger). The agent reads all files there at the start of every session. Move them to `inbox/resolved/` when done.

**Owner → Agent (one-time instructions):** Create a GitHub Issue labeled `directive`. The agent reads open directive issues each session, acts on them, comments to acknowledge, and closes them. You can create these from the GitHub UI or via `gh issue create --label directive`.

**Agent → Owner:** The agent creates GitHub Issues labeled `request` when it needs something (an API key, a new data source, a permission). You get notified by email. Close the issue once resolved.

## Multi-User Support

Multiple users can share one brain repo with fully isolated state. Each user gets their own CCR trigger with a unique `BRAIN_USER` value:

```
CCR Trigger (Alice)          CCR Trigger (Bob)
BRAIN_USER=alice             BRAIN_USER=bob
        └──────────┬──────────────┘
                   ▼
            Same git repo / main branch
                   │
        ┌──────────┴────────────────┐
        ▼                           ▼
inbox/alice/                  inbox/bob/
data/alice/                   data/bob/
logs/alice/evolution/         logs/bob/evolution/
```

To add a user, use the `/brain-manage` skill's **Add a User** workflow.

## Adding Users

To give another person their own isolated namespace in this brain, open this repo in Claude Code and run:

```
/brain-manage add user
```

It will ask for their GitHub username, create their `inbox/<username>/`, `data/<username>/`, and `logs/<username>/evolution/` directories, commit them, and create a new CCR scheduled trigger with `BRAIN_USER=<username>` wired up automatically.

## Upgrading the Framework

The harness (skills, plugin config, inbox structure) is versioned via GitHub releases. Your agent-owned files (`prompts/role.md`, `CLAUDE.md`, `data/`, `logs/`) are never touched.

To upgrade, open your brain repo in Claude Code and run:

```
/brain-manage upgrade framework
```

It will:
1. Show your current version and available releases
2. Recommend the latest **safe update** (same major version — no breaking changes)
3. Display the release notes for the target version
4. Warn if upgrading across a major version boundary (breaking changes possible)
5. Apply the upgrade and push

To check what version you're on without upgrading:
```bash
cat .brain-agent-version
```

To see all available releases:
```bash
gh release list --repo ArcInstitute/brain-agent-template
```

## Releasing a New Harness Version

For maintainers of this template — to publish a new versioned release:

```
/brain-manage cut release
```

It will ask whether this is a patch / minor / major bump, update `.brain-agent-version`, tag, push, and create a GitHub release with auto-generated notes.

**Semver guide:**
- `patch` (`1.0.0 → 1.0.1`): bug fixes, doc updates, no behavior change
- `minor` (`1.0.0 → 1.1.0`): new features, backward-compatible
- `major` (`1.0.0 → 2.0.0`): breaking changes to harness structure or agent file formats

## Skills (auto-loaded when this repo is open in Claude Code)

| Skill | Description |
|---|---|
| `/new-brain-agent` | Scaffold a new brain agent from this template |
| `/brain-manage` | Check status, upgrade framework, add users, cut releases |

## Dependency Management

Always use `uv add <package>` — never bare `pip install`. This installs the package and updates `pyproject.toml` so all future agent sessions pick it up via `uv sync`.
