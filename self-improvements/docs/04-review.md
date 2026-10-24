# 04 · Review: turning evidence into knowledge

## Who runs it

A separate agent (not the working session) processes queued digests ([02-capture.md](02-capture.md)) together with selfim's existing records. It decides what to add, patch, merge, tighten, promote, demote or revert. Mechanical counts come from scripts ([05-lifecycle.md](05-lifecycle.md#stats)); the agent spends tokens only on judgment.

What starts a review (by hand or automatically after a session) is a setup choice ([07-setup.md](07-setup.md#setup-first-invoke)). Once a review runs, its changes apply automatically.

## When something new is added

The user: "add something new when confidence is high and issue of same nature keep repeating. We track our changes so greater freedom of automatic improvement is a better workflow for ease of use".

- **Patch first.** The default is to reinforce or patch an existing record. Matching is on scope + area + subject, and the agent is handed a precomputed list of similar existing records.
- **Every new finding starts as a candidate** (recorded, not loaded).
- **A candidate goes on trial automatically** when the issue has **repeated** (2+ occurrences, across sessions) **and** confidence is high: observed or user-sourced (not merely inferred), and never marked harmful **(my call, thresholds tunable)**. A direct user correction counts as strong confidence but still needs the repetition, unless the user explicitly confirms the finding, which puts it on trial at once.
- Because every change is a commit and reverts are cheap, selfim leans towards acting rather than asking.

What happens after that is in [05-lifecycle.md](05-lifecycle.md).

## Apply and trace

Every change to selfim-owned outputs is one git commit in the runtime repo. The commit message records:

- **what** changed (record/skill id, scope)
- **why** (one or two lines)
- **evidence** (session ids, intervention kinds, stats)
- **expected effect** (which intervention kind should drop) and **expected cost** (tokens added)

The user: "some form of git like commit structure on what changed and why so it can be reevaluated later in time". Changes move automatically; the user's later intervention is the signal to revert ("so things can move automatically and revert back if i later have to intervene"). When a revert happens is defined in [05-lifecycle.md](05-lifecycle.md#transitions). A revert is itself a commit with its reason, and counts as evidence against that kind of change.

Only decisions are committed. Stats are rebuilt from evidence and kept out of git, so the history stays a readable log of decisions.

## Checks after each run

Scripts re-check size caps, near-duplicates and skill validity ([06-outputs.md](06-outputs.md)). A change that fails is not committed.

## Suggestions for user-owned files

For the user's own CLAUDE.md, `authoring-*` skills and memory folder, the review only **suggests**, with evidence (e.g. "this `authoring-helm` rule never matched", "this proven `selfim-helm` rule could merge into `authoring-helm`"). It never edits them ([06-outputs.md](06-outputs.md#ownership)).
