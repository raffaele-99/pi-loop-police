---
name: loop-police-postmortem
description: Analyze current-session pi-loop-police detections, classify them, and recommend evidence-based tuning. Use when asked why it fired, whether it was wrong, or how to tune it.
license: MIT
---

# Loop Police Post-Mortem

Analyze every current-session detection from transcript evidence. Do not guess or recommend unrelated changes.

## Find evidence

| Detector | Trace |
|---|---|
| Thinking loop | `[THINKING LOOP — truncated by loop-police]`; `⚠️ THINKING LOOP DETECTED` |
| Semantic loop | `[SEMANTIC LOOP — truncated by loop-police]`; `⚠️ SEMANTIC LOOP DETECTED` |
| Output loop | `[OUTPUT LOOP — truncated by loop-police]`; `⚠️ OUTPUT LOOP DETECTED` |
| Output semantic | `[SEMANTIC OUTPUT LOOP — truncated by loop-police]`; `⚠️ OUTPUT SEMANTIC LOOP DETECTED` |
| Consecutive escalation | `⚠️ CONSECUTIVE LOOP ({count}x)` |
| Stagnation | `⚠️ REASONING STAGNATION` |
| File-read ceiling | `loop-police: file read {count}x total — {path}`; `⚠️ FILE READ CEILING` |
| Redundant re-read | Blocked result `⚠️ REDUNDANT RE-READ` with `{count}`/`{window}` |
| Search spiral | `loop-police: search spiral "{pattern}"`; `⚠️ SEARCH EXPANSION SPIRAL` |
| Tool-call loop | Blocked result `⚠️ TOOL CALL LOOP` with `{windowSize}` |
| Re-derived reasoning | `[REDERIVED REASONING — trimmed by loop-police: …]`; `⚠️ REDERIVED REASONING` or `⚠️ STUCK ({count}x)` |

Customized `MSG_*` may change warnings; fixed markers/block prefixes take priority. Before 1.12.0, file loops may instead show `loop-police: file read {count}x — {path}` and `⚠️ FILE READ LOOP`.

Find `extensions/loop-police.json` under the first applicable install root:

- Global `~/.pi/agent/` or local `./.pi/agent/`
- Then `npm/node_modules/pi-loop-police/`, `git/github.com/sebaxzero/pi-loop-police/`, or `extensions/pi-loop-police/`

Defaults:

```text
THINKING_WINDOW=80 OUTPUT_WINDOW=100 MAX_WINDOW=4000 STRIDE=50
PARA_MIN_LEN=40 FINGERPRINT_LEN=60 SEMANTIC_THRESHOLD=3
STAGNATION_WINDOW=4 STAGNATION_THRESHOLD=0.85 FILE_SCAN_LIMIT=20
REREAD_WINDOW=10 REREAD_RATIO=0.4 SEARCH_EXPAND_LIMIT=3
CONSECUTIVE_LOOP_LIMIT=2 TOOL_LOOP_BAN=1 REDERIVE_THRESHOLD=0.85
```

Session `/loop-police set` values override JSON. Old stream-key names migrate automatically. `0` disables the detector for `THINKING_WINDOW`, `OUTPUT_WINDOW`, `SEMANTIC_THRESHOLD`, `STAGNATION_WINDOW`, `FILE_SCAN_LIMIT`, `REREAD_WINDOW`, `SEARCH_EXPAND_LIMIT`, `TOOL_LOOP_BAN`, and `REDERIVE_THRESHOLD`; it disables escalation for `CONSECUTIVE_LOOP_LIMIT`. `REREAD_RATIO=0` is most aggressive, not off. Skip disabled detectors.

Only executed reads count toward `FILE_SCAN_LIMIT`; state resets on agent start or `/loop-police reset`. A re-read is redundant until that path is edited.

If no detection exists, say so and stop.

## Analyze

For each firing, chronologically identify:

1. The task and immediately preceding reasoning/tool calls.
2. The repeated text, plan, path, pattern, or call. Infer removed signed reasoning only from nearby evidence; stagnant reasoning remains in the transcript but is omitted from later model context.
3. Whether recovery caused a pivot, a repeat on the same target, or a workaround through another tool.

Assign one verdict:

| Verdict | Meaning |
|---|---|
| Justified | Real loop; no config change |
| False positive | Legitimate work was blocked too early |
| Justified but ineffective | Real loop repeated or escalated; change recovery, not sensitivity |

## Tune

| Detector/case | False-positive evidence | Next change |
|---|---|---|
| File ceiling | Huge file paged usefully; edited hot file revisited | Set `FILE_SCAN_LIMIT` to `30`–`40`; targeted grep; `/loop-police reset` for long edit sessions |
| Redundant re-read | Unchanged huge/reference file reread usefully | Set `REREAD_RATIO` to `0.5`–`0.6`; targeted grep; disable only on explicit request |
| Search spiral | Systematic multi-package search whose results were used | `SEARCH_EXPAND_LIMIT=5` |
| Tool loop | Intentional polling or rerun | Interleave another call or reset; `TOOL_LOOP_BAN=0` only on explicit request |
| Tool loop, ineffective | Blocked call keeps returning | `TOOL_LOOP_BAN=2` |
| Semantic loop | Structured items share prefixes | Set `SEMANTIC_THRESHOLD` to `4`–`5`, `FINGERPRINT_LEN=100`, or raise `PARA_MIN_LEN`; counters are normalized and fenced code is skipped |
| Thinking character loop | Legitimate repeated quote/boilerplate | Set `THINKING_WINDOW` to `120`–`160` |
| Output loop | Requested/generated adjacent repetition | Set `OUTPUT_WINDOW` to `200`–`400`; disable only on explicit request |
| Stagnation | Similar batch work still progressed | Set `STAGNATION_THRESHOLD` to `0.90`–`0.95` or `STAGNATION_WINDOW=6` |
| Re-derived reasoning | Similar text was a pivot/report, not a retry | Set `REDERIVE_THRESHOLD` to `0.90`–`0.95`; disable only on explicit request |
| Re-derived, ineffective | `STUCK` keeps escalating | Rewrite `MSG_STUCK`; suggest user intervention or a stronger model |
| Stream loop, ineffective | Same loop repeats/escalates | Shorten the matching `MSG_*`, name an alternative action, or lower `CONSECUTIVE_LOOP_LIMIT` |
| Late detection | Large prefix was already wasted | Lower stream window or `SEMANTIC_THRESHOLD` |

Only tune detectors that fired. Move one notch, preserve `{placeholders}`, and treat one ambiguous incident as “watch; change if repeated.” Repetition strengthens a recommendation.

## Report

Provide:

1. One-paragraph count, estimated blocked/truncated cost, and config verdict.
2. One short evidence-backed block per incident: detector, event, outcome, verdict.
3. Only warranted changes: one `/loop-police set KEY=VAL ...` line for numeric session changes and a minimal JSON snippet for persistence or `MSG_*` edits.
4. An offer to edit the located config; do not edit without confirmation.
