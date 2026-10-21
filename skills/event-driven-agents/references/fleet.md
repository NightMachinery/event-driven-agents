# Fleet policy

Read this only when supervising many workers or jobs for hours. The core
[skill](../SKILL.md) supplies the event-waiting mechanics.

Terms used below:

- An **event** is a state transition somebody must act on: a job drained, a
  node published and verified, a preemption, a stall, a failure, a decision
  that needs the coordinator or the user. Progress ("7 of 12 done") is not an
  event.
- A **hand-back** is the message a worker sends its coordinator. It is sent on
  events only.
- A **one-shot timer** is a scheduled wakeup for a milestone that is expected
  at a known time.
- The **recovery ping** is one recurring hourly wakeup whose only job is to
  notice an event that was missed.
- A **checkpoint** is the worker's file on disk with the current facts, the
  running commands, the projected times and a resume procedure a stranger can
  follow.

## Why fleets need recovery

Per-cell progress reports can turn a coordinator into a relay, consuming its
attention while postponing decisions. Checkpoints hold progress; hand-backs
announce transitions that need action.

A watch or its owning session can die, a one-shot can be missed, and a hand-back
can arrive while its recipient is unavailable. Silence cannot distinguish these
failures from ongoing work. The recovery ping catches missed fleet events.

## Three layers

1. **Watches** for anticipated worker transitions, using the core skill.
2. **One-shot timers** for milestones with a known time: a quota reset, a
   projected drain, a deadline. The timer's prompt carries the check and the
   action, so it is self-contained after a compaction.
3. **One recovery ping**, hourly, at an off-minute. Never more than one, and
   never a second poller next to it: when the ping misses a class of event,
   widen its checklist.

## Rules for a worker

- Hand back only on events: resource drained and verified, stall, preemption
  or failure, decision needed, all done. Never a progress count.
- Keep the checkpoint current instead of reporting. It holds: what is running
  where, the measured rates and projected times, the exact NEXT COMMANDS, the
  ids of any timers or watches you armed, and a RESUME block that says what to
  do first after a pause. The coordinator reads the checkpoint, not your
  transcript, so a stale checkpoint is a lie.
- One lean watch, as a script file. Test it once in single-pass mode before
  arming it, filter to the lines you would act on, and make it emit on every
  terminal state, not only success. Watches expire: on expiry re-arm and note
  the time in the checkpoint.
- Query remote state with the cheapest call that answers the question (a
  plain batch-mode ssh, a single list command), on an interval measured in
  tens of minutes. A heavy interactive shell wrapper per poll multiplied by
  several workers is what got waiters killed for memory.
- When you are told to pause, finish the in-flight verify or publish, write
  the checkpoint, and stop. Autonomous jobs keep running on their own.

## Rules for the coordinator

- Give each launched or resumed worker a fallback one-shot for the time by
  which it should have handed back. Resuming by message counts as a launch;
  use the core skill for arming and delivery mechanics.
- Keep exactly one recovery ping. Its prompt is one facts call (the resource
  list, the checkpoints' modification times and last lines), a comparison
  against the latest dated line of your own state file, and a fixed list of
  discrepancies to act on. Otherwise it appends nothing and replies in one
  line. The template is below.
- After compaction or restart, check which scheduled jobs remain armed;
  session-scoped jobs may be gone. Read checkpoints and restore the missing
  recovery ping and fallback one-shots before continuing. Record their ids in
  state so the next reader can tell what is armed.
- Do not idle autonomous work to protect quota. Jobs that run on their own
  once started should be started; a brief pause at the reset boundary is
  cheaper than an idle hour of machines.
- A missed event is a bug in the checklist, not bad luck. When the ping or a
  human finds one, add its signature to the ping prompt.

## The recovery ping prompt, as a template

Adapt the names; keep the shape. The stale-check threshold below is a policy
default, not a measurement from a run.

```
Hourly recovery ping. This is the missed-event catch, not a progress poll.
Do exactly this: one facts call (list the live resources; stat the worker
checkpoints and read their last lines), then compare with the latest dated
line in STATE.md. Act only on a discrepancy:
- a resource that its checkpoint says is published and verified but still
  exists: re-verify the durable copy, then release it;
- a checkpoint untouched for more than 90 minutes while its resources run:
  message that worker once;
- a dead watcher, loop or pane: restart it with its recorded command;
- a worker silent past the milestone it was given: read its checkpoint and
  act on its NEXT COMMANDS.
Otherwise append nothing and reply in one line.
```

## Anti-patterns

- Per-cell progress hand-backs and coordinator relays of them.
- A watch typed inline and armed untested.
- Two pollers covering the same fleet, one of them "just to be safe".
- Treating a quiet notification stream as proof that nothing changed.
- Releasing a resource on a completion log line without counting the durable
  artifacts.
- Pausing everything, including autonomous jobs, because a quota window is
  nearly used up.

## Incident lessons

- A process-wide termination killed watchers and publication loops; scheduled
  one-shots did not act, and drained compute idled until a human noticed. An
  hourly recovery ping would have detected dead loops and stale checkpoints.
- Operators reported progress and polled repeatedly, consuming coordinator
  attention before decisions were needed. Checkpoints plus event-only hand-backs
  reduced the traffic while preserving intervention at meaningful transitions.

Use `long-run-handoff` for durable recovery and result-file ownership, and
`quota-management` for the coordinator's allowance. If either is absent, retain
private checkpoints and report the missing companion rather than inventing tools.
