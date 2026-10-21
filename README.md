# event-driven-waiting

Wait for long jobs without spending model turns on status checks. The skill
chooses the backend's completion event first, then a tracked background waiter
or scheduled wakeup, and verifies the durable result when it arrives.

## Install

```sh
npx skills add NightMachinery/event-driven-waiting --global \
  --agent claude-code codex --skill event-driven-waiting
```

For another Claude profile, set `CLAUDE_CONFIG_DIR` and install with
`--agent claude-code`. Start with [the guide](docs/event-driven-waiting.md).
No custom shell setup is required. MIT licensed, see [LICENSE](LICENSE).
