# 02 · Capture: how sessions are tracked

## Hooks

The user: "make use of claude hooks to add hooks to relevant hooks to helps the skills capture relevant facts, logs and progress even when skills isnt in play". Capture runs through hooks; it doesn't depend on the selfim skill being loaded. All hooks are scripts (no LLM), and all of them exit immediately in projects where selfim isn't enabled ([07-setup.md](07-setup.md#activation-off-by-default)).

| Hook | Records | Why it's needed beyond the transcript |
|---|---|---|
| SessionStart | session opened, project ([identity](03-knowledge.md#project-identity-my-call)), git HEAD, model and Claude Code version, what selfim injected and its size. Also: the catch-up sweep, migrations after an update, the project-skill link | start of the commit range; injected-token cost; fair comparison across model changes |
| UserPromptSubmit | flag when a prompt looks like a correction or states a decision | marks likely interventions and user decisions as they happen; the digest confirms them |
| PreToolUse | pitfall check, warn/block ([06-outputs.md](06-outputs.md#pre-command-pitfall-check)); logs each firing | the only real-time action |
| PostToolUse | a success of the same command family right after a failure becomes a "what fixed it" pair; probes that verified a fact | cheap failure→fix pairing |
| PostToolUseFailure | failures (tool, normalized command, redacted one-line error) | failures arrive directly |
| PermissionDenied | denied tool calls | a denial is an intervention |
| Stop (each turn) | checkpoint: transcript offset captured so far | killed sessions are still captured; no double-processing |
| SubagentStop | the subagent's transcript path | subagent work lives in separate transcript files |
| PreCompact | checkpoint, digest and queue | long sessions are captured in pieces |
| InstructionsLoaded | which native instruction files loaded, and their size | native baseline for the ledger |
| SessionEnd | end marker, digest and queue | the normal hand-off. The user: "a hook to run it before closing a session or before compaction is optimal" |

**Synchronous vs. async.** Anything that adds text to the agent's context (session-start injection, pitfall warnings, note prompts) must run synchronously, so each hook is split: a fast synchronous part that decides and injects, and an async part that logs. Only the synchronous parts count toward hook latency.

**Hook payloads are the primary source.** The transcript format is internal to Claude Code and changes between versions, so only token usage is read from it, in one tolerant module; if that breaks, capture continues without token stats ([09-stack.md](09-stack.md#capture-sources-hook-payloads-first)).

**Commit range, not a git hook.** Commits made during a session are taken from the git range between the HEAD recorded at session start and HEAD at the last checkpoint. No git hooks are installed in project repos.

**Progress** mostly comes free: the agent's todo/task tool calls arrive through tool hook payloads and the digest extracts them.

**selfim ignores itself.** Hooks skip `selfim` CLI calls, and a review run sets `SELFIM_REVIEW=1` so every hook exits ([04-review.md](04-review.md#who-runs-it)).

### Killed sessions and the catch-up sweep

A session can be killed at any time; nothing special is needed for that. Hooks write evidence as each event happens, so a kill loses at most the event in flight, and token usage is read later from Claude Code's transcript, which survives the kill. A resumed session (same `session_id`) keeps appending to the same log.

What a kill skips is the end marker and the queueing. So every SessionStart runs a **catch-up sweep**: logs with checkpoints but no end marker that aren't queued get digested and queued. Script only, no tokens. The queue can always be rebuilt from the logs and `reviews/*.jsonl`, so deleting `.cache/` loses nothing.

## Prompted notes (high precision, when applicable)

The user: "prompt is fine so we can capture useful data with high precision whenever applicable". The transcript records what happened, not always why. When a hook detects a moment worth recording, it injects one short line asking the agent to write a structured note with a small CLI (`selfim note`), so the reasoning is captured by the agent that has it.

| Moment (detected by script) | Hook | Note asked for |
|---|---|---|
| user correction flagged | UserPromptSubmit | what was wrong, the correct rule or fact, its scope |
| user states a decision ("let's go with", "instead of", "always/never …") | UserPromptSubmit | decision: question, choice, reason, what it replaces |
| failure followed by a fix of the same command family, not matching a known pitfall | PostToolUse | pitfall: what failed, why, what works instead |
| decision language in the agent's last message | Stop | decision: question, choice, reason, what it replaces |
| a fact about a system was verified (probe command succeeded on a target) | PostToolUse | fact: statement, target, stability |

Guards, so prompting stays cheap **(my call, values tunable)**:
- at most one prompt per turn and a per-session cap; a Stop prompt blocks at most once (`stop_hook_active`) and never twice in a row;
- no prompt when the same note already exists (matched on scope + area + subject);
- a note is one CLI call with fields, not prose; setup allows `selfim note` so it never triggers a permission prompt.

Notes are session evidence with `source: noted` (higher precision than extracted evidence). A user correction that contradicts a loaded record also marks that record **harmful** at once ([05-lifecycle.md](05-lifecycle.md#transitions)). Each prompt type is itself measured like a record: its cost is the prompt plus the note; its impact is whether its notes become records that prove themselves. A prompt type that rarely yields useful notes is turned down or off.

## Digest (script, zero tokens)

The digest runs **automatically** at SessionEnd, PreCompact and in the catch-up sweep, whatever the review setting, so a session never depends on its transcript surviving until review. It reduces the session's hook records, notes and (for token usage) the transcript to a compact digest of the moments that matter:

- failed commands and errors (with the command that fixed them, if any)
- user interventions: corrections, tool-call denials, interrupts, repeated explanations
- decisions and their stated reasons
- tools and skills used; commands per area
- successful sequences (what worked), as raw material for procedures
- git log for the session's commit range
- token usage for the session (from the usage figures in the transcript)
- selfim's own surfacing: what was injected at session start, which `selfim-*` skills loaded, which pitfall warnings/blocks fired and what the agent did next
- selfim's own cost: sizes of everything injected, note calls, and per-hook timings (for the [whole-solution ledger](05-lifecycle.md#whole-solution-ledger))

Resumed sessions are digested again from their last offset only; forks are counted from the fork point, so the parent's history isn't counted twice. The LLM reads the digest, never the full transcript. If a needed detail is missing from the payloads, the digest may look it up in the transcript by its stored pointer, tolerating format changes.

## Processed-data cursors

The user: "keep track of already processed data somehow so it doesnt reconsume same data". Every stage records what it has consumed, **in git** (not in `.cache/`, so deleting the cache or moving to another machine never causes reprocessing), and every stage is idempotent: running it twice on the same input changes nothing **(my call)**.

| Source → stage | Cursor | Stored in |
|---|---|---|
| transcript → digest | last digested message id/offset per session; resumes and forks continue from it | marker lines in the session's own log (`projects/<id>/log/<session_id>.jsonl`) |
| hook records → digest | everything after the last `digested` marker in the same log | same file |
| digest → review | digest ids consumed by each review run | `reviews/<date>.jsonl` and the evidence commit's trailers |
| queue | digests not yet consumed by any review: derived, never stored as truth | rebuilt into `.cache/` |
| git history → bootstrap and later runs | last processed commit per branch; commits are deduplicated by sha, so rebases and merges don't count twice | `projects/<id>/project.json` |
| past transcripts → bootstrap | session ids already processed; a session with an existing log is never re-read | the logs themselves |
| native memory and instructions | content hash per file; re-read only when the hash changes | `projects/<id>/project.json` (machine-level files: `machines/<host-id>/config.json`) |

A digest records the version of the digest script that produced it. A newer script version only re-digests on an explicit `selfim migrate`, never silently.

## Session evidence

Hook records, prompted notes and digests are the **session-evidence layer**: immutable, stored as JSONL per session under the project it belongs to (`projects/<id>/log/`; sessions outside any project are not captured, since selfim is enabled per project; see [07-setup.md](07-setup.md#data-placement)). All counting ([05-lifecycle.md](05-lifecycle.md#stats)) comes only from this layer. Knowledge records ([03-knowledge.md](03-knowledge.md)) are curated from it. Evidence is committed by the review in a separate evidence commit before its decision commits ([04-review.md](04-review.md#apply-and-trace)).

## Keeping captured data concise

The user: "ensure our hook capture consise data without irrelevant verbosity". Rules for everything hooks, notes and digests write **(my call; limits tunable)**:

| Rule | Detail |
|---|---|
| **Point, don't copy** | Store the transcript path + message id/offset instead of content. Hooks never copy messages or tool output. |
| **Fixed, small schema** | One JSONL line per event, short fixed keys, per hook type. Hard cap per line (about 300 bytes); anything longer is truncated, not stored in full. |
| **Successes are counted, not logged** | Successful routine calls (reads, listings, searches, passing commands) become per-area counts in the digest. Only failures, interventions, fixes, decisions and surfacing get individual entries. |
| **Errors as one line** | Keep the first meaningful error line (ANSI stripped, paths shortened, secrets redacted), truncated to about 160 chars. No stack traces or full output. |
| **Commands normalized** | Whitespace collapsed, long arguments and inline data elided (`…`), truncated to about 160 chars. Enough to match a pitfall pattern, no more. |
| **Flags carry no prose** | Correction and decision flags record the matched pattern and the first ~80 chars of the prompt; the digest reads the rest from the transcript if needed. |
| **Repeats collapse** | The same failure (same normalized command + error) within a session becomes one entry with a count. |
| **Notes are fields, not prose** | `selfim note` takes named fields with per-field length limits and rejects anything longer; one note ≤ about 400 chars. |
| **Digest has a budget** | Each session digest is capped (about 1–2k tokens), filled by priority: interventions, then failures and fixes, decisions, notes, surfacing, then counts. Lowest-priority items are dropped first, with a count of what was dropped. |

Log size per session is tracked as part of selfim's own cost ([05-lifecycle.md](05-lifecycle.md#whole-solution-ledger)); a hook type whose entries are rarely used in review gets trimmed.

## Secrets (my call)

The digest script and every hook redact secrets (known token patterns, values of env vars whose names look secret, `Authorization:`/password-like fields) before anything is written. Raw transcripts are never copied into the repo, only referenced by path. This follows the user's global rule never to log secret values; the repo is meant to be pushable.

## Bootstrap on first enable

When `/selfim` enables a project, it **asks the user** whether to bootstrap from existing history or start from nothing, because bootstrapping costs tokens. The user: "skill should prompt user since this costs tokens to backfill or start from nothing. IF project also have git its also useful for gathering already defined code commits and coding decisions".

1. Scripts gather and digest, at zero tokens:

   | Source | Gives |
   |---|---|
   | Claude Code transcripts of this project (every folder and worktree of the repo, within `cleanupPeriodDays`) | past failures, corrections, decisions; a real "before" baseline of interventions per area |
   | git history, when the project is a git repo: commit messages, reverts and fixups, frequently changed files, commit conventions | coding decisions already made, conventions, active areas, repeated fixes (pitfall hints) |
   | repo structure (`Chart.yaml`, `Dockerfile`, `.github/workflows`, build files, languages) | tech used and areas, so the project index starts populated |
   | native memory and instructions ([Native sources](#native-sources-read-only)) | what is already known |

2. `/selfim` shows what was found and the estimated cost, e.g. "42 past sessions, 380 commits; areas: helm, gitops, java. Bootstrap review ≈ X tokens / $Y", and asks: **bootstrap** or **start from nothing**.
3. On bootstrap, one capped review run turns it into records. Findings that already repeat across past sessions can go straight to trial; git-derived coding decisions start as decisions with `source: observed`.

Starting from nothing skips all of this; the baseline then builds up from new sessions.

## Native sources (read-only)

Claude Code's own memory and instructions are inputs, never outputs (they belong to the user, [06-outputs.md](06-outputs.md#ownership)):

| Native source | Use |
|---|---|
| auto memory (`~/.claude/projects/<project>/memory/`) | counted as already known: selfim doesn't re-learn or re-inject it; overlaps become suggestions |
| `CLAUDE.md`, `AGENTS.md`, `.claude/rules/`, the user's skills | same |
| `InstructionsLoaded` hook | records which native instructions loaded and their size, so the ledger shows selfim's cost next to the native baseline |

Rule: anything already in native memory or instructions is not injected again by selfim; selfim only tracks whether it works and suggests changes.
