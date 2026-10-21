# Waiting guide

[The skill](../skills/event-driven-waiting/SKILL.md) keeps the choice of wakeup
mechanism and its failure checks in one place. Its examples cover a result file,
an alert directory, a same-day time and a remote done-marker file.

A backend completion notice avoids maintaining another watcher. Where none
exists, a tracked background waiter can turn an external event into a completion
notification. A bare detached process has no general way to wake the model.
If the harness has neither notifications nor a scheduler, report that limitation
and leave durable state for the next user-driven turn.

## Companions

- `event-driven-coordination` owns fleet policy, checkpoints, event-based hand-backs
  and the hourly recovery ping; this skill owns the mechanics of one wait.

- The installed `paseo` skill's Waiting section owns Paseo completion delivery.
  This skill refers to it rather than copying its tool defaults.
- `paseo-cache-warmer` owns keepalive during long waits when it is installed.
- [long-run-handoff](https://github.com/NightMachinery/long-run-handoff) preserves
  recovery files and checklist ownership across coordinator changes.

Read only the backend guidance needed for the selected mechanism. Save worker
identifiers, event targets and output paths in private state, outside public
repositories. Verify the durable event before changing task status; notification
success and job success are separate facts.
