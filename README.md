# pi-loop-police

`pi-loop-police` is a [pi](https://pi.dev) extension that tries to stop models from entering thinking/output/tool-call loops. It uses configurable [detectors](docs/DETECTORS.md) to interrupt the current stream or tool call whenever a loop is identified, then attempts to re-orient the agent.

## Install

```bash
pi install [-l] git:github.com/raffaele-99/pi-loop-police.git
```

## Use

It'll be active in any session where the extension is installed.

As mentioned above, it triggers automatically based on the detector configuration; you can read more about the configuration options in [this file](docs/CONFIG.md).

> [!NOTE] Detections may add recovery messages to the session context. If configured, [observers](docs/OBSERVERS.md) can also write detection metadata to a JSONL file.

### Commands

Should you want to view or change the active settings mid-session:

| Command | Effect |
|---|---|
| `/loop-police` | Show state and config |
| `/loop-police reset` | Clear session state |
| `/loop-police set KEY=VAL [KEY=VAL …]` | Change session config |
| `/loop-police save` | Save current config |


### Skills

The extension includes two skills that provide agent-facing help:

- `loop-police-help`: commands, config, and install paths.
- `loop-police-postmortem`: detection analysis and tuning.

Works with Pi-normalised reasoning, including OpenAI-compatible Qwen and DeepSeek models. [pi-canary](https://github.com/sebaxzero/pi-canary) yields on aborted turns.

## License

MIT
