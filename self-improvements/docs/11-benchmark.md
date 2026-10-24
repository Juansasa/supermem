# 11 · Benchmark from real sessions

A growing test set built from the user's own hardest sessions, used to measure whether selfim actually makes the agent better on real work. The user: "having benchmark setup of real tickets of complicated sessions that we build during each review and collection with high collision so it can be test and evaluated to measure", and "these sessions can be extracted in concise steps that we can use for benchmark. We just need to somehow store relevant metrics of these sessions so during review we can extract and save as testing set".

It complements live measurement: recurrence tracking ([05-lifecycle.md](05-lifecycle.md)) shows what happens day to day; the benchmark gives a controlled comparison on the same hard tickets.

## Flow

```
capture: session scorecard (script, per session)        → 02-capture
review:  pick high-collision sessions, extract cases     → 04-review
store:   projects/<id>/benchmark/<case-id>/              → 07-setup
run:     /selfim benchmark (on request, costs tokens)    → this file
report:  vs. original baseline and earlier runs          → 05-lifecycle
```

## Picking sessions (during review)

The session scorecard ([02-capture.md](02-capture.md#session-scorecard)) gives each session a **collision score**: how much went wrong at once (corrections, denials, repeated failures, retry chains, overturned decisions), weighted by turns and tokens spent **(my call, formula tunable)**. Review picks the highest-scoring sessions not yet in the set, **spread across areas** so the set doesn't become all one area, and only sessions whose starting commit is still reachable.

## Case format

Each case is concise and content-free apart from the goal text and commit references **(my call)**:

| Part | From |
|---|---|
| goal (short) and starting commit | scorecard |
| steps as they happened: what was tried, where it went wrong, the correction, what fixed it | digest (command families, outcomes, failure→fix pairs, corrections, decisions) |
| checks derived from the steps, e.g. "doesn't repeat failure X", "follows correction Y", "uses tool order Z", "reaches outcome W" | review, preferring deterministic checks (tool used, tool order, pattern) over model judges |
| baseline numbers: corrections, failures, turns, tokens of the original session | scorecard |

Checks that would pass without any selfim knowledge are flagged and dropped, so a case actually discriminates.

## Running

`/selfim benchmark` (on request only; every run costs tokens):

- **sample** (a few cases, cheap) or **full** suite, showing the estimated cost first;
- each case is replayed in isolation from its starting commit with selfim's current knowledge loaded, and optionally without it (or with an earlier version) for comparison; a few runs per case to see variance;
- the user isn't in the loop during a replay, so "corrections the agent would have needed" are the checks derived from the original corrections.

Claude Code's `claude plugin eval` (isolated runs, repeated, with/without comparison, deterministic graders, scriptable JSON results) is the preferred runner where its case format fits; verified at build time.

## Report

Per case and overall: checks passed, corrections the agent would still have needed, repeated failures, turns and tokens, against the original baseline and against earlier benchmark runs. Results are logged in `reviews/` and shown in `/selfim`; they feed the long-term metrics ([05-lifecycle.md](05-lifecycle.md#long-term-metrics)).

## Offline checks of single records

Separate from the suite: during review, when needed and started by the user, a candidate or a cheaper version of a proven record can be checked against the benchmark cases it relates to ([04-review.md](04-review.md#offline-checks-on-request)). There is no automatic offline gate before trial; the user: "since we have review we do it there when necessary, invoked by user".

## Upkeep

Cases whose starting commit is gone, whose checks no longer discriminate, or whose area selfim no longer covers are retired (kept in git, not run). The set has a size cap per project **(my call)**.
