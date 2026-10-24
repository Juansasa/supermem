# 09 · Stack

Tooling chosen for building selfim. Based on research of the Claude Code docs (code.claude.com, fetched 2026-10-24) and of comparable tools; sources at the end. All choices here are **my call** unless stated otherwise.

## Packaging: a Claude Code plugin

A plain skill can't do the job: hooks declared in a skill's frontmatter only run while that skill is in use, so always-on capture ([02-capture.md](02-capture.md#hooks)) needs a **plugin**.

```
selfim/                          # plugin root (replaced on every update; holds no state)
  .claude-plugin/plugin.json
  hooks/hooks.json               # all hook registrations
  skills/selfim/SKILL.md         # setup, review, note: the user-facing skill
  bin/selfim                     # CLI on PATH while enabled: note, review, setup, stats
  lib/                           # Python modules used by hooks and CLI
```

| Location | Holds |
|---|---|
| `${CLAUDE_PLUGIN_ROOT}` | code only; replaced on update |
| runtime repo | everything in [07-setup.md](07-setup.md#runtime-repo). Chosen at setup; default `${CLAUDE_PLUGIN_DATA}` (survives updates) |
| `~/.claude/skills/selfim-<area>/` or `<project>/.claude/skills/selfim-<area>/` | symlinks to generated skills in the runtime repo, placed according to where selfim is active. Picked up live by Claude Code |

Install: plugin marketplace, or `--plugin-dir` for local development.

## Runtime: Python ≥ 3.9, standard library only

- Available wherever git is (macOS Command Line Tools ship both), standard on Linux. Startup measured at ~17–25 ms on this machine; bash+jq is ~5 ms but fragile for JSON; Go/Rust binaries need per-platform builds and signing; Node/Bun add a runtime.
- Capture hooks run **async** so they never block Claude; matchers and `if` conditions keep hooks from spawning when irrelevant. The pre-command pitfall check is the only synchronous hot-path hook.
- Hook modules import only `json`, `os`, `time`; heavier imports are deferred to the digest and review.
- No `tomllib` (3.11+) or YAML parser in stdlib: record frontmatter is JSON (or a flat key/value subset).

## Capture sources: hook payloads first

The transcript format is internal to Claude Code and "changes between versions" (sessions docs). So:

| Need | Source |
|---|---|
| failures, denials | `PostToolUseFailure`, `PermissionDenied` hook payloads |
| tool calls and results | `PreToolUse` / `PostToolUse` payloads |
| corrections | `UserPromptSubmit` `prompt` |
| decisions | `Stop` `last_assistant_message` (the transcript can lag behind) |
| subagent work | `SubagentStop` `agent_transcript_path` |
| token usage | the transcript: the **only** parsed part, in one tolerant module that skips unknown entries; if it breaks, capture continues without token stats |

Hook constraints that shape the design:
- `SessionEnd` has a 1.5 s budget shared by all hooks and isn't guaranteed on crash or kill: it only enqueues. The `Stop` checkpoint per turn is what makes capture reliable.
- `SessionStart` injected context is capped at **10,000 characters**: the hard cap for rule file + project index together.
- Stop prompts must check `stop_hook_active` (block at most once).

## Storage

| Data | Format | In git |
|---|---|---|
| evidence | append-only JSONL, one file per session, `.gitattributes` `merge=union` | yes |
| records | one markdown file per record, JSON frontmatter, stable id | yes |
| rendered outputs | `rules.md`, `selfim-*` skills, project indexes | yes |
| stats and search | SQLite, rebuilt from the files (stdlib `sqlite3`, FTS5) | no (gitignored) |

## Git

`git` CLI via `subprocess`, always with `-C <repo>`. Decisions go in commit **trailers** (`Selfim-What`, `Selfim-Why`, `Selfim-Evidence`, `Selfim-Effect`, `Selfim-Cost`), written with `git commit --trailer` and queried with `git log --format='%(trailers:...)'`. No git libraries.

## Review agent: `claude -p`

Run from the CLI (`selfim review`), with the runtime repo as working directory:

- `--output-format json --json-schema <changes schema>`: the agent returns proposed changes; selfim's own code applies, checks and commits them.
- `--permission-mode dontAsk`, a short `--allowedTools` list (read and edit; no network, no arbitrary shell).
- `--max-budget-usd`, `--max-turns`, `--no-session-persistence`.
- A `PreToolUse` guard denies any file path outside the runtime repo. That hook is the enforcement; permission rules alone are not treated as a sandbox.
- `--bare` (skips the user's hooks, skills, plugins and CLAUDE.md) is used when an API key is available; subscription users run without it.

Rejected: Agent SDK (more code, only needed for programmatic approval callbacks); in-session subagent (inherits the user's session and permissions; not isolated).

## Supporting pieces

| Concern | Choice | Instead of |
|---|---|---|
| secret redaction | built-in rules adapted from gitleaks' MIT rule set (cloud keys, GitHub/Anthropic/OpenAI/Slack tokens, JWTs, private keys, `Bearer`) + entropy check on `KEY=value` | gitleaks binary (optional extra in the digest when installed); trufflehog (network verification, too slow) |
| near-duplicates | normalized word-shingle Jaccard + `difflib` tie-break; FTS5 `bm25()` for candidates once records grow; the review agent decides | rapidfuzz (dependency); embeddings (only worth it at thousands of paraphrased records) |
| token estimates | real `usage` from transcripts for cost; ~3 chars/token (conservative) for size caps | tiktoken (wrong vocabulary); `count_tokens` API as optional calibration (needs an API key) |
| transcript parsing | own ~150-line tolerant parser | depending on claude-code-log / ccusage / cc-transcript (used as references only) |

## Not reused

Claude Code's auto memory (`~/.claude/projects/<project>/memory/`) is the user's memory folder: selfim only suggests changes there ([06-outputs.md](06-outputs.md#ownership)).

## Unverified (checked at build time)

- whether `SessionEnd` fires after a hard kill (the design doesn't depend on it);
- path-scoped syntax for `Edit` permission rules (the guard hook covers it);
- whether plugin-shipped skills may be symlinks (generated skills are linked into user/project skill dirs instead).

## Sources

- Claude Code: [hooks](https://code.claude.com/docs/en/hooks.md) · [headless](https://code.claude.com/docs/en/headless.md) · [skills](https://code.claude.com/docs/en/skills.md) · [plugins](https://code.claude.com/docs/en/plugins/overview.md) · [plugin components](https://code.claude.com/docs/en/plugins/components.md) · [sessions](https://code.claude.com/docs/en/sessions.md) · [memory](https://code.claude.com/docs/en/memory.md) · [token counting](https://platform.claude.com/docs/en/build-with-claude/token-counting)
- Git trailers: https://git-scm.com/docs/git-interpret-trailers
- Prior art: [ACE](https://academy.dair.ai/papers/agentic-context-engineering-ace) · [Letta sleep-time](https://docs.letta.com/guides/agents/architectures/sleeptime) · [Hermes Agent](https://hermes-agent.nousresearch.com/docs) · [claude-reflect](https://smithery.ai/skills/BayramAnnakov/claude-reflect) · [claude-mem](https://docs.claude-mem.ai/architecture/overview) · [claude-code-log](https://github.com/daaain/claude-code-log) · [ccusage](https://npmjs.com/package/ccusage) · UniM0cha/self-improving-skills
