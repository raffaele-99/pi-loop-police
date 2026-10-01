# pi-loop-police

`pi-loop-police` is a [pi](https://pi.dev) extension that tries to stop models from entering thinking/output/tool-call loops.

## Install

```bash
pi install [-l] git:github.com/raffaele-99/pi-loop-police.git
```

## Use

The "protection" starts in any session where the extension is installed.

> [!NOTE] See [Detectors](docs/DETECTORS.md) for triggers, recovery behavior, and tuning.

| Command | Effect |
|---|---|
| `/loop-police` | Show state and config |
| `/loop-police reset` | Clear session state |
| `/loop-police set KEY=VAL [KEY=VAL …]` | Change session config |
| `/loop-police save` | Save current config |

String values extend to the next `KEY=` token. Invalid numeric values are rejected or defaulted.

> [!NOTE] See [Configuration](docs/CONFIG.md) for settings and recovery messages, and [Observers](docs/OBSERVERS.md) for integrations.

Two bundled skills provide agent-facing help:

- `loop-police-help`: commands, config, and install paths.
- `loop-police-postmortem`: detection analysis and tuning.

Works with Pi-normalized reasoning, including OpenAI-compatible Qwen and DeepSeek models. [pi-canary](https://github.com/sebaxzero/pi-canary) yields on aborted turns.

## License

MIT
