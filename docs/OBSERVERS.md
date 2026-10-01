# Detection observers

Every detection emits metadata, never thinking or tool arguments:

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

`model` may be `null`.

| Event(s) | `details` |
|---|---|
| `thinking_loop`, `semantic_loop`, `output_loop`, `output_semantic_loop` | `{ stream, kind, escalated }` |
| `stagnation` | `{ window, threshold }` |
| `file_scan_loop` | `{ toolName, path, count }` |
| `redundant_reread` | `{ toolName, path, count, window }` |
| `search_spiral` | `{ toolName, pattern, paths }` |
| `tool_loop` | `{ toolName, windowSize, banned }` |
| `rederived_reasoning` | `{ streak }` |

Observers cannot alter detection:

- `loop-police:detection`: always-on extension event. Subscribe with `pi.events.on("loop-police:detection", handler)`. See [`examples/listener-extension.ts`](../examples/listener-extension.ts) and [pi-input-bar](https://github.com/sebaxzero/pi-input-bar).
- `HOOK_CMD`: direct, shell-free, fire-and-forget command receiving payload JSON as its last argument. It uses whitespace splitting, so paths cannot contain spaces; output/status are ignored, timeout uses `HOOK_TIMEOUT_MS`, and failure warns once. See [`examples/hook.mjs`](../examples/hook.mjs), which supports OS notifications and optional `LOOP_POLICE_NTFY_TOPIC` phone pushes.
- `HOOK_LOG`: one payload per JSONL line; relative paths use session `cwd`.

```text
/loop-police set HOOK_CMD=node /path/to/hook.mjs HOOK_LOG=/path/to/loops.jsonl
```
