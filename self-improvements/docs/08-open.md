# 08 · Open: calls to check, builder's choices, sources

## Calls made by Claude (please check)

| # | Call | Where |
|---|---|---|
| 1 | Hooks are scripts only; no LLM runs inside them (prompted notes are written by the working agent itself) | [02-capture.md](02-capture.md) |
| 2 | Each hook split into a synchronous decide/inject part and an async log part | [02-capture.md](02-capture.md#hooks) |
| 3 | Hook payloads first; the transcript is read only for token usage | [02-capture.md](02-capture.md#hooks) |
| 4 | Catch-up sweep at every session start for killed sessions; queue rebuildable | [02-capture.md](02-capture.md#killed-sessions-and-the-catch-up-sweep) |
| 5 | Digest runs automatically after every session, whatever the review setting | [02-capture.md](02-capture.md#digest-script-zero-tokens) |
| 6 | Commit range from session HEADs; no git hooks in project repos | [02-capture.md](02-capture.md#hooks) |
| 7 | Prompted-note moments and guards; decisions also detected in user prompts; prompt types measured and turned off when unproductive | [02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable) |
| 8 | Concision rules and limits (pointers not copies, ~300-byte lines, successes as counts, one-line errors, digest ~1–2k tokens) | [02-capture.md](02-capture.md#keeping-captured-data-concise) |
| 9 | Secrets redacted before writing; raw transcripts never copied | [02-capture.md](02-capture.md#secrets-my-call) |
| 10 | Bootstrap sources (transcripts, git history, repo structure, native memory), cost estimate before running, straight-to-trial for findings that already repeat | [02-capture.md](02-capture.md#bootstrap-on-first-enable) |
| 10a | Native memory and instructions read as "already known"; never re-injected | [02-capture.md](02-capture.md#native-sources-read-only) |
| 11 | Remote-system facts carry their explicit target and never leave their project | [03-knowledge.md](03-knowledge.md#scopes) |
| 12 | Project identity: normalized remote, fallback root path, subprojects declared in `registry.json` | [03-knowledge.md](03-knowledge.md#project-identity-my-call) |
| 13 | Goals/progress only as a short per-session note | [03-knowledge.md](03-knowledge.md#session-note) |
| 14 | High confidence = 2+ occurrences, observed or user-sourced, never harmful; user-stated decisions and confirmed corrections go on trial at once | [04-review.md](04-review.md#when-something-new-is-added) |
| 15 | Review: detached `claude -p`, hooks skip it, user settings/CLAUDE.md excluded, read-only excerpts, JSON changes applied by code | [04-review.md](04-review.md#who-runs-it) |
| 16 | Promotion pass compares know-how (never facts or evidence) across projects | [04-review.md](04-review.md#promotion-pass) |
| 17 | Evidence committed separately before decision commits; a revert is a forward commit to state `reverted` | [04-review.md](04-review.md#apply-and-trace) |
| 18 | Lifecycle thresholds; minimum sample sizes; attribution by trigger match first | [05-lifecycle.md](05-lifecycle.md#stats) |
| 19 | Contradicted records excluded from loading from the next session start, before review | [05-lifecycle.md](05-lifecycle.md#transitions) |
| 20 | Trial window restarts on a model or Claude Code version change | [05-lifecycle.md](05-lifecycle.md#transitions) |
| 21 | Long-term metrics set and their thresholds | [05-lifecycle.md](05-lifecycle.md#long-term-metrics) |
| 22 | Ledger compares tokens only (interventions reported alongside); guard applies after a warm-up | [05-lifecycle.md](05-lifecycle.md#whole-solution-ledger) |
| 23 | User-owned files only ever receive suggestions | [06-outputs.md](06-outputs.md#ownership) |
| 24 | Size caps on rule file, indexes and skills | [06-outputs.md](06-outputs.md#output-shape) |
| 25 | Starting load level: rules/facts/decisions `always` within budget, pitfalls `trigger` | [06-outputs.md](06-outputs.md#load-levels) |
| 26 | Pitfall check warns by default; proven, precise pitfalls may block | [06-outputs.md](06-outputs.md#pre-command-pitfall-check) |
| 27 | Runtime repo default `~/.selfim`, not the plugin data folder | [07-setup.md](07-setup.md#runtime-repo) |
| 28 | Activation tracked in `registry.json`; hooks exit in disabled projects | [07-setup.md](07-setup.md#activation-off-by-default) |
| 29 | Repo layout and placement rules; project-skill link excluded via `.git/info/exclude` | [07-setup.md](07-setup.md#data-placement) |
| 30 | Review started by hand by default | [07-setup.md](07-setup.md#setup-first-selfim) |
| 31 | Second machine: clone at setup, per-machine folder keyed by hardware id, pull/rebase before review | [07-setup.md](07-setup.md#second-machine) |
| 32 | Schema version and `selfim migrate` after plugin updates | [07-setup.md](07-setup.md#upgrades) |
| 33 | Stack: plugin, Python ≥3.9 stdlib, JSONL + markdown/JSON frontmatter in git, derived SQLite, git CLI with trailers, built-in redaction, shingle-Jaccard dedup | [09-stack.md](09-stack.md) |

## Left to the builder

- Exact threshold values, sample sizes and size budgets.
- Heuristics for detecting interventions and decisions in prompts and messages.
- Commit-message format details, as long as the trailers carry what / why / evidence / expected effect / expected cost.
- The command/path → area mapping and command-family normalization for failure→fix pairing.
- The stable machine id source on each OS.

## Borrowed from

- **supermem design** (`supermem-docs/`): trigger-matched pitfalls, load levels, helpful/harmful counters, trust by source, last-verified dates, revisit-when on decisions, itemized edits, keeping successful paths, mechanical area inference.
- **UniM0cha/self-improving-skills**: patch-before-create, precomputed similar-record list, reader-only activity, idle-time lifecycle with longer protection for used records, post-run mechanical checks, few broad skills.
- **Claude Code sessions and memory**: session-id files, resume/fork handling, worktree sharing, index plus on-demand files ([07-setup.md](07-setup.md#borrowed-from-claude-codes-own-sessions)).
- **Others** (ACE, Letta, Hermes, claude-reflect, claude-mem): see [09-stack.md](09-stack.md#sources).
