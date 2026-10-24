# 07 · Setup and runtime data

## Runtime repo

One global git repo, local, pushable later. The user: "we can keep everything in one global git repo for selfim so it can be kept local and later pushed if needed".

Default location: `~/.selfim` **(my call)**. Not the plugin's data folder: that folder is likely removed when the plugin is uninstalled, and the decision history must outlive the plugin. Hooks find a non-default location through a one-line pointer file in `${CLAUDE_PLUGIN_DATA}`.

## Activation: off by default

The user: "selfim should be false be default, and enabled by user on first call /selfim".

- The plugin's hooks are registered machine-wide, but **every hook first checks whether the current project is enabled** in `registry.json` and exits immediately if not (no logging, no injection). Until setup has run, hooks do nothing except a one-line hint at session start in a git repo: "selfim is installed; run /selfim to enable it here".
- `/selfim` in a repo: the first time on the machine, it runs setup (below) and turns selfim on for that project. In any later repo, it only turns it on there and offers to learn from existing history, without repeating the rest of setup. `/selfim off` turns a project off again; its data stays. In a repo where selfim is already on, `/selfim` shows status (queued sessions, records on trial, cost vs. savings, pending suggestions) and offers to learn when there's a large backlog **(my call)**.
- Uninstalling the plugin stops all hooks; the runtime repo stays. Full cleanup means removing the `selfim-*` skill links from `~/.claude/skills/` and enabled projects ([README](../README.md#uninstall)).
- Enablement lives in `registry.json`, keyed by the project's identity, so all worktrees and clones of an enabled repo are enabled, and nothing is written into the project to switch selfim on.
- Cost in disabled repos: hook processes still start and exit (a few ms each; the synchronous pre-command hook runs on every tool call). This latency is in the [ledger](05-lifecycle.md#whole-solution-ledger).

## Data placement

The user: "in any machine there are multiple projects and whatever selfim need to store it should be well defined so we dont clutter data where it doesnt belong". Layout **(my call)**:

```
<selfim-repo>/
  registry.json                  # project id ← normalized remotes / paths / subpaths; enabled flag
  machines/<host-id>/
    config.json                  # this machine's setup answers and local paths
    records/                     # machine-scope facts
  areas/<area>/records/          # area know-how (helm, gitops, …), cross-project
  projects/<id>/
    project.json                 # identity, subpaths
    records/                     # project facts, decisions, pitfalls, rules (never leave)
    log/<session_id>.jsonl       # this project's evidence only
    index.md                     # rendered project index (always loaded)
    triggers.json                # rendered pitfall triggers for the pre-command hook
    skill/selfim-project/        # rendered project skill
  out/rules.md                   # rendered always-loaded rules (machine + universal)
  out/triggers.json              # rendered area pitfall triggers
  out/skills/selfim-<area>/      # rendered global area skills
  reviews/<date>.jsonl           # review-run log (cost, changes)
  .cache/                        # queue, derived SQLite, stats (gitignored)
```

Rules:

1. **Nothing selfim writes in a project is committed or shows in `git status`.** Project identity and enablement live in `registry.json` (no marker files, no settings files). The only thing placed in a project is the project-skill link (below), added to the repo's `.git/info/exclude`.
2. **One home per datum, decided by scope:** machine facts → `machines/<host-id>/`, area know-how → `areas/<area>/`, everything project-specific → `projects/<id>/`.
3. **No crossing between projects.** A project's records and evidence are read only when working on that project, or by the review agent reviewing it. One exception, the **promotion pass**: it compares know-how records (never facts, never evidence) across projects to find what is proven in 2+ projects. Promoted know-how carries provenance as project ids only. Sessions outside an enabled project are not captured; machine facts are learned from sessions in enabled projects.
4. **Retention:** evidence is kept N days (default: Claude Code's `cleanupPeriodDays`), then compacted into stats summaries, except sessions still referenced by a record on trial or by a baseline **(my call, tunable)**. Archived and reverted records stay. `.cache/` can be deleted any time and is rebuilt.

### Skill layers

The user: "there are multiple layers, we'll have global selfim skills and project skills". Claude Code lists every skill it finds (user skills, the current project's `.claude/skills/`, enabled plugins) in every session, so each layer is placed where it belongs:

| Layer | Holds | Linked as | Listed in |
|---|---|---|---|
| global skills `selfim-<area>` | cross-project know-how | `~/.claude/skills/selfim-<area>` → `out/skills/selfim-<area>` | every session (global by definition) |
| project skill `selfim-project` | the project's knowledge beyond the short index: tech used, gitops layout, design, standards, patterns | `<project>/.claude/skills/selfim-project` → `projects/<id>/skill/selfim-project`, added to `.git/info/exclude`; created by the session-start hook in each enabled worktree | that project only |
| always-loaded | machine facts, universal rules, project index | injected by the session-start hook | enabled projects only |
| pitfall triggers | project and area pitfalls | `triggers.json` files read by the pre-command hook | enabled projects only |

Global skills also appear in repos where selfim is off; there they work but go unmeasured. That is the accepted cost of global knowledge; their descriptions are kept short. Updating a skill never requires loading it: the review edits the rendered files and Claude Code picks up changes live (a session that already loaded a skill keeps the old text until it reloads).

### Borrowed from Claude Code's own sessions

The user: "borrow claude session ideas wherever applicable".

| Claude Code | selfim |
|---|---|
| one transcript per session, named by `session_id` | one evidence file per session, `projects/<id>/log/<session_id>.jsonl`, so every entry can point back to its transcript |
| subagent transcripts nested under their session | subagent evidence stays in the parent session's file, tagged with `agent_id` and pointing to the subagent transcript |
| sessions can be resumed or forked (SessionStart `source`) | a resume keeps appending to its file, and only the new part is queued again; a fork starts a new file that records its parent and counts only from the fork point |
| auto memory is shared across a repo's worktrees | project identity is the repo, not the folder: worktrees and clones share one project |
| auto memory loads a short index (`MEMORY.md`) and reads topic files on demand | project `index.md` is always loaded; records are read on demand (the `search` load level) |
| transcripts are deleted after `cleanupPeriodDays` (default 30) | the zero-token digest runs automatically after every session ([02-capture.md](02-capture.md#digest-script-zero-tokens)), so nothing depends on a transcript surviving until review |

Claude Code's per-folder project naming is **not** copied: it splits one repo into several projects when opened from different folders.

## Install

selfim ships as a Claude Code **plugin** (hooks + the `selfim` skill + a `selfim` CLI), installed from a plugin marketplace or, for development, with `--plugin-dir` ([09-stack.md](09-stack.md#packaging-a-claude-code-plugin)). Install it at user scope; activation is per project (above). The plugin folder holds code only; all state lives in the runtime repo.

## Setup (first `/selfim`)

The user: "When user import/invoke this skill it should prompt user about where to store its runtime data", and "user decide if they want to add it to their hook skill should prompt this for setup". The first `/selfim` asks, one prompt each, and records the answers in `machines/<host-id>/config.json`:

1. **Runtime repo:** create a new one (default `~/.selfim`, `git init`), or **clone an existing one** (e.g. from another machine).
2. **Hook groups** (each shown with what it does; see [02-capture.md](02-capture.md#hooks)). All hooks are registered by the plugin; a group that's off exits immediately.
   - **core capture:** SessionStart, Stop checkpoint, SubagentStop, PreCompact, SessionEnd. Always on while a project is enabled.
   - **signals:** correction flags, failure→fix pairs.
   - **prompted notes** ([02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable)).
   - **pitfall check** ([06-outputs.md](06-outputs.md#pre-command-pitfall-check)).
3. **When selfim learns:** by hand (`/selfim learn`), with a one-line reminder at session start when the queue passes a threshold; or automatically in the background after a session ends. Suggested default: by hand **(my call)**, since background runs spend tokens unattended. Either way, once a review runs its changes apply automatically, and the zero-token digest always runs automatically.
4. **Learn from existing history or start fresh:** asked whenever a project is turned on (and later when a large backlog builds up), showing what was found and the estimated cost ([02-capture.md](02-capture.md#learning-from-existing-history)).

Setup also:
- adds a permission rule allowing `Bash(selfim note:*)`, so writing a note never causes a permission prompt (that would be an intervention caused by selfim);
- reads `cleanupPeriodDays` to set evidence retention;
- enables the current project and asks whether it contains subprojects (declared as subpaths in `registry.json`; also possible later with `/selfim on <dir>`).

Every answer can be changed later with `/selfim settings`.

## Commands

The user: "ensure selfim arg have intuitive naming". One entry point; everything else is a plain verb **(my call)**:

| Command | Does |
|---|---|
| `/selfim` | the main entry: turns selfim on here (first time: settings questions), shows status, and offers to learn from history when there's a lot unprocessed |
| `/selfim learn` | learn from captured sessions now (runs the review) |
| `/selfim settings` | change the setup answers |
| `/selfim on` / `/selfim off` | turn selfim on or off for this repo; turning off keeps the data |
| `/selfim on <dir>` | treat a folder of this repo as its own subproject |
| `selfim note …` | used by the agent when prompted, not by people |

Upgrades after a plugin update run automatically; there is no user command for them.

## Second machine

Setup option 1 "clone an existing repo" pulls the shared history. Per-machine data stays apart: `machines/<host-id>/` holds that machine's config and facts, keyed by a stable id (hardware UUID, not the hostname). Remotes are normalized (`host/owner/repo`, ssh and https alike, no `.git`), so the same project gets the same id on every machine. Each review pulls and rebases before it commits, and pushes after if a remote is configured.

## Upgrades

`config.json` and every record carry a `schema_version`. After a plugin update, the first session-start hook runs the upgrade script (committed as one migration commit) before anything else.

## Where the source lives

Design and implementation of the skill live in `supermem/self-improvements/`. supermem's own docs (`supermem-docs/`) are for a different system and don't apply here.
