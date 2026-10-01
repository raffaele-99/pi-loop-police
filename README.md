# pi-loop-police

[![npm](https://img.shields.io/npm/v/pi-loop-police)](https://www.npmjs.com/package/pi-loop-police)

A dependency-free [pi](https://pi.dev) extension that detects, interrupts, removes, and recovers from thinking, output, and tool-call loops in real time.

## Install

```bash
pi install npm:pi-loop-police
# or
pi install git:github.com/sebaxzero/pi-loop-police.git
```

Add `-l` for a project-local install. No build or setup is required.

## Detectors

| Detector | Default trigger | Result |
|---|---|---|
| Thinking loop | Adjacent repeated thinking block, ≥80 chars | Abort; sanitize signed reasoning; recover |
| Semantic loop | Thinking paragraph fingerprint seen 3 times | Same |
| Output loop | Adjacent repeated output block, ≥100 chars | Abort; truncate; recover |
| Output semantic loop | Output paragraph fingerprint seen 3 times | Same |
| Stagnation | Last 4 turns' thinking all ≥85% similar | Sanitize current and omit window from future context |
| File-read ceiling | Same path read 20 times | Block |
| Redundant re-read | ≥40% of last 10 reads revisit unchanged paths | Block |
| Search spiral | Same pattern searched in 3 paths | Block |
| Tool-call loop | Exact tool-call sequence repeats adjacently | Block |
| Re-derived reasoning | Next thinking after a detection is ≥85% similar | Sanitize and recover |

Streaming detection follows Pi's active `contentIndex` and runs every `STRIDE` characters. Character detection compares adjacent verbatim blocks of `THINKING_WINDOW`/`OUTPUT_WINDOW` through `MAX_WINDOW` characters. Semantic detection compares each paragraph's first `FINGERPRINT_LEN` characters, ignores leading ordered-list counters and fenced code, and fires at `SEMANTIC_THRESHOLD` matches.

Stream matches abort immediately. Thinking becomes an unsigned marker; output ends at the match. Recovery starts a new turn and escalates after `CONSECUTIVE_LOOP_LIMIT` consecutive stream loops.

Tool rules:

- Sequence matching hashes tool name and arguments. Any adjacent cycle length fires; an intervening call breaks it.
- File totals span line ranges and count only executed reads.
- A re-read is redundant only when its path was read and not edited since. Its window resets after firing.
- Search tracking counts distinct paths per pattern.
- Blocked tools return one recovery result in the same turn and never update another detector.
- `TOOL_LOOP_BAN=1` blocks adjacent repeats; `2` bans the call for the session. `TOOL_LOOP_EXEMPT` names are never blocked but remain in history.

After any detection, the re-derived-reasoning guard compares the next thinking block with the failed plan, removes a match including its provider signature, and stays armed so repeated matches escalate to `STUCK`.

## Commands

| Command | Effect |
|---|---|
| `/loop-police` | Show state and config |
| `/loop-police reset` | Clear session state |
| `/loop-police set KEY=VAL [KEY=VAL …]` | Change session config |
| `/loop-police save` | Save current config |

String values extend to the next `KEY=` token, so `HOOK_CMD=node /path/hook.mjs` works. Invalid numeric values are rejected or defaulted.

## Config

`extensions/loop-police.json` is auto-created beside the extension and backfilled with new defaults.

| Key | Default | Meaning |
|---|---:|---|
| `THINKING_WINDOW` | `80` | Minimum verbatim thinking repeat |
| `OUTPUT_WINDOW` | `100` | Minimum verbatim output repeat |
| `MAX_WINDOW` | `4000` | Maximum verbatim repeat checked |
| `STRIDE` | `50` | New stream characters between checks |
| `PARA_MIN_LEN` | `40` | Minimum fingerprinted paragraph |
| `FINGERPRINT_LEN` | `60` | Fingerprint characters |
| `SEMANTIC_THRESHOLD` | `3` | Repeated fingerprints required |
| `STAGNATION_WINDOW` | `4` | Compared thinking turns |
| `STAGNATION_THRESHOLD` | `0.85` | Jaccard similarity required |
| `FILE_SCAN_LIMIT` | `20` | Executed reads allowed per path |
| `SEARCH_EXPAND_LIMIT` | `3` | Distinct-path attempt that blocks a pattern |
| `REREAD_WINDOW` | `10` | Recent executed reads checked |
| `REREAD_RATIO` | `0.4` | Redundant share required |
| `CONSECUTIVE_LOOP_LIMIT` | `2` | Stream-loop escalation count |
| `TOOL_LOOP_BAN` | `1` | `0` off; `1` adjacent; `2` session ban |
| `TOOL_LOOP_EXEMPT` | `""` | Case-insensitive exempt tool names, comma-separated |
| `REDERIVE_THRESHOLD` | `0.85` | Failed-plan similarity required |
| `HOOK_CMD` | `""` | Observer command |
| `HOOK_TIMEOUT_MS` | `5000` | Observer timeout |
| `HOOK_LOG` | `""` | Observer JSONL path |

Set these to `0` to disable their detector: `THINKING_WINDOW`, `OUTPUT_WINDOW`, `SEMANTIC_THRESHOLD` (both semantic streams), `STAGNATION_WINDOW`, `FILE_SCAN_LIMIT`, `SEARCH_EXPAND_LIMIT`, `REREAD_WINDOW`, `TOOL_LOOP_BAN`, and `REDERIVE_THRESHOLD`. `CONSECUTIVE_LOOP_LIMIT=0` disables escalation. `REREAD_RATIO=0` is most aggressive, not off.

| Problem | Adjustment |
|---|---|
| Verbatim false positive | Raise `THINKING_WINDOW`/`OUTPUT_WINDOW` |
| Similar structured paragraphs | Raise `SEMANTIC_THRESHOLD`/`FINGERPRINT_LEN` |
| Heavy file use | Raise `FILE_SCAN_LIMIT`/`REREAD_RATIO`; set `REREAD_WINDOW=0` if needed |
| Large repository searches | Raise `SEARCH_EXPAND_LIMIT` |
| Late semantic detection | Lower `SEMANTIC_THRESHOLD` to `2` |

### Recovery text

Edit `MSG_*` only in JSON. Known `{placeholders}` are replaced; unknown ones remain visible.

| Key | Placeholders |
|---|---|
| `MSG_THINKING_LOOP` | — |
| `MSG_SEMANTIC_LOOP` | — |
| `MSG_OUTPUT_LOOP` | — |
| `MSG_OUTPUT_SEMANTIC_LOOP` | — |
| `MSG_CONSECUTIVE_LOOP` | `{count}` |
| `MSG_STAGNATION` | `{window}`, `{threshold}` |
| `MSG_FILE_SCAN_LOOP` | `{path}`, `{count}` |
| `MSG_SEARCH_SPIRAL` | `{pattern}`, `{paths}` |
| `MSG_REREAD` | `{path}`, `{count}`, `{window}` |
| `MSG_TOOL_LOOP` | `{windowSize}` |
| `MSG_REDERIVED` | — |
| `MSG_STUCK` | `{count}` |
| `MSG_SUFFIX` | —; appended to every message |

## Detection observers

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

`model` may be `null`. Events and details:

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

- `loop-police:detection`: always-on extension event. Subscribe with `pi.events.on("loop-police:detection", handler)`. See [`examples/listener-extension.ts`](examples/listener-extension.ts) and [pi-input-bar](https://github.com/sebaxzero/pi-input-bar).
- `HOOK_CMD`: direct, shell-free, fire-and-forget command receiving payload JSON as its last argument. It uses whitespace argument splitting, so paths cannot contain spaces; output/status are ignored, timeout uses `HOOK_TIMEOUT_MS`, and failure warns once. See [`examples/hook.mjs`](examples/hook.mjs), which supports OS notifications and optional `LOOP_POLICE_NTFY_TOPIC` phone pushes.
- `HOOK_LOG`: one payload per JSONL line; relative paths use session `cwd`.

Example:

```text
/loop-police set HOOK_CMD=node /path/to/hook.mjs HOOK_LOG=/path/to/loops.jsonl
```

## Skills

- `loop-police-help`: commands, config, and install paths.
- `loop-police-postmortem`: evidence-based detection classification and tuning.

## Upgrades

Migrations preserve custom values and apply new defaults:

- `<1.12.0`: remove `FILE_READ_LIMIT`/`MSG_FILE_READ_LOOP`; untouched `FILE_SCAN_LIMIT` becomes `20`.
- `<1.8.0`: rename `MIN_THINKING_WINDOW`, `MIN_OUTPUT_WINDOW`, `MAX_THINKING_WINDOW`, `CHECK_STRIDE`, `PARA_FINGERPRINT_LEN`, `PARA_LOOP_THRESHOLD`; untouched `MAX_WINDOW` becomes `4000`.
- `<1.5.0`: shift `TOOL_LOOP_BAN` from old `0/1` to new `1/2`; new `0` means off.

## Compatibility

Works with Pi-normalized reasoning, including OpenAI-compatible Qwen and DeepSeek models. [pi-canary](https://github.com/sebaxzero/pi-canary) yields on aborted turns.

## License

MIT
