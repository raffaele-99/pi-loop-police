# AGENTS.md

Rules for **pi-loop-police**, a [pi](https://github.com/badlogic/pi-mono) extension that stops thinking and tool-call loops.

## Structure

| Path | Purpose |
|---|---|
| `extensions/index.ts` | Pi entry point |
| `extensions/loop-police.ts` | All wiring and logic |
| `extensions/loop-police.json` | Auto-created persistent config |
| `docs/CONFIG.md` | Keys, messages, and migrations |
| `docs/DETECTORS.md` | Detector behavior and tuning |
| `docs/OBSERVERS.md` | Detection payload and sinks |
| `skills/loop-police-help/SKILL.md` | User reference |
| `skills/loop-police-postmortem/SKILL.md` | Detection analysis |
| `examples/hook.mjs` | `HOOK_CMD` example |
| `package.json` | Pi extension/skill entries |

Pi loads TypeScript directly. Add no dependencies or build step.

## Runtime

| Hook | Work |
|---|---|
| `message_update` | Character and semantic detection on the active thinking/output `contentIndex`; `ctx.abort()` on match |
| `message_end` | Sanitize signed/aborted reasoning; recover; detect stagnation and re-derived reasoning |
| `context` | Exclude marked loop reasoning from model context, not the transcript |
| `tool_call` | In order: repeated tool sequence, file-read ceiling, redundant re-read, search spiral; block on match |
| `agent_start`, `turn_start` | Reset session/per-stream state respectively |

Blocked tools return one in-place recovery result. Stream/reasoning detections use `pi.sendMessage(..., { triggerTurn: true })`.

Every detection goes through `buildDetectionPayload()` to `loop-police:detection` and configured `HOOK_CMD`/`HOOK_LOG` sinks. Its fields and event names are public API; do not rename them.

Keep Pi wiring inside the default export. Keep algorithms, migrations, and string helpers below it as plain functions without Pi imports.

## Config

`loop-police.json` is merged over `DEFAULTS`.

- Put numeric, string, and recovery keys in `NUMERIC_DEFAULTS`, `STRING_DEFAULTS`, and `MESSAGE_DEFAULTS` respectively. Format `{placeholders}` with `fmt()`.
- Every detector must treat its documented key value `0` as disabled.
- New defaults are backfilled automatically.
- For renamed/removed keys, bump `CONFIG_VERSION` and follow `migrateRenamedKeys()`/`migrateRemovedKeys()` so custom values survive and stale defaults update.
- `/loop-police set` changes session config; `/loop-police save` persists it. `MSG_*` is JSON-only.

## Detector checklist

1. Store state in closures and clear it in `reset()`; clear per-turn state on `turn_start`. Never count blocked/aborted work.
2. Add numeric keys, `0` handling, and the top-of-file disable comment.
3. Add a documented `MESSAGE_DEFAULTS` template and pass it through `withSuffix()`.
4. Call `emitDetection(ctx, "<stable_snake_case_event>", details)` on every firing.
5. Warn through `ctx.ui.notify(...)`.
6. Update `docs/DETECTORS.md`, `docs/CONFIG.md`, both skills, and `docs/OBSERVERS.md` for payload changes.
7. Keep transcript-marker wording stable; postmortems grep it.

## Contributions

Open behavior-change issues before PRs at <https://github.com/sebaxzero/pi-loop-police>. Keep each PR to one concern, the single-file style, and comments for non-obvious invariants. Ship user-visible code and docs together. Maintainers alone bump versions, tag, and publish.
