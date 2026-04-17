# User Directives

Write `.md` files in this directory to send standing instructions to the agent.
The agent reads all files here at the start of every session.

This directory is a template placeholder — rename it to match the `BRAIN_USER` value
you set in your CCR trigger (e.g. `inbox/alice/` for `BRAIN_USER=alice`).

Move handled files to `inbox/resolved/` when no longer needed.

## Examples

**`pause.md`**
```
Skip trading/work this week. Markets closed for holiday.
```

**`focus.md`**
```
For the next month, prioritize dividend-paying stocks only.
```

**`override.md`**
```
Do not take any positions in NVDA until further notice — earnings risk.
```
