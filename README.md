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

### Separate accounts (independent quotas)
By default the dashboard treats everyone's Claude Code entries as one shared
pool with one shared 5-hour limit — right for a team on one account. If
instead you (or some subset of people) each have your **own separate**
Claude account with its own separate limit, pooling those together produces
a meaningless combined "share %" and capacity bar. Tag those entries with
`USAGE_TRACKER_ACCOUNT` and the dashboard gives each account its own card,
with its own 5h windows and its own self-calibrated ceiling — never summed
with anyone else's:

```json
{
  "hooks": {
    "SessionEnd": [
      { "hooks": [{ "type": "command", "command": "USAGE_TRACKER_USER=\"Claude Code-mac-air\" USAGE_TRACKER_ACCOUNT=\"mac-air\" node ~/.usage-tracker/trigger.js" }] }
    ]
  }
}
```

Rows without this tag keep behaving exactly as before (one shared pool), so
existing team setups are unaffected. This also works per-tool: e.g. keep
Codex shared across the team while splitting Claude Code into separate
per-person accounts — just set `USAGE_TRACKER_ACCOUNT` only in the Claude
Code hook.

[Cloudlfare worker URL](https://usage-tracker-worker.omer-aj.workers.dev)
