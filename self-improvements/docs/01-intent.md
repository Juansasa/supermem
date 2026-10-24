# 01 · Intent

## Why

Long sessions involve design back-and-forth, testing, finding holes, changing decisions. None of it gets tracked. Across several sessions on the same project, each working a different issue, the facts and workflow never converge, so the user's workflow doesn't improve over time. The user loses track of why rules were changed and whether a change actually made things better. Rules also grow more complicated and are often irrelevant to the session at hand ("gitops rules have no use in a helm work session").

There is no defined workflow today for **what to track, how to evaluate, and whether to promote or demote**. The user already has destinations (CLAUDE.md, a memory folder, `authoring-*` skills). What's missing is the process.

`selfim` is that process, packaged as a skill. supermem will be its first user ("so that work can later use this skill for it's own development").

## Measure of success

In the user's words: good means "producing desired result with least amount of intervention or correction". Concretely, over time:

- The same mistake is not made twice.
- The agent stops re-checking things the user has already confirmed.
- The agent stops retrying commands that have already failed.
- The agent stops re-probing systems that never change.

Failure means the same issues keep requiring the user's intervention. Worse is "more of the same issues in nature that requires more intervention".

**User interventions are the primary metric** (corrections, rejected tool calls, interrupts, re-explanations of settled things). **Token cost** is the second: it decides how proven knowledge is optimized ([05-lifecycle.md](05-lifecycle.md#impact-vs-token-cost)).

## Goals

- Capture, per session: goals, purpose, facts, decisions, progress, failures, errors, tools used, git activity, user interventions and token usage.
- Over many sessions, build a knowledge base per project (tech used, gitops structure, application design, patterns, coding and documentation standards) and per working area (helm, gitops, operations, machine access, …).
- Evaluate knowledge by evidence (matches, use, effect, cost) and promote or demote it accordingly.
- Every change is traced like a git commit (what, why, evidence) and can be reverted.
- Runs independently of the working session: a separate agent can do the evaluation from previously logged data alone.

## Constraints

- Simplicity and optimal token use come first. The user: "We need to find solutions that keep siplicity and produce optimal token usage."
- Global knowledge is fine only if "cheap to retrive and use, cheap means low token use".
- `selfim` keeps its own logs and decision metadata, independent from CLAUDE.md.
- One global git repo for all runtime data, local, pushable later. Every datum has one defined home by scope, and nothing selfim writes in a project is committed or shows in `git status` ([07-setup.md](07-setup.md#data-placement)).
- The skill must be importable by others: on first invoke it asks the user where to store its runtime data ([07-setup.md](07-setup.md)).
- Off by default: selfim does nothing until the user runs `/selfim` in a project ([07-setup.md](07-setup.md#activation-off-by-default)).

## Non-goals

- Editing the user's own hand-written files automatically ([06-outputs.md](06-outputs.md#ownership)).
- Multi-user or team sharing (beyond pushing the repo).
- Tracking ongoing work in depth; that is supermem's job ([03-knowledge.md](03-knowledge.md#session-note)).

## Anti-goals (failure even if it works)

- Interventions don't drop, or similar interventions grow.
- The rule set grows more complicated over time instead of tighter.
- Knowledge is loaded into sessions where it's irrelevant.
- selfim's injected tokens creep up without a matching drop in interventions.
- selfim as a whole (hooks, injected context, notes, review runs) costs more than it saves ([whole-solution ledger](05-lifecycle.md#whole-solution-ledger)).
- The working session spends noticeable tokens or effort on capture. Allowed exceptions, both capped and measured: the pre-command pitfall check (costs tokens only when a pitfall matches) and prompted notes at moments worth recording ([02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable)).
