# event-driven-agents

Coordinate long jobs without spending model turns on status checks. The core
skill selects and verifies one event wait. A separate fleet reference adds
worker checkpoints, event-based hand-backs, milestone timers and a recovery ping
when supervising many jobs for hours.

## Install

```sh
npx skills add NightMachinery/event-driven-agents --global \
  --agent claude-code codex --skill event-driven-agents
```

For another Claude profile, set `CLAUDE_CONFIG_DIR` and install with
`--agent claude-code`. Start with [the guide](docs/event-driven-agents.md).
No custom shell setup is required. MIT licensed, see [LICENSE](LICENSE).
