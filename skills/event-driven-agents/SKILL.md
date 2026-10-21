---
name: event-driven-agents
description: Coordinate agents and jobs through events, from a single completion wait to a fleet running for hours. Arm notifications at launch, avoid polling with model turns, and use fleet checkpoints and recovery pings when needed.
---

# Event-driven agents

For many workers or jobs supervised for hours, also read [fleet policy](references/fleet.md).
For one wait, use only this file.

Never poll with model turns. Arm one waiter whose completion is the notification,
in the **same turn as the launch**. Do independent work or yield while it waits.
A detached shell process alone does not guarantee that the coordinator wakes.

## Choose the most specific mechanism

1. Use the backend's completion notice. For Paseo, follow the installed `paseo`
   skill's **Waiting** section for `notifyOnFinish` and background
   `send_agent_prompt`; do not duplicate its scheduling or delivery rules.
   Workers assigned a report-back duty use `paseo send --no-wait COORDINATOR_ID
   "ASSIGNMENT STATUS RESULT_PATH"` through an appropriate delivery channel.
2. Otherwise use the harness's tracked background task, such as Claude Code Bash
   with `run_in_background`, whose completion re-invokes the agent. Register the
   waiter there; do not replace tracked execution with an untracked shell `&`.
3. For external signals, put a plain shell waiter in that tracked task. The
   loops below are process checks, not recurring model turns. Paths are examples;
   replace project/run placeholders and handle command failures. The default
   `~/.agents/runs/<project>/<run>/` is shared across agents; tilde expands below.

   ```sh
   run_dir=~/.agents/runs/'<project>/<run>'
   # File published by a worker, preferably with an atomic rename.
   until test -f "$run_dir/results/worker.result.md"; do command sleep 60; done

   # Alert directory: matches nothing safely when empty.
   until command find "$run_dir/alerts" -type f -name '*.md' -print -quit | command grep -q .; do command sleep 60; done

   # Hypothetical same-day target, later today: 18:00.
   until test "$(command date +%H%M)" -ge 1800; do command sleep 60; done

   # Remote producer publishes JOB_DONE only after durable outputs exist.
   ssh REMOTE_HOST 'run_dir=~/.agents/runs/"<project>/<run>"; until test -f "$run_dir/JOB_DONE"; do command sleep 60; done'
   ```

   Use unique run targets so an old marker cannot complete a new assignment.
   A dropped remote connection is a failed waiter, not job completion. Prefer
   a native job-completion event when the remote backend exposes one.
4. For a future time, prefer a one-shot harness wakeup when available. Claude
   Code's `CronCreate` can schedule the check; remove the job after it fires.
   Without a scheduler, use a tracked timer. Do not invent a notification channel.

## Waiter traps

- Use absolute paths for targets, logs and result files. A relative path can
  silently wait forever after an unrelated working-directory change.
- Consume or archive handled alerts before re-arming, or the next waiter fires
  immediately. Do not remove another worker's unconsumed alert.
- The time-of-day loop cannot cross midnight. For another day, schedule a full
  date/time. When a predicted reset fires, reread actual usage before acting.
- Queue messages or deliver when idle if the backend supports it; do not interrupt
  busy agents merely to notify them. `--no-wait` frees the sender, not a promise
  that recipient delivery waits for idle. Follow the backend's verified semantics.
- Keepalive during long waits belongs to `paseo-cache-warmer` when installed.
  If absent, let the wait stand without inventing recurring model pings.

## On notification

Verify the event itself: read the result's first line and evidence, inspect the
remote done marker and durable outputs, or reread the real usage number. A
notification, quiet pane or process exit alone does not prove success. Record
failures and re-arm only after settling them. For recovery across coordinator
changes, use `long-run-handoff`; if absent, save the event and result paths in
private durable state. If `paseo` is absent, use the harness mechanism above.

