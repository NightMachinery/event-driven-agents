# Event-driven agents guide

Load [the skill](../skills/event-driven-agents/SKILL.md) for a single wait or
fleet coordination. Its core selects the completion mechanism, handles waiter
traps and verifies the durable event. Read
[fleet policy](../skills/event-driven-agents/references/fleet.md) only for many
workers or jobs supervised for hours. It owns checkpoints, event-only hand-backs,
milestone timers and the one hourly recovery ping.

A backend completion notice avoids another watcher. Where none exists, a tracked
background waiter can turn an external event into a completion notification.
A bare detached process has no general way to wake the model. If the harness has
neither notifications nor a scheduler, save state for the next user-driven turn.

## Companions

- The installed `paseo` skill's Waiting section owns Paseo completion delivery.
- `paseo-cache-warmer` owns keepalive during long waits when installed.
- [long-run-handoff](https://github.com/NightMachinery/long-run-handoff) preserves
  private recovery files and checklist ownership across coordinator changes.

Use `~/.agents/runs/<project>/<run>/` as the default shared run directory across
agent harnesses. Record worker identifiers, event targets and outputs there,
outside repositories. Neither a notification nor a process exit proves job success.
