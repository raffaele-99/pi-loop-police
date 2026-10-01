---
name: loop-police-help
description: Explain pi-loop-police detections, commands, configuration, persistence, recovery text, or observer hooks.
license: MIT
---

# Loop Police Help

pi-loop-police interrupts thinking, output, and tool-call loops before they exhaust context.

## Detectors

| Detector | Trigger/result |
|---|---|
| Thinking/output loop | Adjacent verbatim stream block; abort and trim |
| Thinking/output semantic loop | Same paragraph fingerprint repeats; abort and trim. Ordered-list counters are normalized; fenced code is ignored |
| Stagnation | Similar thinking across turns; sanitize current block and omit the window from future context |
| File-read ceiling | Same path read too often across all ranges; block. Only executed reads count |
| Redundant re-read | Recent reads revisit unchanged paths too often; block |
| Search spiral | Same pattern reaches too many paths; block |
| Tool-call loop | Exact call sequence repeats adjacently; block |
| Re-derived reasoning | Post-detection thinking repeats the failed plan; remove the full signed block |

`CONSECUTIVE_LOOP_LIMIT` escalates repeated stream aborts.

## Commands

| Command | Effect |
|---|---|
| `/loop-police` | Show state/config |
| `/loop-police reset` | Clear state |
| `/loop-police set KEY=VAL [KEY=VAL …]` | Change session config |
| `/loop-police save` | Persist current config |

## Config

| Key | Default | Meaning |
|---|---:|---|
| `THINKING_WINDOW` | `80` | Minimum verbatim thinking repeat |
| `OUTPUT_WINDOW` | `100` | Minimum verbatim output repeat |
| `MAX_WINDOW` | `4000` | Maximum repeat checked |
| `STRIDE` | `50` | Stream characters between checks |
| `PARA_MIN_LEN` | `40` | Minimum fingerprinted paragraph |
| `FINGERPRINT_LEN` | `60` | Fingerprint characters |
| `SEMANTIC_THRESHOLD` | `3` | Fingerprint repetitions required |
| `STAGNATION_WINDOW` | `4` | Compared turns |
| `STAGNATION_THRESHOLD` | `0.85` | Jaccard similarity required |
| `FILE_SCAN_LIMIT` | `20` | Executed reads per path |
| `SEARCH_EXPAND_LIMIT` | `3` | Distinct-path attempt that blocks a pattern |
| `REREAD_WINDOW` | `10` | Executed reads checked |
| `REREAD_RATIO` | `0.4` | Redundant share required; window resets after firing |
| `CONSECUTIVE_LOOP_LIMIT` | `2` | Stream-loop escalation count |
| `TOOL_LOOP_BAN` | `1` | `0` off; `1` adjacent block; `2` session ban |
| `TOOL_LOOP_EXEMPT` | `""` | Case-insensitive exempt tools, comma-separated; still recorded in history |
| `REDERIVE_THRESHOLD` | `0.85` | Failed-plan similarity required |
| `HOOK_CMD` | `""` | Observer command |
| `HOOK_TIMEOUT_MS` | `5000` | Observer timeout |
| `HOOK_LOG` | `""` | Observer JSONL path |

`0` disables the detector for `THINKING_WINDOW`, `OUTPUT_WINDOW`, `SEMANTIC_THRESHOLD` (both streams), `STAGNATION_WINDOW`, `FILE_SCAN_LIMIT`, `SEARCH_EXPAND_LIMIT`, `REREAD_WINDOW`, `TOOL_LOOP_BAN`, and `REDERIVE_THRESHOLD`; it disables escalation for `CONSECUTIVE_LOOP_LIMIT`. `REREAD_RATIO=0` is most aggressive, not off.

The auto-created config is `extensions/loop-police.json` below the install root. Global roots use `~/.pi/agent/`; local roots use `./.pi/agent/`. Install suffixes are:

- `npm/node_modules/pi-loop-police/`
- `git/github.com/sebaxzero/pi-loop-police/`
- `extensions/pi-loop-police/`

Use `/loop-police set FILE_SCAN_LIMIT=30` then `/loop-police save`, or edit JSON. Missing keys use defaults. Pre-1.8.0 stream-key renames and pre-1.12.0 removal of `FILE_READ_LIMIT`/`MSG_FILE_READ_LOOP` migrate automatically.

## Recovery text

Edit these only in JSON. Known `{placeholders}` are replaced at runtime.

| Key | Placeholders |
|---|---|
| `MSG_THINKING_LOOP` | — |
| `MSG_SEMANTIC_LOOP` | — |
| `MSG_OUTPUT_LOOP` | — |
| `MSG_OUTPUT_SEMANTIC_LOOP` | — |
| `MSG_CONSECUTIVE_LOOP` | `{count}` |
| `MSG_STAGNATION` | `{window}`, `{threshold}` |
| `MSG_FILE_SCAN_LOOP` | `{path}`, `{count}` |
| `MSG_REREAD` | `{path}`, `{count}`, `{window}` |
| `MSG_SEARCH_SPIRAL` | `{pattern}`, `{paths}` |
| `MSG_TOOL_LOOP` | `{windowSize}` |
| `MSG_REDERIVED` | — |
| `MSG_STUCK` | `{count}` |
| `MSG_SUFFIX` | —; appended to every message |

## Observers

Every detection sends metadata (`event`, `timestamp`, `model`, `sessionId`, `sessionFile`, `cwd`, `turnIndex`, `consecutiveLoops`, `details`) to:

- `loop-police:detection` on Pi's event bus.
- `HOOK_CMD`, with JSON as the last argument. It is shell-free, whitespace-split, timed out, and cannot use paths containing spaces.
- `HOOK_LOG`, one JSONL record per event; relative paths use session `cwd`.

Events: `thinking_loop`, `semantic_loop`, `output_loop`, `output_semantic_loop`, `stagnation`, `file_scan_loop`, `redundant_reread`, `search_spiral`, `tool_loop`, `rederived_reasoning`. The README defines `details`.
