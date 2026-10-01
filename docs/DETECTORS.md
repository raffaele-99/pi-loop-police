# Detectors

`pi-loop-police` includes ten detectors, all of which are active by default. Each one looks for a different way an agent can get stuck; their settings are explained in [this file](CONFIG.md).

| Detector | What it looks for | What happens |
|---|---|---|
| Thinking loop | Two adjacent copies of the same ≥80-character thinking block | The stream is stopped, the signed thinking is removed, and the agent is prompted to try again |
| Semantic loop | The same thinking paragraph appears three times | The stream is stopped and the repeated reasoning is removed |
| Output loop | Two adjacent copies of the same ≥100-character response block | The stream is stopped and the repeated response is cut off |
| Output semantic loop | The same response paragraph appears three times | The stream is stopped and the repeated response is cut off |
| Stagnation | The last four thinking turns are all at least 85% similar | The current reasoning is removed and that window is kept out of future context |
| File-read ceiling | One path has already been read 20 times | The next read is blocked |
| Redundant re-read | At least 40% of the last 10 reads revisit unchanged paths | The current read is blocked |
| Search spiral | The same pattern is about to be searched in a third path | The current search is blocked |
| Tool-call loop | An exact sequence of tool calls repeats back-to-back | The repeated call is blocked |
| Re-derived reasoning | Thinking immediately after a detection is at least 85% similar to the failed plan | The repeated reasoning is removed and the agent is prompted to change course |

## While the model is writing

Pi tells the extension which thinking or response block is currently being streamed. That block is checked after every `STRIDE` new characters in two ways:

- The character detector looks for two adjacent, verbatim copies of a block between `THINKING_WINDOW`/`OUTPUT_WINDOW` and `MAX_WINDOW` characters long.
- The semantic detector compares the first `FINGERPRINT_LEN` characters of each paragraph. It normalises leading ordered-list numbers, ignores fenced code, and fires once the same fingerprint reaches `SEMANTIC_THRESHOLD` matches.

When either detector finds a loop, only the current stream is aborted. Thinking is replaced with an unsigned marker, while visible output ends where the repeated section began. The extension then starts a recovery turn; if this happens `CONSECUTIVE_LOOP_LIMIT` times in a row, the warning becomes more direct.

## Across turns and tool calls

The stagnation detector uses Jaccard similarity to compare recent thinking turns. The original reasoning stays in the stored transcript for later inspection, but it won't be sent back to the model as context.

Tool calls are handled slightly differently:

- A sequence is based on the tool name and its arguments. Any repeated cycle length can be caught, while a different call in between breaks the pattern.
- File totals include every line range, but only reads that actually ran. A blocked read doesn't increase another detector's count.
- A re-read counts as redundant until that path is edited. Its recent-read window is cleared whenever the detector fires.
- Search tracking counts the distinct paths used for each pattern.
- A blocked tool receives one recovery result in the same turn; the call itself never runs.
- With `TOOL_LOOP_BAN=1`, a call is blocked only while it repeats back-to-back. `2` bans it for the rest of the session. Tools in `TOOL_LOOP_EXEMPT` are never blocked, although they still break other sequences.

After any kind of detection, the extension also watches the agent's next thinking block. If it simply re-derives the failed plan, that reasoning and its provider signature are removed. The guard stays active, so doing it again produces the stronger `STUCK` message.

## Tuning a detector

If a detector is getting in the way of legitimate work, start with the smallest relevant change:

| What you're seeing | Setting to try |
|---|---|
| Legitimate text is mistaken for a verbatim loop | Raise `THINKING_WINDOW` or `OUTPUT_WINDOW` |
| Structured paragraphs are mistaken for one another | Raise `SEMANTIC_THRESHOLD` or `FINGERPRINT_LEN` |
| A long task genuinely needs many file reads | Raise `FILE_SCAN_LIMIT` or `REREAD_RATIO`; use `REREAD_WINDOW=0` only if needed |
| Searches in a large repository are blocked too early | Raise `SEARCH_EXPAND_LIMIT` |
| A semantic loop runs too long before being stopped | Lower `SEMANTIC_THRESHOLD` to `2` |
