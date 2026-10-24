# 06 · Outputs: what sessions get

## Ownership

| Output | Who changes it | How |
|---|---|---|
| `out/rules.md` (machine facts + a few universal rules) | selfim, automatically | git commits, revertible |
| global skills `selfim-<area>` (`selfim-helm`, `selfim-gitops`, …) | selfim, automatically | git commits, revertible |
| project knowledge: `index.md`, `selfim-project` skill, `triggers.json` | selfim, automatically | git commits, revertible |
| user's CLAUDE.md, `authoring-*` skills, memory folder | user only | selfim **suggests** with evidence; never edits ([04-review.md](04-review.md#suggestions-for-user-owned-files)) **(my call: the user confirmed automatic changes for selfim's own files; this boundary for user-owned files was proposed and not contradicted)** |

The user's direction: "Best if it can produce it's own rule file, track it's own skills selfim-* with clear decison logs like git". Where each output is linked or injected: [07-setup.md](07-setup.md#skill-layers).

## Output shape

- **Few, broad skills:** one global `selfim-<area>` skill per area and one `selfim-project` skill per project, each with reference files beneath it, rather than many small skills. The skill listing truncates descriptions (1,536 characters per skill), so a large library becomes invisible (observed in UniM0cha/self-improving-skills: 385 skills, 13 visible).
- **Global skills hold area-scope records only.** Project records never go into a global skill; they render into the project's index, `selfim-project` skill and `triggers.json`.
- **Generated from records:** all outputs are rendered from the records ([03-knowledge.md](03-knowledge.md)). Edits are itemized, never wholesale rewrites (repeated summarising erodes content).
- **Size caps (my call):** the rule file, each index and each skill have a size budget. Adding past it forces a merge, tighter wording, or a demotion. This is what stops rules from creeping in complexity.

## Load levels

Each record's load level decides where it appears and what it costs:

| Load level | Area or machine record | Project record | Cost paid |
|---|---|---|---|
| always | `out/rules.md`, injected at session start | `index.md`, injected at session start | every session |
| trigger | body of its `selfim-<area>` skill; pitfalls in `out/triggers.json` | body of `selfim-project`; pitfalls in the project's `triggers.json` | per load or match |
| search | not loaded; read on demand from the repo | not loaded; listed by id in `index.md` | per lookup |

- **Why a hook, not CLAUDE.md:** the user considered referencing selfim from the project's CLAUDE.md or a `.claude/rules/selfim.md` file and chose the hook ("hook is better"). Hook injection loads only in enabled projects, is measured exactly, and writes nothing into the project. The injected block starts with a one-line header naming its source and size (`selfim · <project> · N items · ~X tokens`) so its origin is visible in the session **(my call)**.
- **Starting level** for a record that goes on trial **(my call)**: rules, facts and decisions start at `always` while the always-loaded budget allows (otherwise `trigger`); pitfalls start at `trigger`. After that, the level follows value ([05-lifecycle.md](05-lifecycle.md#impact-vs-token-cost)).
- A session pays only skill descriptions until a body loads. A helm session never pays for gitops rules.
- The project index is one line per item, for the current project only.
- Hard limit: `rules.md` + `index.md` together stay under 10,000 characters, Claude Code's cap on context injected at session start.

## Pre-command pitfall check

A hook in the **pitfall check** group ([07-setup.md](07-setup.md#setup-first-selfim)) matches each tool call against the triggers in `out/triggers.json` and the current project's `triggers.json` (command regex, path glob, tool, area) with a script, no LLM.

- **Warn** (default): the call is allowed, and one line naming the pitfall and what works instead is added to the agent's context.
- **Block:** the call is denied with that line as the reason. Only pitfalls that have proven themselves with high warning precision can be raised to block **(my call; the user hasn't chosen)**.

Each warning or block is logged as surfacing evidence, along with what the agent did next.
