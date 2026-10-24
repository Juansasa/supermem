# 08 · Open: calls to check, builder's choices, sources

## Calls made by Claude (please check)

| # | Call | Where |
|---|---|---|
| 1 | Hooks are scripts only; no LLM runs inside them (prompted notes are written by the working agent itself) | [02-capture.md](02-capture.md) |
| 2 | The digest is produced by a script before any LLM reads it | [02-capture.md](02-capture.md#digest-script-zero-tokens) |
| 3 | Counting (matches, use, interventions, cost) is done by scripts; stats are kept out of git | [05-lifecycle.md](05-lifecycle.md#stats) |
| 4 | Secrets redacted before writing; raw transcripts never copied | [02-capture.md](02-capture.md#secrets-my-call) |
| 5 | Optional bounded backfill, default off | [02-capture.md](02-capture.md#backfill-my-call) |
| 6 | Remote-system facts carry their explicit target and never leave their project | [03-knowledge.md](03-knowledge.md#scopes) |
| 7 | Project identity: git remote, fallback root path, `.selfim-project` marker | [03-knowledge.md](03-knowledge.md#project-identity-my-call) |
| 8 | Goals/progress only as a short per-session note | [03-knowledge.md](03-knowledge.md#session-note) |
| 9 | High confidence = 2+ occurrences, observed or user-sourced, never harmful | [04-review.md](04-review.md#when-something-new-is-added) |
| 10 | Lifecycle thresholds (trial window, stale and archive windows) | [05-lifecycle.md](05-lifecycle.md#transitions) |
| 11 | Linking interventions to records by (area, kind); causation judged by the review agent | [05-lifecycle.md](05-lifecycle.md#stats) |
| 12 | User-owned files only ever receive suggestions | [06-outputs.md](06-outputs.md#ownership) |
| 13 | Size caps on the rule file and each skill | [06-outputs.md](06-outputs.md#output-shape) |
| 14 | Rule file and project index injected by a session-start hook, not by editing CLAUDE.md | [06-outputs.md](06-outputs.md#load-levels) |
| 15 | Pitfall check warns by default; proven pitfalls may block | [06-outputs.md](06-outputs.md#pre-command-pitfall-check) |
| 16 | Repo layout | [07-setup.md](07-setup.md#runtime-repo) |
| 17 | Setup defaults: active in this project only; review started by hand | [07-setup.md](07-setup.md#setup-first-invoke) |
| 18 | Scripts use Python 3 standard library only (no dependencies), for portability | — |
| 19 | Hook set and what each records | [02-capture.md](02-capture.md#hooks) |
| 20 | Prompted-note moments and guards (one per turn, session cap, no Stop prompt twice in a row, skip if note exists); prompt types measured and turned off when unproductive | [02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable) |
| 21 | Concision rules and limits for hook entries, notes and digests (pointers not copies, ~300-byte lines, successes as counts, one-line errors, digest budget ~1–2k tokens) | [02-capture.md](02-capture.md#keeping-captured-data-concise) |

## Left to the builder

- Exact threshold values and size budgets.
- Heuristics for detecting interventions in transcripts.
- Commit-message format (trailers or similar), as long as it carries what / why / evidence / expected effect / expected cost.
- Packaging and install mechanics (hooks, linking skills into the skills directory).
- The command/path → area mapping.

## Borrowed from

- **supermem design** (`supermem-docs/`): trigger-matched pitfalls, load levels, helpful/harmful counters, trust by source, last-verified dates, revisit-when on decisions, itemized edits, keeping successful paths, mechanical area inference.
- **UniM0cha/self-improving-skills**: patch-before-create, precomputed similar-record list, reader-only activity, idle-time lifecycle with longer protection for used records, post-run mechanical checks, few broad skills.
