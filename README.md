# usage-tracker

Tracks Claude Code / Codex usage across a team sharing one account.

- `usage-tracker-worker/` — Cloudflare Worker backend (receives + stores usage events)
- `collector/` — local script triggered by a Claude Code `SessionEnd` hook
- `docs/` — public dashboard, served via GitHub Pages. Per shared 5-hour session it
  shows each user's **share** of what the account actually burned — measured in cost,
  not raw tokens, so Opus vs Sonnet usage compares fairly (who to blame when you're
  both waiting on a reset) — and how full the session got vs your priciest session on
  record. Claude's real 5-hour limit isn't a published number, so that priciest
  session is the auto ceiling — pin `SESSION_COST_LIMIT_USD` in `docs/index.html`
  only if you know your plan's actual budget.

## Setup for teammates
Run:
```sh
npx github:omerAJ/usage-tracker
```

This installs the collector and wires up Claude Code / Codex hooks automatically.
If you use Codex, open it once afterward and run `/hooks` to trust the new hook.

Dashboard: https://omerAJ.github.io/usage-tracker/

### Custom user label
By default each event is tagged with `os.userInfo().username` (the OS account
running the hook). If you use multiple machines under one Claude/Codex account
and want them to show up as distinct, readable rows on the dashboard, set
`USAGE_TRACKER_USER` in the hook command instead of editing any code:

```json
{
  "hooks": {
    "SessionEnd": [
      { "hooks": [{ "type": "command", "command": "USAGE_TRACKER_USER=\"Claude Code-mac-air\" node ~/.usage-tracker/trigger.js" }] }
    ]
  }
}
```

This only changes what your own machine reports — everyone else's events are
unaffected.

[Cloudlfare worker URL](https://usage-tracker-worker.omer-aj.workers.dev)
