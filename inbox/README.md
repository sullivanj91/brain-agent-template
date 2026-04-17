# Inbox

Two-way communication between users and the agent.

## User → Agent: `inbox/<username>/`

Each user gets their own inbox directory, scoped by `$BRAIN_USER`. Write `.md` files here to send standing instructions to the agent. The agent reads all files in its inbox directory at the start of every session.

The inbox directory must match the `BRAIN_USER` value set in the CCR trigger. For example, if `BRAIN_USER=alice`, the agent reads `inbox/alice/`.

Examples:
- `pause.md`: "Skip this week — markets are closed for the holiday"
- `focus.md`: "Prioritize X for the next month"
- `override.md`: "Do not touch NVDA this week"

Move files to `inbox/resolved/` once they've been acted on.

## Agent → Owner: GitHub Issues

The agent creates GitHub Issues labeled `request` when it needs something from you (API keys, permissions, data sources). You'll receive an email notification. Close the issue once resolved.

To send a directive proactively, create a GitHub Issue labeled `directive`. The agent will read it, act, comment to acknowledge, and close it.

## Adding a New User

1. Create their directories:
   ```bash
   mkdir -p inbox/<username> data/<username> logs/<username>/evolution
   touch inbox/<username>/.gitkeep data/<username>/.gitkeep logs/<username>/evolution/.gitkeep
   git add . && git commit -m "feat: add user <username>" && git push
   ```
2. Create a new CCR trigger with `BRAIN_USER=<username>` in the environment variables.

## Backward Compatibility

Existing brains using `inbox/owner/` continue to work unchanged by setting `BRAIN_USER=owner` in the CCR trigger environment.
