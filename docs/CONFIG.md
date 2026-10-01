# Configuration

`extensions/loop-police.json` is auto-created beside the extension and backfilled with new defaults. `/loop-police set KEY=VAL [...]` changes the current session; `/loop-police save` persists it. `MSG_*` keys are JSON-only.

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

`0` disables the detector for `THINKING_WINDOW`, `OUTPUT_WINDOW`, `SEMANTIC_THRESHOLD` (both streams), `STAGNATION_WINDOW`, `FILE_SCAN_LIMIT`, `SEARCH_EXPAND_LIMIT`, `REREAD_WINDOW`, `TOOL_LOOP_BAN`, and `REDERIVE_THRESHOLD`. It disables escalation for `CONSECUTIVE_LOOP_LIMIT`; `REREAD_RATIO=0` is most aggressive, not off.

## Recovery messages

Known `{placeholders}` are replaced; unknown ones remain visible.

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

See [Observers](OBSERVERS.md) for `HOOK_CMD`, `HOOK_LOG`, and the extension event API.

## Migration

Migration is automatic:

- `<1.12.0`: removes `FILE_READ_LIMIT`/`MSG_FILE_READ_LOOP`; untouched `FILE_SCAN_LIMIT` becomes `20`.
- `<1.8.0`: renames the stream keys; untouched `MAX_WINDOW` becomes `4000`.
- `<1.5.0`: shifts `TOOL_LOOP_BAN` from old `0/1` to new `1/2`; new `0` means off.
