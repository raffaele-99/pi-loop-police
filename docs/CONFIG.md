# Configuration

When it first loads, `pi-loop-police` creates `extensions/loop-police.json` beside the extension. Any settings added in a later version are filled in automatically without replacing your existing choices.

You can use `/loop-police set KEY=VAL [...]` to change the current session, then `/loop-police save` if you'd like to keep those changes. Recovery messages are the exception: `MSG_*` values need to be edited directly in the JSON file.

## Settings

These are the available settings and their defaults:

| Key | Default | What it changes |
|---|---:|---|
| `THINKING_WINDOW` | `80` | Smallest verbatim repeat checked in thinking |
| `OUTPUT_WINDOW` | `100` | Smallest verbatim repeat checked in the response |
| `MAX_WINDOW` | `4000` | Largest verbatim repeat checked in either stream |
| `STRIDE` | `50` | New characters generated between stream checks |
| `PARA_MIN_LEN` | `40` | Shortest paragraph included in semantic checks |
| `FINGERPRINT_LEN` | `60` | Characters used to identify a paragraph |
| `SEMANTIC_THRESHOLD` | `3` | Matching paragraphs needed to identify a loop |
| `STAGNATION_WINDOW` | `4` | Recent thinking turns compared for stagnation |
| `STAGNATION_THRESHOLD` | `0.85` | Jaccard similarity needed to identify stagnation |
| `FILE_SCAN_LIMIT` | `20` | Completed reads allowed for one path |
| `SEARCH_EXPAND_LIMIT` | `3` | Distinct-path attempt that gets blocked for one pattern |
| `REREAD_WINDOW` | `10` | Recent reads checked for unchanged files |
| `REREAD_RATIO` | `0.4` | Share of that window that needs to be redundant |
| `CONSECUTIVE_LOOP_LIMIT` | `2` | Consecutive stream loops before the warning escalates |
| `TOOL_LOOP_BAN` | `1` | `0` turns it off; `1` blocks adjacent repeats; `2` bans the call for the session |
| `TOOL_LOOP_EXEMPT` | `""` | Comma-separated tool names that should not be blocked |
| `REDERIVE_THRESHOLD` | `0.85` | Similarity needed to identify a re-derived plan |
| `HOOK_CMD` | `""` | Command run after a detection |
| `HOOK_TIMEOUT_MS` | `5000` | Time allowed for `HOOK_CMD` before it is stopped |
| `HOOK_LOG` | `""` | JSONL file that receives each detection |

You can turn a detector off by setting its main value to `0`. This works for `THINKING_WINDOW`, `OUTPUT_WINDOW`, `SEMANTIC_THRESHOLD`, `STAGNATION_WINDOW`, `FILE_SCAN_LIMIT`, `SEARCH_EXPAND_LIMIT`, `REREAD_WINDOW`, `TOOL_LOOP_BAN`, and `REDERIVE_THRESHOLD`.

> [!NOTE] `CONSECUTIVE_LOOP_LIMIT=0` only turns off the stronger follow-up warning. `REREAD_RATIO=0` is the most aggressive setting, not an off switch.

## Recovery messages

Whenever a detector fires, the extension gives the agent a short message intended to help it change course. You can rewrite these messages in `loop-police.json`; any known `{placeholders}` will be filled in at runtime, while unknown ones are left visible.

| Key | Available placeholders |
|---|---|
| `MSG_THINKING_LOOP` | None |
| `MSG_SEMANTIC_LOOP` | None |
| `MSG_OUTPUT_LOOP` | None |
| `MSG_OUTPUT_SEMANTIC_LOOP` | None |
| `MSG_CONSECUTIVE_LOOP` | `{count}` |
| `MSG_STAGNATION` | `{window}`, `{threshold}` |
| `MSG_FILE_SCAN_LOOP` | `{path}`, `{count}` |
| `MSG_SEARCH_SPIRAL` | `{pattern}`, `{paths}` |
| `MSG_REREAD` | `{path}`, `{count}`, `{window}` |
| `MSG_TOOL_LOOP` | `{windowSize}` |
| `MSG_REDERIVED` | None |
| `MSG_STUCK` | `{count}` |
| `MSG_SUFFIX` | None; added to every recovery message |

If you'd like to send detection data somewhere outside the session, the available options are described in [this file](OBSERVERS.md).

## Updating older configuration files

Configuration migrations happen automatically when the extension loads:

- Before `1.12.0`, `FILE_READ_LIMIT` and `MSG_FILE_READ_LOOP` were used by a detector that no longer exists. They're removed, and an untouched `FILE_SCAN_LIMIT` becomes `20`.
- Before `1.8.0`, the stream settings had different names. They're renamed, and an untouched `MAX_WINDOW` becomes `4000`.
- Before `1.5.0`, `TOOL_LOOP_BAN` used `0` and `1` for its two modes. Those become `1` and `2`, leaving the new `0` value free to turn the detector off.
