# Watch advice — RESOLVED ledger

<!-- drained-through: (none) -->
<!-- armed-at: 6f08e303754f9b47de929b79f373c41a0cc2a1f5 -->

The drain ledger for the TRACKED watch advice (`advice.jsonl` / `advice.md`, appended by
`codex_watch.py` on every reviewed commit): one row per NEEDS_REVISION / ERROR entry that
has been handled, keyed `ts · ref`. `python3 .claude/scripts/watch_drain.py status` counts
what is owed; `list --unresolved` prints the entries that still need a row; `rotate
--through <ts>` archives what is rowed and advances the marker above. The `armed-at`
marker is the review's starting line: commits at or before it are never selected.

| status | meaning |
|---|---|
| `FIXED §<id>` | cured in that commit |
| `REFUTED` | the finding is wrong; the row says why |
| `RETRY — <when>` | an ERROR entry re-reviewed later |

| ts | kind | ref | model | verdict | status | evidence |
|---|---|---|---|---|---|---|
