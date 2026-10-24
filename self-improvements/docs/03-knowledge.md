# 03 · Knowledge: what is tracked

## Scopes

| Scope | Holds | Example |
|---|---|---|
| **machine** | facts about the machine the agent runs on (one folder per host); shared by all projects on it | OS version, installed tools, Colima not Docker Desktop, local limitations |
| **area (global)** | know-how about a working area, not tied to one project | helm, gitops, operations, machine access |
| **project** | remote systems and project specifics; never leaks to other projects | clusters, registries, hosts, gitops layout, design decisions, standards |

Project facts about remote systems carry their target explicitly (cluster, host, namespace…) so they can't be applied to the wrong system **(my call)**. Such facts are never promoted out of their project; only know-how can move from project to area scope ([05-lifecycle.md](05-lifecycle.md#transitions)).

## Record kinds

Knowledge records are curated by the review agent ([04-review.md](04-review.md)) from session evidence ([02-capture.md](02-capture.md#session-evidence)) and are durable.

| Kind | Holds | Targets |
|---|---|---|
| fact | statement, scope, target system, stability, last verified | re-probing systems that don't change |
| decision | question, choice, why, what it replaces, confirmed by user or not, **revisit when** | re-asking what was settled; losing why a rule changed |
| pitfall | what was tried, what went wrong, what worked instead, **triggers** (command regex, path glob, tool, area) | retrying failed commands |
| procedure | a sequence that worked (successful path) | becomes skill content |
| rule | instruction, area, the intervention kind it targets | the measured, promotable unit |

## Fields every record carries

| Field | Meaning |
|---|---|
| id | stable identifier, used in commits and stats |
| scope, area | where it applies ([Scopes](#scopes), [Areas](#areas)) |
| state | lifecycle state ([05-lifecycle.md](05-lifecycle.md#states)) |
| source | observed · user · inferred (trust) |
| load level | always · trigger · search ([06-outputs.md](06-outputs.md#load-levels)) |
| targets | the intervention kind it should reduce |
| state history | via git; counters (helpful/harmful, matches, use) live only in the derived stats database ([05-lifecycle.md](05-lifecycle.md#stats)) |
| provenance | sessions and projects it came from |

Facts carry a **last verified** date: stable facts are trusted without re-probing; past a limit they are marked stale and queued for one cheap re-check. Decisions carry **revisit when**: they resurface only when that condition matches.

## Areas

Areas are inferred mechanically from commands and paths (`helm`/`charts/**` → helm, `kubectl` → k8s, …), so records and evidence don't depend on the agent tagging them. The exact mapping is left to the builder.

## Project identity (my call)

A project is keyed by its git remote URL (falling back to the repo root path). Remotes are normalized (`host/owner/repo`, ssh and https alike, no `.git`). Subprojects inside one repo (e.g. `supermem/self-improvements/`) are declared as subpaths in the selfim repo's `registry.json` (at setup or with `/selfim on <dir>`); each subproject gets its own index and project skill, and a session counts as the subproject when its working directory is inside that subpath. No marker files are written into project repos ([07-setup.md](07-setup.md#data-placement)). Worktrees and clones map to the same project.

## Session note

Goals, purpose and progress are kept only as a short per-session note (goal, outcome, loose ends); tracking ongoing work is supermem's job **(my call, not yet confirmed)**.
