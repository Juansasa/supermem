# 04 · Review: turning evidence into knowledge

## Who runs it

A separate agent (not the working session) processes queued digests ([02-capture.md](02-capture.md#digest-script-zero-tokens)) together with selfim's existing records. It decides what to add, patch, merge, tighten, promote, demote or revert. Mechanical counts come from scripts ([05-lifecycle.md](05-lifecycle.md#stats)); the agent spends tokens only on judgment.

How a run works ([09-stack.md](09-stack.md#review-agent-claude--p)):

1. `selfim review` (from `/selfim review`, the CLI, or automatically after a session, per setup) starts a **detached** `claude -p` run with the runtime repo as working directory, so it isn't bound by the calling session's tool timeout.
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
- **Explicit user statements go on trial at once:** a decision the user states, or a correction the user confirms as a standing rule, counts as explicit confirmation and doesn't need to repeat.
- A reverted record with the same subject blocks re-adding the same change unless new evidence differs from what caused the revert.
- Because every change is a commit and reverts are cheap, selfim leans towards acting rather than asking.

What happens after that is in [05-lifecycle.md](05-lifecycle.md).

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

**A revert is a forward commit**, not `git revert`: it sets the record's state to `reverted` (keeping the record, so it blocks re-adding), re-renders the outputs, and records why. When a revert happens is defined in [05-lifecycle.md](05-lifecycle.md#transitions). The revert counts as evidence against that kind of change.

Stats are rebuilt from evidence and kept out of git; record files hold decisions only.

## Checks after each run

Scripts re-check size caps (including the 10,000-character session-start limit), near-duplicates and skill validity ([06-outputs.md](06-outputs.md)). A change that fails is not committed.

## Suggestions for user-owned files

For the user's own CLAUDE.md, `authoring-*` skills and memory folder, the review only **suggests**, with evidence (e.g. "this `authoring-helm` rule never matched", "this proven `selfim-helm` rule could merge into `authoring-helm`"), based on the read-only excerpts it was given. It never edits them ([06-outputs.md](06-outputs.md#ownership)). Suggestions are shown by `/selfim` and logged; the user's accept/reject is recorded for the [suggestion-acceptance metric](05-lifecycle.md#long-term-metrics).
