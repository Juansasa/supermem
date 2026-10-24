# selfim

A Claude Code plugin (skill + hooks + CLI) that makes agent sessions improve over time. It captures what happens in sessions, turns repeated issues into facts, rules and area skills, measures whether they reduce user interventions at an acceptable token cost, and promotes, demotes or reverts them. Every change is a git commit with what, why and evidence.

> Status: design draft. Full design in [docs/](docs/README.md). Nothing is built yet.

## The problem

- Agents repeat failed commands, re-check confirmed things, re-probe systems that don't change.
- Sessions on the same project never converge on the same facts and workflow.
- Rules pile up in CLAUDE.md, load everywhere, and nobody knows why they changed or whether they help.

**Success metric:** fewer user interventions (corrections, denials, interrupts, re-explanations). **Secondary:** token cost. selfim keeps a ledger of its own total cost (injected context, hooks, notes, review runs) against what it saves.

## How it works

```mermaid
flowchart LR
  subgraph S["Working session"]
    H["Hooks<br/>(scripts, no LLM)"]
  end
  subgraph R["selfim repo (git)"]
    L[("log/<br/>evidence JSONL")]
    K[("records<br/>fact · decision · pitfall<br/>procedure · rule")]
    O["outputs<br/>rules.md · selfim-* skills<br/>project index"]
  end
  A["Review agent<br/>(separate, offline)"]
  U["User-owned files<br/>CLAUDE.md · own skills"]

  H -- "pointers, flags,<br/>notes, checkpoints" --> L
  L -- "digest<br/>(script, ~1-2k tok)" --> A
  A -- "add · patch · revert<br/>1 change = 1 commit" --> K
  K -- "render with<br/>size caps" --> O
  O -- "inject / load<br/>by relevance" --> S
  A -. "suggest only" .-> U
```

Three rules hold it together:

1. **Hooks are cheap.** Scripts only, pointers instead of copies, successes counted rather than logged. The agent is only asked for a note at moments worth recording.
2. **The LLM only judges.** Scripts digest transcripts and count stats; the review agent decides what to add, patch, promote, demote or revert.
3. **Git is the decision log.** One change per commit (what, why, evidence, expected effect, expected cost). Undo is `git revert`.

## A session, step by step

```mermaid
sequenceDiagram
  autonumber
  actor User
  participant Agent
  participant Hooks
  participant Log as selfim log

  Hooks->>Log: SessionStart: project enabled? else exit. Sweep killed sessions
  Hooks->>Agent: inject rules.md + project index
  User->>Agent: prompt
  Hooks->>Log: correction? flag it (+ ask agent for a note)
  Agent->>Hooks: tool call
  Hooks-->>Agent: PreToolUse: known pitfall? warn / block
  Hooks->>Log: failure / denial, or failure→fix pair
  Agent->>Hooks: end of turn
  Hooks->>Log: Stop: checkpoint offset
  Hooks-->>Agent: decision made? ask for a note
  Agent->>Log: selfim note (fields, ≤400 chars)
  Hooks->>Log: PreCompact / SessionEnd: digest (script) + queue for review
```

## What gets tracked

| Layer | Written by | Contents |
|---|---|---|
| **Evidence** (immutable) | hooks, digest script, `selfim note` | interventions, failures + fixes, decisions, tool/skill use per area, git range, token usage, what selfim injected |
| **Records** (curated) | review agent | `fact` (target, stability, last verified) · `decision` (choice, why, revisit-when) · `pitfall` (triggers, what works instead) · `procedure` (what worked) · `rule` (targeted intervention kind) |

Scopes: **machine** (all projects) · **area** (helm, gitops, …; global) · **project** (remote systems, standards; never leaks). Target-bound facts never leave their project.

## Lifecycle

```mermaid
stateDiagram-v2
  direction LR
  [*] --> candidate: finding
  candidate --> trial: repeated 2+ and high confidence
  trial --> proven: targeted interventions drop
  trial --> reverted: user contradicts it, or no effect
  proven --> reverted: user contradicts it
  proven --> proven: optimize for tokens (re-trialled)
  trial --> stale: not loaded for N sessions
  proven --> stale: not loaded for N sessions
  stale --> trial: used again (back to previous state)
  stale --> archived: idle longer
```

| State | Loaded? | Judged by |
|---|---|---|
| candidate | no | repetition + confidence |
| trial | yes | **interventions** of its targeted kind |
| proven | yes | **token cost** (same effect, fewer tokens) |
| stale / archived | no | reader activity |
| reverted | no | kept to block re-adding the same change |

`pinned` records never move. Only reads (loaded, matched, surfaced) keep a record alive; selfim's own edits don't. Nothing is deleted: a revert is a forward commit that changes state.

## What a session pays

```mermaid
flowchart TB
  R["record"] --> Q{"load level"}
  Q -- always --> A1["rules.md / project index.md<br/>every session"]
  Q -- trigger --> A2["skill body (global selfim-area<br/>or project selfim-project),<br/>or pitfall warning on match"]
  Q -- search --> A3["read on demand<br/>via index pointer"]
```

| Skill layer | Holds | Visible in |
|---|---|---|
| global `selfim-<area>` (`~/.claude/skills`) | cross-project know-how | every session |
| project `selfim-project` (`<project>/.claude/skills`, git-excluded) | that project's knowledge | that project only |

A record stays `always` only while its impact pays for that cost; otherwise it drops to `trigger` or `search`. Skills are few and broad (one `selfim-<area>` per area) so their descriptions stay visible.

## Ownership

| selfim changes automatically (traced, revertible) | selfim only suggests |
|---|---|
| `rules.md`, `selfim-*` skills, project index, skill and triggers | your CLAUDE.md, your own skills, your memory folder |

Data lives in one git repo (`~/.selfim` by default): `machines/<host>/`, `areas/<area>/`, `projects/<id>/`. Nothing selfim writes in a project is committed or shows in `git status`, and project data never crosses into other projects.

## Setup

selfim is **off by default**. Install the plugin, then run `/selfim` in a repo to enable it there. The first time, it asks: new runtime repo or clone an existing one · which hook groups to use (core capture, signals, prompted notes, pitfall check) · review by hand or automatically after each session · backfill from past transcripts. After that, `/selfim` in another repo just enables it; `selfim disable` turns it off.

## Design docs

| | |
|---|---|
| [01-intent](docs/01-intent.md) | why, success metric, goals, anti-goals |
| [02-capture](docs/02-capture.md) | hooks, prompted notes, concision rules, digest, secrets |
| [03-knowledge](docs/03-knowledge.md) | scopes, record kinds and fields, project identity |
| [04-review](docs/04-review.md) | review agent, when something new is added, commits |
| [05-lifecycle](docs/05-lifecycle.md) | stats, states, transitions, impact vs. token cost |
| [06-outputs](docs/06-outputs.md) | ownership, skills, size caps, load levels, pitfall check |
| [07-setup](docs/07-setup.md) | runtime repo layout, setup prompts |
| [08-open](docs/08-open.md) | open design calls to review, builder's choices, prior art |
| [09-stack](docs/09-stack.md) | tooling: plugin, Python stdlib, storage, git trailers, `claude -p` review |
