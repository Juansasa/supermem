# 05 · Lifecycle: measuring, promoting, demoting

This file is the single definition of how records change state. The user: "so rules are defined and actually produce desired effect and actually being used so they can be promote and demote".

## Stats

Each record has script-counted stats, rebuilt from session evidence ([02-capture.md](02-capture.md#session-evidence)) and kept out of git:

| Stat | Meaning |
|---|---|
| matches | sessions where its area or trigger was relevant |
| used | sessions where it was loaded (skill invoked, injected, or pitfall fired) |
| effect | rate of interventions of its targeted kind before vs. after it went on trial |
| cost | tokens it adds × loads ([Impact vs. token cost](#impact-vs-token-cost)) |
| helpful / harmful | counters updated when it's surfaced |

**Linking interventions to records (my call):** each record declares its area and the intervention kind it targets. Scripts count interventions per (area, kind) per session. Whether a specific intervention was caused by a specific change is a judgment by the review agent, recorded in the revert commit's evidence.

## States

`candidate → trial → proven → stale → archived`, plus `pinned` (never changes automatically).

| State | Loaded into sessions? | Judged by |
|---|---|---|
| candidate | no; recorded only | repetition and confidence ([04-review.md](04-review.md#when-something-new-is-added)) |
| trial | yes | interventions of its targeted kind |
| proven | yes | token cost |
| stale | no | reader activity |
| archived | no; kept in the repo | — |

## Transitions

Thresholds are **my call**, tunable.

| Transition | When |
|---|---|
| candidate → trial | repeated and high confidence ([04-review.md](04-review.md#when-something-new-is-added)) |
| trial → proven | after a trial window of N sessions in its area, its targeted interventions are measurably lower than before |
| trial → reverted | **immediately** if the user directly contradicts or corrects it; otherwise when, after the trial window, the targeted interventions continue at the same rate (no effect), or it's marked harmful more than helpful |
| proven → cheaper form | token optimization (tighter wording, lower load level, merge); the optimization itself goes on trial and is reverted if interventions return |
| trial/proven → stale | not loaded or matched in N sessions of its area |
| stale → trial/proven | loaded or matched again (returns to its previous state) |
| stale → archived | still idle after a longer window; longer for records with a history of use |
| project → area scope | the same **know-how** is proven in 2+ projects. Facts tied to a specific target system are never promoted out of their project ([03-knowledge.md](03-knowledge.md#scopes)). |

Only **reader** activity (loaded, matched, surfaced) keeps a record alive; selfim's own edits don't. Archived records stay in the repo; nothing is deleted.

## Impact vs. token cost

The user: "What we need to track is also the resulted impact vs the new cost of token use".

| Side | Measured as |
|---|---|
| **Cost** | tokens it adds × how often it's loaded. `always` records cost every session; `trigger` records per match; a skill body per load. Size is estimated from the rendered text; loads come from session evidence. |
| **Impact** | interventions of its targeted kind before vs. after; failures it warned about that were then avoided; tokens and turns no longer spent on the repeated failure or re-check it replaced. |

**Which measure decides.** The user: "intervention is fine for new findings and settings, token usage is relevant when it's about optimizing for things that have always worked".

| Record is… | Judged by | Goal |
|---|---|---|
| on **trial** (newly added or changed finding/rule/setting) | interventions of its targeted kind | prove it reduces interventions; revert if it doesn't |
| **proven** (impact established, stable) | token cost | keep the effect at the lowest cost: tighten wording, lower load level, merge; revert the optimization if interventions return |

Consequences:

- **Load level follows value.** A record stays `always` only while its impact justifies paying for it every session; otherwise it drops to `trigger` or `search`. A high-impact `trigger` record can be raised.
- **No-impact records are demoted** even if they're used: used but not changing outcomes is cost without value.
- **System trend per project and area:** injected tokens per session, total session tokens, interventions per session. selfim is working when interventions fall without injected tokens creeping up.

## Long-term metrics

Script-computed from evidence unless marked; they feed periodic reviews and the transitions above.

**Outcome (headline)**

| Metric | Why |
|---|---|
| turns between interventions | single headline number; rises as selfim works |
| redundant probes (re-checking facts known to be stable) | measures "keeps checking systems that never change" |
| re-asks (asking about something already decided) · *review-agent judgment* | measures "checking what I've confirmed" |
| retries before success after a failure | shows whether pitfalls shorten the path to a fix |

**Per record (drives promotion and demotion)**

| Metric | Drives |
|---|---|
| warning precision (warned, and the command really was bad) vs. false alarms | tighten or drop noisy triggers; only precise pitfalls may block |
| misses (known pitfall recurred, no warning fired) | widen the trigger |
| ignored rate (warned, agent went ahead, failed anyway) | rewrite the wording, or raise to block |
| churn (edits over time, flip-flopping) | freeze the record, or ask the user |
| projects proven in | project → area promotion |
| fact volatility (re-checks that found the fact changed) | how long a fact is trusted before re-checking |

**Knowledge-base health**

| Metric | Why |
|---|---|
| coverage gaps (interventions and failures per area with no matching record) | where knowledge is missing; costliest areas first |
| library shape (records per state, cap fill, share of the always-loaded budget, stale ratio) | early warning of bloat |
| contradictions (records conflicting with each other or the user's files) · *review-agent judgment* | resolve first; they mislead |

**Review quality**

| Metric | Why |
|---|---|
| revert rate per change type (add, patch, merge, optimize) | be more cautious where selfim is often wrong |
| suggestion acceptance for user-owned files | calibrates what's worth suggesting |
| manual reverts by the user vs. automatic ones | a manual revert is a strong "selfim got it wrong" signal |

**Fair comparison**

- Each session records the **model and Claude Code version**, so a model upgrade isn't credited or blamed on a record.
- Rates are **normalized by activity in the area** (per session in that area), not raw totals.

Thresholds tied to these metrics are **my call**, tunable.

## Whole-solution ledger

The user asked whether selfim tracks "total token use for the whole selfim solution as a whole including hooks". Every cost selfim causes is recorded in one ledger, so its overhead can be set against its savings:

| Bucket | What's counted | Source |
|---|---|---|
| session-start injection | rule file + project index | SessionStart hook (size known when injected) |
| skill listing | `selfim-*` descriptions, present every turn | rendered size × turns |
| skill bodies | `selfim-*` bodies loaded | skill-load evidence |
| pitfall warnings | lines injected by the pre-command check | PreToolUse hook |
| note prompts | lines injected asking for a note | UserPromptSubmit / PostToolUse / Stop hooks |
| note writing | the agent's output for `selfim note` calls | tool hook payloads |
| review runs | tokens and USD per `claude -p` run | its JSON output (`usage`, `total_cost_usd`), logged per run |
| hook latency | ms added per turn (p50/p95), per hook | timing written by each hook |
| **savings** | interventions avoided; tokens and turns no longer spent on repeated failures and re-checks | impact stats above |

- Context repeated on every turn is mostly served from the prompt cache; the ledger counts first-time and cached tokens separately so selfim's cost isn't overstated.
- Reported per project, per area and overall: **total selfim cost vs. estimated savings**, and the trend over time.
- **Guard:** if selfim's overhead exceeds its estimated savings over the last N sessions **(my call, N tunable)**, the next review starts by cutting the most expensive buckets (lower load levels, fewer prompt types, trimmed hooks) before adding anything new.
