# Detection observers

Every detection produces a small metadata payload. It doesn't include the model's thinking or tool arguments, and observing it can't cancel or change the detection that already happened.

## Available observers

There are three ways to receive the payload:

- `loop-police:detection` is always emitted on Pi's extension event bus. Another extension can subscribe with `pi.events.on("loop-police:detection", handler)`; [`examples/listener-extension.ts`](../examples/listener-extension.ts) and [pi-input-bar](https://github.com/sebaxzero/pi-input-bar) show how.
- `HOOK_CMD` runs an external command with the JSON payload as its final argument. It runs without a shell and splits arguments on whitespace, so paths containing spaces aren't supported. Output and exit status are ignored, it is stopped after `HOOK_TIMEOUT_MS`, and a failure is only shown once per session. [`examples/hook.mjs`](../examples/hook.mjs) can send an OS notification and, with `LOOP_POLICE_NTFY_TOPIC`, a phone notification.
- `HOOK_LOG` adds one JSON payload per line to a JSONL file. Relative paths are resolved from the session's working directory.

You can enable both external observers during a session with:

```text
/loop-police set HOOK_CMD=node /path/to/hook.mjs HOOK_LOG=/path/to/loops.jsonl
```

Use `/loop-police save` afterwards if you'd like to keep those settings.

## Payload

The payload has this shape:

```json
{
  "event": "tool_loop",
  "timestamp": "2026-07-17T14:03:22.123Z",
  "model": { "id": "qwen3:14b", "name": "Qwen3 14B", "provider": "ollama" },
  "sessionId": "…",
  "sessionFile": "/path/to/session.jsonl",
  "cwd": "/path/to/project",
  "turnIndex": 12,
  "consecutiveLoops": 0,
  "details": { "toolName": "bash", "windowSize": 3, "banned": false }
}
```

`model` will be `null` when no model is selected. The contents of `details` depend on the event:

| Event | Details |
|---|---|
| `thinking_loop`, `semantic_loop`, `output_loop`, `output_semantic_loop` | `{ stream, kind, escalated }` |
| `stagnation` | `{ window, threshold }` |
| `file_scan_loop` | `{ toolName, path, count }` |
| `redundant_reread` | `{ toolName, path, count, window }` |
| `search_spiral` | `{ toolName, pattern, paths }` |
| `tool_loop` | `{ toolName, windowSize, banned }` |
| `rederived_reasoning` | `{ streak }` |
