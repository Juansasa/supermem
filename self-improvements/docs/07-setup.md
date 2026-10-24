# 07 · Setup and runtime data

## Runtime repo

One global git repo, local, pushable later. The user: "we can keep everything in one global git repo for selfim so it can be kept local and later pushed if needed".

## Data placement

The user: "in any machine there are multiple projects and whatever selfim need to store it should be well defined so we dont clutter data where it doesnt belong". Layout **(my call)**:

```
<selfim-repo>/
  config.json                    # setup answers
  registry.json                  # project id ← git remotes / paths / subpaths
  machines/<host>/               # machine-scope facts, one folder per machine
  areas/<area>/records/          # area know-how (helm, gitops, …), cross-project
  projects/<id>/
    project.json                 # identity, paths, active or not
    records/                     # project facts, decisions, pitfalls (never leave)
    log/<session>.jsonl          # this project's evidence only
    index.md                     # rendered project index
  projects/_unscoped/log/        # sessions outside any known project
  out/rules.md                   # rendered always-loaded rules
  out/skills/selfim-<area>/      # rendered area skills (linked into ~/.claude/skills)
  reviews/<date>.jsonl           # review-run log (cost, changes)
  .cache/                        # queue, derived SQLite, stats (gitignored)
```

Rules:

1. **Nothing is written into project repos.** Project identity lives in `registry.json` (no marker files); project-only activation uses the project's `.claude/settings.local.json`, which Claude Code keeps out of git; generated skills are linked only into `~/.claude/skills/`.
2. **One home per datum, decided by scope:** machine facts → `machines/<host>/`, area know-how → `areas/<area>/`, everything project-specific → `projects/<id>/`.
3. **No crossing between projects.** A project's records and evidence are read only when working on that project, or by the review agent reviewing it. Only know-how moves project → area, with provenance listing project ids only, never another project's content. `_unscoped` sessions can teach machine facts, nothing more.
4. **Retention:** evidence is kept N days, then compacted into stats summaries **(my call, tunable)**; archived records stay; `.cache/` can be deleted any time and is rebuilt.

`machines/<host>/` keeps two machines' facts apart once the repo is pushed and used elsewhere.

### Borrowed from Claude Code's own sessions

The user: "borrow claude session ideas wherever applicable". selfim's layout follows how Claude Code stores sessions and memory (`~/.claude/projects/<project>/`), so the two line up:

| Claude Code | selfim |
|---|---|
| one transcript per session, named by `session_id` | one evidence file per session, `projects/<id>/log/<session_id>.jsonl`, using Claude's own `session_id`, so every entry can point back to its transcript |
| subagent transcripts nested under their session (`<session>/subagents/agent-<id>.jsonl`) | subagent evidence stays in the parent session's file, tagged with `agent_id` and pointing to the subagent transcript |
| sessions can be resumed or forked (SessionStart `source`) | a resumed session keeps appending to its file; a fork starts a new file that records its parent session |
| auto memory is shared across a repo's worktrees | project identity is the repo, not the folder, so worktrees and clones share one project |
| auto memory loads a short index (`MEMORY.md`) and reads topic files on demand | project `index.md` is always loaded; records are read on demand (the `search` load level) |
| transcripts are deleted after `cleanupPeriodDays` (default 30) | evidence keeps only pointers into transcripts, so each session must be **digested before Claude Code deletes its transcript**. Setup reads `cleanupPeriodDays`, and the session-start reminder escalates when queued sessions get close to it. Evidence retention N defaults to the same period |

Claude Code's per-folder project naming (cwd with non-alphanumerics replaced) is **not** copied: it splits one repo into several projects when it's opened from different folders.

## Install

selfim ships as a Claude Code **plugin** (hooks + the `selfim` skill + a `selfim` CLI), installed from a plugin marketplace or, for development, with `--plugin-dir` ([09-stack.md](09-stack.md#packaging-a-claude-code-plugin)). The plugin folder holds code only; all state lives in the runtime repo.

## Setup (first invoke)

The user: "When user import/invoke this skill it should prompt user about where to store its runtime data", and "user decide if they want to add it to their hook skill should prompt this for setup". On first invoke, the skill asks, one prompt each, and records the answers in the repo:

1. **Where to store runtime data** (creates and `git init`s the repo). Default: the plugin's persistent data folder (`${CLAUDE_PLUGIN_DATA}`), which survives plugin updates.
2. **Which hook groups to install** (each shown with what it does; see [02-capture.md](02-capture.md#hooks)):
   - **core capture:** SessionStart (incl. loading), Stop checkpoint, SubagentStop, PreCompact, SessionEnd. Without it, capture runs only when the skill is invoked by hand.
   - **signals:** correction flags (UserPromptSubmit), failure/fix pairs (PostToolUse).
   - **prompted notes:** the agent is asked for a structured note at moments worth recording ([02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable)).
   - **pitfall check:** warn/block before commands ([06-outputs.md](06-outputs.md#pre-command-pitfall-check)).
   - **git:** post-commit hook tagging commits with their session, optional.
3. **Where selfim is active:** this project only, or every session on the machine. This scopes the hooks: the plugin is enabled in the project's `.claude/settings.local.json` (not committed) or at user level. Nothing is written into the project repo ([Data placement](#data-placement)). Area skills are linked into `~/.claude/skills/` only; whether their descriptions can be limited to active projects (e.g. via the skill `paths` field) is checked at build time. Suggested default: this project only **(my call)**.
4. **What starts the review step:** by hand (`/selfim review`), with a one-line reminder from the session-start hook when the queue passes a threshold; or automatically in the background after a session ends. Suggested default: by hand **(my call)**, since background runs spend tokens unattended. Either way, once a review runs its changes apply automatically; with the by-hand default, nothing improves until a review is started.
5. **Backfill** from existing transcripts ([02-capture.md](02-capture.md#backfill-my-call)). Default: off.

Every answer can be changed later by re-running setup.

## Where the source lives

Design and implementation of the skill live in `supermem/self-improvements/`. supermem's own docs (`supermem-docs/`) are for a different system and don't apply here.
