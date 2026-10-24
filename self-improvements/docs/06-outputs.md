# 06 · Outputs: what sessions get

## Ownership

| Output | Who changes it | How |
|---|---|---|
| `selfim` rule file (machine facts + a few universal rules) | selfim, automatically | git commits, revertible |
| `selfim-*` skills, one per area (`selfim-helm`, `selfim-gitops`, …) | selfim, automatically | git commits, revertible |
| project knowledge (per project folder in the repo) | selfim, automatically | git commits, revertible |
| user's CLAUDE.md, `authoring-*` skills, memory folder | user only | selfim **suggests** with evidence; never edits ([04-review.md](04-review.md#suggestions-for-user-owned-files)) **(my call: the user confirmed automatic changes for selfim's own files; this boundary for user-owned files was proposed and not contradicted)** |

The user's direction: "Best if it can produce it's own rule file, track it's own skills selfim-* with clear decison logs like git".

## Output shape

- **Few, broad skills:** one `selfim-<area>` skill per area, with reference files beneath it, rather than many small skills. The skill listing truncates descriptions, so a large library becomes invisible (observed in UniM0cha/self-improving-skills: 385 skills, 13 visible).
- **Generated from records:** `rules.md`, `selfim-*` skills and project indexes are rendered from the records ([03-knowledge.md](03-knowledge.md)). Edits are itemized, never wholesale rewrites (repeated summarising erodes content).
- **Size caps (my call):** the rule file and each `selfim-*` skill have a size budget. Adding past it forces a merge, tighter wording, or a demotion. This is what stops rules from creeping in complexity.

## Load levels

Each record's load level decides where it appears and what it costs:

| Load level | Where it loads | Cost paid |
|---|---|---|
| always | the rule file and the current project's index, injected at session start by a hook (CLAUDE.md stays untouched) **(my call)** | every session |
| trigger | the body of its `selfim-<area>` skill, loaded when the area comes up; or a pitfall warning from the pre-command check | per match |
| search | not loaded; read on demand from the repo (the index points to it) | per lookup |

- `selfim-*` are ordinary Claude Code skills. A session pays only their one-line descriptions until the body loads. A helm session never pays for gitops rules.
- The project index is one line per item, for the current project only.
- Hard limit: rule file + project index together stay under 10,000 characters, Claude Code's cap on context injected at session start.
- Which level a record gets is driven by its value ([05-lifecycle.md](05-lifecycle.md#impact-vs-token-cost)).

## Pre-command pitfall check

An optional hook ([07-setup.md](07-setup.md#setup-first-invoke)) matches each tool call against pitfall triggers (command regex, path glob, tool, area) with a script, no LLM. A match **warns** by default (one injected line naming the pitfall and what works instead); a pitfall that has proven itself can be raised to **block** **(my call; the user hasn't chosen)**. Each warning or block is logged as surfacing evidence.
