# Hive

A file-based coordination hive for a fleet of Claude Code agents, run by one
harness process. Agents share a workspace folder instead of a message bus:
the harness moves messages between `inbox/` and `outbox/`, owns git, and
exposes live fleet state (`fleet.json`) alongside the roster (`registry.json`)
and the event feed (`log.jsonl`).

## How it works

- **Workspace per agent** — `agents/<id>/` holds `identity.md`, `memory.md`,
  `inbox/` (with `.done/`) and `outbox/`. Agents read their inbox at task
  start, move handled messages to `.done/`, and drop replies in `outbox/`.
- **`god` coordinator** — a designated `god` agent registers itself, watches
  the fleet, schedules work (`scheduler → god` requests) and keeps
  situational awareness from `fleet.json` + `registry.json` + `log.jsonl`.
- **Guardrails** — a circuit breaker escalates `steer → constrain → stop` on
  runaway loops, error storms or overspending; the floor carries a global
  token budget with per-agent `tokenCap`s.
- **Commands** — `COMMANDS.md` is the reference: slash commands operate only
  on the agent's own session, CLI commands can spawn or query the fleet.

## Layout

```
agents/<id>/      agent workspaces (identity, memory, inbox, outbox)
bin/              harness helpers
spawn-requests/   queued agent spawn requests
tasks.json        task board
registry.json     agent roster
hooks.token       local shared hook token (git-ignored)
log.jsonl         event feed
```

## Note

`hooks.token` is deliberately excluded from version control — it is a
shared secret for the harness hooks.
