# 05 · Lifecycle: measuring, promoting, demoting

This file is the single definition of how records change state. The user: "so rules are defined and actually produce desired effect and actually being used so they can be promote and demote".

The guiding measure, in the user's words: "no correction is also one of the best metrics, after long usage with very little correction what remains is the token usage". A record proves itself when the issue it targets **stops recurring** and the user no longer has to correct that kind of thing; once proven, what's left to optimize is its token cost.

## Stats

Script-counted from session evidence ([02-capture.md](02-capture.md#session-evidence)), kept only in the derived stats database (never in record files or git):

| Stat | Meaning |
|---|---|
| matches | sessions where the record's **trigger or subject** matched the session's activity (area alone doesn't count) |
| used | sessions where it was loaded (skill invoked, injected, or pitfall fired) |
| recurrences | occurrences of the record's specific issue while it was loaded, vs. the baseline rate before it |
| cost | tokens it adds × loads ([Impact vs. token cost](#impact-vs-token-cost)) |
| helpful | surfaced, and its issue didn't occur in that session |
| harmful | contradicted through a **hard link**: a note cites the record, a call it warned about was denied by the user, or a correction's subject matches the record's subject. Weaker overlaps are recorded as *suspected* and left to the review |

**Measure per record kind** (my call):

| Kind | Its issue, counted as recurrences |
|---|---|
| pitfall, rule | the specific failure or correction it targets |
| fact | redundant probes of its target, and probes that contradict it |
| decision | re-asks about the same question (review-agent judgment) |
| procedure | retries and turns to success on the same task family |

**Sample sizes (my call, tunable):** proving needs K matched sessions where the baseline expected the issue at least twice. Matched sessions count across all enabled projects, not only one project's area. The baseline comes from existing history when the user chose to learn from it.

## States

`candidate → trial → proven → stale → archived`, plus `reverted`, `superseded` and `pinned`.

| State | Loaded into sessions? | Judged by |
|---|---|---|
| candidate | no; recorded only | repetition and confidence ([04-review.md](04-review.md#when-something-new-is-added)) |
| trial | yes | recurrences of its issue (corrections, failures, re-asks, re-probes) |
| proven | yes | token cost; impact re-checked periodically |
| stale | no | reader activity |
| archived | no; kept in the repo | — |
| reverted | no; kept to block re-adding the same change for a while | — |
| superseded | no; links to the decision or record that replaced it | — |
| pinned | as its load level says; never changes automatically | — |

Facts also carry an **unverified** flag (past their last-verified limit, or a watched path changed): they stay loaded, marked "verify before relying on it", and are queued for a cheap re-check. This is separate from the `stale` state, which means "not used".

**Versions:** every record keeps its previous versions. A change that fails is undone by restoring the previous version; the `reverted` state is only for records that were wrong in substance.

## Transitions

Thresholds are **my call**, tunable.

| Transition | When |
|---|---|
| candidate → trial | repeated and high confidence ([04-review.md](04-review.md#when-something-new-is-added)); records from commit messages or repo files also need the user's approval for `always`, global or blocking ([04-review.md](04-review.md#approval-gate)) |
| trial → proven | zero recurrences in K matched sessions where the baseline expected at least two |
| trial → reverted | two or more recurrences while loaded, or harmful (hard link) more than helpful |
| trial → inconclusive | after a maximum of 3 trial windows without a verdict: drops to `trigger` or `search`, or reverts if it never matched |
| any → excluded at once | a hard-link contradiction or a probe that contradicts a fact: excluded from loading from the next session start by a script (no review needed); the next review confirms or restores it |
| proven → cheaper version | token optimization (tighter wording, lower load level, merge); the new version goes on trial; if its issue recurs, the previous version is restored (the record stays proven) |
| proven → re-checked | periodically, its issue's recurrence is re-measured; a proven record whose issue is back returns to trial |
| trial/proven → stale | no trigger or subject match in N sessions where it was loaded or available |
| stale → previous state | matched again |
| stale → archived | still idle after a longer window; longer for records with a history of use |
| decision → superseded | a newer user decision replaces it; a user-stated decision always overrides a revert block |
| project → area scope | the same know-how is proven in 2+ projects ([promotion pass](04-review.md#promotion-pass)); the promoted record starts proven; project copies stop loading but are kept, and come back if the area record is reverted. Target-bound facts never leave their project ([03-knowledge.md](03-knowledge.md#scopes)) |
| area rule → project exception | an area record that is harmful in one project gets a project-scope exception record ("doesn't apply here") instead of a global revert |

**Model changes:** session evidence records model and Claude Code version as context. Trials restart only on a **model-family** change, not on Claude Code updates (which ship too often for any trial to finish).

**Revert blocks expire** after M sessions or on any user-sourced evidence; a record excluded by a script without review confirmation doesn't block re-adding.

Only **reader** activity (matched, surfaced) keeps a record alive; selfim's own edits don't, and for `always` records being injected doesn't count, only a trigger or subject match. Nothing is deleted: undoing is a forward commit ([04-review.md](04-review.md#apply-and-trace)), never `git revert`.

## Impact vs. token cost

The user: "What we need to track is also the resulted impact vs the new cost of token use".

| Side | Measured as |
|---|---|
| **Cost** | tokens it adds × how often it's loaded. `always` records cost every session; `trigger` records per match; a skill body per load. |
| **Impact** | recurrences of its issue avoided; tokens and turns no longer spent on the repeated failure, correction or re-check. |

**Which measure decides.** The user: "intervention is fine for new findings and settings, token usage is relevant when it's about optimizing for things that have always worked".

| Record is… | Judged by | Goal |
|---|---|---|
| on **trial** | recurrences of its issue (no corrections = success) | prove it removes the issue; revert if it doesn't |
| **proven** | token cost | keep the effect at the lowest cost; restore the previous version if the issue returns |

Consequences:
- **Load level follows value.** At every render, `always` slots are ranked by value per character; the lowest drop to `trigger` or `search` when the budget is full. Pinned records count against the budget but aren't demoted (pinned share capped, e.g. 30%).
- **No-impact records are demoted** even if they're used.
- **System trend per project and area:** injected tokens per session, total session tokens, corrections per session.

## Long-term metrics

Script-computed from evidence unless marked; they feed periodic reviews and the transitions above.

**Outcome (headline)**

| Metric | Why |
|---|---|
| corrections per session, and turns between corrections | the headline: falls as selfim works |
| redundant probes (re-checking facts known to be stable) | "keeps checking systems that never change" |
| re-asks (asking about something already decided) · *review-agent judgment* | "checking what I've confirmed" |
| retries before success after a failure | whether pitfalls shorten the path to a fix |

**Per record (drives promotion and demotion)**

| Metric | Drives |
|---|---|
| warning precision, measured only on **ignored warnings and user denials**, plus occasional shadow runs (log only, no warning) | tighten or drop noisy triggers; only precise pitfalls may block |
| misses (known pitfall recurred, no warning fired) | widen the trigger |
| ignored rate (warned, agent went ahead, failed anyway) | rewrite the wording, or raise to block |
| churn (edits over time, flip-flopping) | freeze the record, or ask the user |
| projects proven in | project → area promotion |
| fact volatility (re-checks or probes that found the fact changed) | how long a fact is trusted before re-checking |

**Benchmark** ([11-benchmark.md](11-benchmark.md))

| Metric | Why |
|---|---|
| benchmark results vs. original baseline and earlier runs: checks passed, corrections still needed, repeated failures, turns, tokens | a controlled check on the hardest real sessions that selfim makes the agent better, and that token optimizations didn't make it worse |

**Knowledge-base health**

| Metric | Why |
|---|---|
| coverage gaps (corrections and failures per area with no matching record) | where knowledge is missing; costliest areas first |
| library shape (records per state, cap fill, `always` budget share, stale ratio) | early warning of bloat |
| contradictions (records conflicting with each other or the user's files) · *review-agent judgment* | resolve first; they mislead |

**Review quality**

| Metric | Why |
|---|---|
| revert rate per change type (add, patch, merge, optimize) | be more cautious where selfim is often wrong |
| changes rejected by the apply checks ([04-review.md](04-review.md#checks-on-the-review-agents-output)) | how often the review agent's output is wrong |
| suggestion and approval acceptance | calibrates what's worth suggesting or asking |
| user overrides (`/selfim drop`, manual edits) vs. automatic reverts | a user override is a strong "selfim got it wrong" signal |

**Fair comparison:** model and Claude Code version recorded per session; rates normalized by matched sessions; sessions with missing capture are marked **unmeasured** and excluded ([10-robustness.md](10-robustness.md#capture-health)).

Status shows "warming up (n/K sessions)" and plain counts (learned, on trial, proven, reverted, injected tokens here) until sample minimums are met, instead of ratios.

## Whole-solution ledger

The user asked whether selfim tracks "total token use for the whole selfim solution as a whole including hooks". Every cost selfim causes is recorded in one ledger:

| Bucket | What's counted | Source |
|---|---|---|
| native baseline | CLAUDE.md, rules and auto memory loaded per session (not selfim's cost; for comparison) | InstructionsLoaded hook |
| session-start injection | rule file + project index | SessionStart hook |
| skill listing | `selfim-*` descriptions and the plugin's own `selfim` skill description, every turn | rendered size × turns |
| skill bodies | `selfim-*` bodies and the `selfim` skill body loaded | skill-load evidence |
| in-session commands | the turns spent on `/selfim` and its subcommands | tool hook payloads |
| pitfall warnings | lines injected by the pre-command check | PreToolUse hook |
| note prompts and writing | lines injected asking for a note, and the agent's `selfim note` calls | hooks and tool payloads |
| learning runs | tokens and USD per run | its JSON output, logged per run |
| hook latency | ms added per turn (p50/p95) by synchronous hook parts, including quick exits in disabled projects | timing written by each hook |
| **savings** | tokens and turns no longer spent on repeated failures, corrections and re-checks | impact stats above |

- First-time and cached tokens are counted separately.
- Reported per project, per area and overall, in tokens. Corrections are reported alongside, not converted.
- **Guard:** compares **in-session overhead** against in-session savings; learning-run cost is held by the per-run spending cap instead, and reported separately. The guard applies after a warm-up (at least one proven record), trips when overhead exceeds 1.5× savings over the last N sessions and clears below 1× **(my call)**. While tripped, the next review starts by cutting the most expensive buckets before adding anything.
