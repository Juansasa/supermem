# 02 · Capture: how sessions are tracked

## Hooks

The user: "make use of claude hooks to add hooks to relevant hooks to helps the skills capture relevant facts, logs and progress even when skills isnt in play". Capture is always on through hooks; it doesn't depend on the selfim skill being loaded. All hooks are scripts (no LLM). Except for the prompted notes below and the pitfall check, they add nothing to the agent's context.

| Hook | Records | Why it's needed beyond the transcript |
|---|---|---|
| SessionStart | session opened, project ([identity](03-knowledge.md#project-identity-my-call)), git HEAD, what selfim injected and its size | start of the commit range; injected-token cost per session |
| UserPromptSubmit | flag when a prompt looks like a correction | marks likely interventions as they happen; the digest confirms them |
| PreToolUse | pitfall check, warn/block ([06-outputs.md](06-outputs.md#pre-command-pitfall-check)), optional; logs each firing | the only real-time action |
| PostToolUse | failures (tool, command, redacted error excerpt); a success right after a failure becomes a "what fixed it" pair | cheap failure→fix pairing |
| Stop (each turn) | checkpoint: transcript offset captured so far | crashed or killed sessions are still captured; no double-processing |
| SubagentStop | the subagent's transcript path | subagent work lives in separate transcript files |
| PreCompact | checkpoint and queue | long sessions are captured in pieces |
| SessionEnd | queue for review | the normal hand-off. The user: "a hook to run it before closing a session or before compaction is optimal" |
| git post-commit (optional) | commit tagged with the session that made it | ties progress and outcomes to sessions |

Also used: `PermissionDenied` and `PostToolUseFailure` deliver denials and failures directly.

**Hook payloads are the primary source.** The transcript format is internal to Claude Code and changes between versions, so only token usage is read from it, in one tolerant module; if that breaks, capture continues without token stats ([09-stack.md](09-stack.md#capture-sources-hook-payloads-first)). Capture hooks run async; `SessionEnd` only enqueues (1.5 s budget, not guaranteed on crash), and the per-turn `Stop` checkpoint is what makes capture reliable.

Progress mostly comes free: the agent's todo/task tool calls arrive through tool hook payloads and the digest extracts them.

The skill can also be invoked by hand mid-session; that queues the session the same way. Which hook groups are installed is a setup choice ([07-setup.md](07-setup.md#setup-first-invoke)).

## Prompted notes (high precision, when applicable)

The user: "prompt is fine so we can capture useful data with high precision whenever applicable". The transcript records what happened, not always why. When a hook detects a moment worth recording, it injects one short line asking the agent to write a structured note with a small CLI (`selfim note`), so the reasoning is captured by the agent that has it.

| Moment (detected by script) | Hook | Note asked for |
|---|---|---|
| user correction flagged | UserPromptSubmit | what was wrong, the correct rule or fact, its scope |
| failure followed by a fix, not matching a known pitfall | PostToolUse | pitfall: what failed, why, what works instead |
| decision language in the exchange ("let's go with", "decided", "instead of") | Stop | decision: question, choice, reason, what it replaces |
| a fact about a system was verified (probe command succeeded on a target) | PostToolUse | fact: statement, target, stability |

Guards, so prompting stays cheap **(my call, values tunable)**:
- at most one prompt per turn and a per-session cap; never two Stop prompts in a row;
- no prompt when the same note already exists (matched on scope + area + subject);
- a note is one CLI call with fields, not prose.

Notes are session evidence with `source: noted` (higher precision than extracted evidence). Each prompt type is itself measured like a record ([05-lifecycle.md](05-lifecycle.md)): its cost is the tokens of the prompt plus the note; its impact is whether its notes become records that prove themselves. A prompt type that rarely yields useful notes is turned down or off.

## Digest (script, zero tokens)

Before any LLM reads anything, a script reduces the session's hook records, notes and (for token usage) the transcript to a compact digest of the moments that matter:

- failed commands and errors (with the command that fixed them, if any)
- user interventions: corrections, tool-call denials, interrupts, repeated explanations
- decisions and their stated reasons
- tools and skills used; commands per area
- successful sequences (what worked), as raw material for procedures
- git log for the session's commit range
- token usage for the session (from the usage figures in the transcript)
- selfim's own surfacing: what was injected at session start, which `selfim-*` skills loaded, which pitfall warnings/blocks fired and what the agent did next
- selfim's own cost: sizes of everything injected, note calls, and per-hook timings (for the [whole-solution ledger](05-lifecycle.md#whole-solution-ledger))

The LLM reads the digest, never the full transcript. If a needed detail is missing from the payloads, the digest may look it up in the transcript by its stored pointer, tolerating format changes.

## Session evidence

The digests, together with hook records and prompted notes, are the **session-evidence layer**: immutable, stored as JSONL per session under the project it belongs to (`projects/<id>/log/`, or `projects/_unscoped/log/`; see [07-setup.md](07-setup.md#data-placement)). All counting ([05-lifecycle.md](05-lifecycle.md#stats)) comes only from this layer. Knowledge records ([03-knowledge.md](03-knowledge.md)) are curated from it.

## Keeping captured data concise

The user: "ensure our hook capture consise data without irrelevant verbosity". Rules for everything hooks, notes and digests write **(my call; limits tunable)**:

| Rule | Detail |
|---|---|
| **Point, don't copy** | Store the transcript path + message id/offset instead of content. The digest reads the transcript once; hooks never copy messages or tool output. |
| **Fixed, small schema** | One JSONL line per event, short fixed keys, per hook type. Hard cap per line (about 300 bytes); anything longer is truncated, not stored in full. |
| **Successes are counted, not logged** | Successful routine calls (reads, listings, searches, passing commands) become per-area counts in the digest. Only failures, interventions, fixes, decisions and surfacing get individual entries. |
| **Errors as one line** | Keep the first meaningful error line (ANSI stripped, paths shortened, secrets redacted), truncated to about 160 chars. No stack traces or full output. |
| **Commands normalized** | Whitespace collapsed, long arguments and inline data elided (`…`), truncated to about 160 chars. Enough to match a pitfall pattern, no more. |
| **Correction flags carry no prose** | The flag records the matched pattern and the first ~80 chars of the prompt; the digest reads the rest from the transcript if needed. |
| **Repeats collapse** | The same failure (same normalized command + error) within a session becomes one entry with a count. |
| **Notes are fields, not prose** | `selfim note` takes named fields with per-field length limits and rejects anything longer; one note ≤ about 400 chars. |
| **Digest has a budget** | Each session digest is capped (about 1–2k tokens), filled by priority: interventions, then failures and fixes, decisions, notes, surfacing, then counts. Lowest-priority items are dropped first, with a count of what was dropped. |

Log size per session is tracked as part of selfim's own cost ([05-lifecycle.md](05-lifecycle.md#impact-vs-token-cost)); a hook type whose entries are rarely used in review gets trimmed.

## Secrets (my call)

The digest script redacts secrets (known token patterns, values of env vars whose names look secret, `Authorization:`/password-like fields) before anything is written. Raw transcripts are never copied into the repo, only referenced by path. This follows the user's global rule never to log secret values; the repo is meant to be pushable.

## Backfill (my call)

Setup offers an optional, bounded backfill from existing Claude Code transcripts (e.g. the last N sessions per project) to give stats a baseline. Default: off.
