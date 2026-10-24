# 04 · Review: turning evidence into knowledge

## Who runs it

A separate agent (not the working session) processes queued digests ([02-capture.md](02-capture.md#digest-script-zero-tokens)) together with selfim's existing records. It decides what to add, patch, merge, tighten, promote, demote or revert. Mechanical counts come from scripts ([05-lifecycle.md](05-lifecycle.md#stats)); the agent spends tokens only on judgment.

How a run works ([09-stack.md](09-stack.md#review-agent-claude--p)):

1. `/selfim learn` (or automatically after a session, per settings, or when the user accepts learning from history) starts a **detached** `claude -p` run with the runtime repo as working directory, so it isn't bound by the calling session's tool timeout.
2. The run sets `SELFIM_REVIEW=1`, so selfim's own hooks don't capture it, and excludes the user's settings, hooks and CLAUDE.md, so the user's interactive rules don't apply to it.
3. Input, prepared by scripts: the queued digests, the affected records, a precomputed list of similar existing records, current stats, **read-only excerpts** of the user's own files where a suggestion is being considered, and a summary of what native memory and instructions already cover ([02-capture.md](02-capture.md#native-sources-read-only)), so nothing already known is added again. The run never reads outside the runtime repo.
4. Output: a JSON list of changes. selfim's code applies them, runs the checks below, and commits.
5. It pulls and rebases first if the repo has a remote, and pushes after.

Each run logs its own tokens and USD to the [whole-solution ledger](05-lifecycle.md#whole-solution-ledger). When the ledger guard is tripped, the run starts by cutting cost before adding anything.

What starts a review (by hand or automatically after a session) is a setup choice ([07-setup.md](07-setup.md#setup-first-selfim)). Once a review runs, its changes apply automatically.

## When something new is added

The user: "add something new when confidence is high and issue of same nature keep repeating. We track our changes so greater freedom of automatic improvement is a better workflow for ease of use".

- **Patch first.** The default is to reinforce or patch an existing record. Matching is on scope + area + subject, using the precomputed similar-record list.
- **Every new finding starts as a candidate** (recorded, not loaded).
- **A candidate goes on trial automatically** when the issue has **repeated** (2+ occurrences, across sessions) **and** confidence is high: observed or user-sourced (not merely inferred), and never marked harmful **(my call, thresholds tunable)**.
- **Explicit user statements go on trial at once:** a decision the user states, or a correction the user confirms as a standing rule, counts as explicit confirmation and doesn't need to repeat (not when it came from pasted text).
- A reverted record with the same subject blocks re-adding the same change for a while, unless new evidence differs or comes from the user ([05-lifecycle.md](05-lifecycle.md#transitions)).
- Because every change is a commit and reverts are cheap, selfim leans towards acting rather than asking.

What happens after that is in [05-lifecycle.md](05-lifecycle.md).

## Approval gate

The user chose a narrow gate ("narrow for now"). Records derived from **commit messages or repo files** (outside text, used when learning from history) need the user's yes before they:

- load in every session (`always`),
- go global (area scope), or
- become blocking.

`/selfim` shows pending approvals as one batch ("3 rules from git history want to load every session: approve all / pick / skip"). Everything learned from the user's own sessions (corrections, decisions, workflow) stays fully automatic.

## Promotion pass

Once per run, a separate pass compares **know-how** records across projects (never facts, never evidence) to find what is proven in 2+ projects, and promotes it to area scope ([05-lifecycle.md](05-lifecycle.md#transitions)). This is the only exception to "no crossing between projects" ([07-setup.md](07-setup.md#data-placement)).

## Apply and trace

1. **Evidence commit:** pending session logs are committed first, in one commit (`selfim-evidence: <n> sessions`). Paths are staged explicitly, never with `-a`.
2. **Decision commits:** every change to selfim-owned records and outputs is one commit, with trailers recording:
   - **what** changed (record/skill id, scope)
   - **why** (one or two lines)
   - **evidence** (session ids, intervention kinds, stats)
   - **expected effect** (which intervention kind should drop) and **expected cost** (tokens added)

The user: "some form of git like commit structure on what changed and why so it can be reevaluated later in time". Changes move automatically; the user's later intervention is the signal to revert ("so things can move automatically and revert back if i later have to intervene").

**Undo is a forward commit**, not `git revert`. A change that failed (e.g. a token optimization whose issue came back) is undone by restoring the record's previous version. A record that was wrong in substance gets the state `reverted` (kept, so it blocks re-adding for a while). Either way the outputs are re-rendered and the reason is recorded. When a revert happens is defined in [05-lifecycle.md](05-lifecycle.md#transitions). The revert counts as evidence against that kind of change.

Stats are rebuilt from evidence and kept out of git; record files hold decisions only.

## Checks on the review agent's output

The review agent's judgment is checked by code before anything lands:

- **Evidence must exist.** Each change cites evidence ids; the apply code checks they exist and contain the claimed kind (correction, failure, note). A change with missing or mismatched evidence is rejected.
- **Source is derived, not claimed.** The record's `source` (observed, user, inferred, and whether it came from outside text) is set by the apply code from the cited evidence, never by the agent.
- **Merges are reversible.** A merge keeps compatible triggers and targets and records `merged_from`, so it can be split later.
- **Mechanical checks:** size caps (including the 10,000-character session-start limit), near-duplicates, skill validity, safe trigger patterns and the rule lint ([10-robustness.md](10-robustness.md#prompt-injection)).
- **Over budget:** when an addition exceeds a size cap, the apply code demotes the lowest value-per-character record that isn't pinned or on trial, instead of rejecting the addition.

Each batch is applied on a temporary branch and lands only if every check passes ([10-robustness.md](10-robustness.md#learning-runs)). `/selfim` lists the changes made since the user last looked.

## Suggestions for user-owned files

For the user's own CLAUDE.md, `authoring-*` skills and memory folder, the review only **suggests**, with evidence (e.g. "this `authoring-helm` rule never matched", "this proven `selfim-helm` rule could merge into `authoring-helm`"), based on the read-only excerpts it was given. It never edits them ([06-outputs.md](06-outputs.md#ownership)). Suggestions are shown by `/selfim` and logged; the user's accept/reject is recorded for the [suggestion-acceptance metric](05-lifecycle.md#long-term-metrics).
