# Detectors

All ten detectors are enabled by default. See [Configuration](CONFIG.md) for keys and defaults.

| Detector | Default trigger | Result |
|---|---|---|
| Thinking loop | Adjacent repeated thinking block, ≥80 chars | Abort; sanitize signed reasoning; recover |
| Semantic loop | Thinking paragraph fingerprint seen 3 times | Same |
| Output loop | Adjacent repeated output block, ≥100 chars | Abort; truncate; recover |
| Output semantic loop | Output paragraph fingerprint seen 3 times | Same |
| Stagnation | Last 4 turns' thinking all ≥85% similar | Sanitize current and omit window from future context |
| File-read ceiling | Same path read 20 times | Block |
| Redundant re-read | ≥40% of last 10 reads revisit unchanged paths | Block |
| Search spiral | Same pattern reaches a third path | Block |
| Tool-call loop | Exact tool-call sequence repeats adjacently | Block |
| Re-derived reasoning | Next thinking after a detection is ≥85% similar | Sanitize and recover |

## Streaming

Pi's active thinking/output `contentIndex` is checked every `STRIDE` characters.

- Character detection compares adjacent verbatim blocks of `THINKING_WINDOW`/`OUTPUT_WINDOW` through `MAX_WINDOW` characters.
- Semantic detection compares each paragraph's first `FINGERPRINT_LEN` characters, normalizes leading ordered-list counters, ignores fenced code, and fires at `SEMANTIC_THRESHOLD` matches.

A match aborts immediately. Thinking becomes an unsigned marker; output ends at the match. Recovery starts a new turn and escalates after `CONSECUTIVE_LOOP_LIMIT` consecutive stream loops.

## Cross-turn and tool detection

Stagnation compares recent thinking with Jaccard similarity. Its reasoning remains in the transcript but is omitted from later model context.

Tool rules:

- Sequence matching hashes tool name and arguments. Any adjacent cycle length fires; an intervening call breaks it.
- File totals span line ranges and count only executed reads.
- A re-read is redundant until its path is edited. Its window resets after firing.
- Search tracking counts distinct paths per pattern.
- Blocked tools return one recovery result in the same turn and never update another detector.
- `TOOL_LOOP_BAN=1` blocks adjacent repeats; `2` bans the call for the session. `TOOL_LOOP_EXEMPT` names are never blocked but remain in history.

After any detection, the re-derived-reasoning guard compares the next thinking block with the failed plan. It removes a match and its provider signature, then stays armed so another match escalates to `STUCK`.

## Tuning

| Problem | Adjustment |
|---|---|
| Verbatim false positive | Raise `THINKING_WINDOW`/`OUTPUT_WINDOW` |
| Similar structured paragraphs | Raise `SEMANTIC_THRESHOLD`/`FINGERPRINT_LEN` |
| Heavy file use | Raise `FILE_SCAN_LIMIT`/`REREAD_RATIO`; set `REREAD_WINDOW=0` if needed |
| Large repository searches | Raise `SEARCH_EXPAND_LIMIT` |
| Late semantic detection | Lower `SEMANTIC_THRESHOLD` to `2` |
