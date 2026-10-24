# 10 · Robustness, safety and recovery

How selfim behaves when things go wrong. Found by simulating failures, concurrency and long-term use. All choices here are **my call** unless quoted; the user: "I think we'll have to define each area better during development and testing", so this file is a starting contract, refined while building.

## Never break the user's session

- **Fail open.** Every hook runs inside a top-level guard that always exits 0 (never "block" by accident) and logs the error to `.cache/errors`. Only the pitfall check may deny a call, and only on purpose.
- **Short timeouts** are set explicitly on every hook; a slow hook is skipped, not waited for.
- **A launcher shim** finds a usable Python ≥3.9 once and caches the result. On macOS it checks the developer tools are installed before touching `/usr/bin/python3`, so no "install developer tools" dialog ever appears. If no Python is found, hooks do nothing and `/selfim` explains why.
- **Missing data means disabled.** If the runtime repo or `registry.json` is missing, hooks treat every project as disabled and never create paths.
- **Version skew.** If data has a newer `schema_version` than the installed plugin (downgrade, or an older plugin on another machine), selfim goes read-only: it still injects, but writes and commits nothing, and `/selfim` says so.
- **User-facing messages** (hints, reminders, "learned" summaries) go to the user through the hook's user-visible message channel, not into the agent's context. The install hint shows once per repo.

## Capture health

- **Atomic appends.** Each event is one single write of ≤300 bytes in append mode, so concurrent writers (sync/async hook parts, subagents, two windows) don't interleave. Readers skip lines that don't parse and count them.
- **Spool, then commit.** Hooks write to an untracked spool (`.cache/spool/<session_id>.jsonl`). When a session is digested, its log moves to `projects/<id>/log/` and only then is it committed. The git work tree is never changed by live sessions, so learning, pulls and rebases always see a clean tree.
- **Liveness.** Each live session keeps a heartbeat (`.cache/live/<session_id>`, pid + timestamp). The catch-up sweep only takes sessions whose process is gone or whose heartbeat is older than T, so a session open in another window is never taken for a killed one.
- **Disk full or write error:** the event is dropped and counted; the session continues.
- **Missing events.** Per session, selfim counts events per hook type. If an expected type never arrives (a hook renamed or a payload field removed by a Claude Code update), the session is marked **unmeasured** and excluded from trial statistics, so silence never looks like "fewer corrections". After a Claude Code version change, trials pause until a session shows every expected event type firing.
- **Heavy work stays out of hooks.** Digests run in a detached background process (started at SessionEnd, PreCompact or by the sweep), never inline in a hook.

## Concurrency

- **One lock** (`.cache/selfim.lock`) around every commit, render, registry write and migration.
- **Single-flight learning.** Only one learning run at a time. A request arriving during a run sets a flag, and one follow-up run happens afterwards. Automatic learning also has a daily spending cap.
- **Atomic file writes** (write to a temp file, then rename) for `registry.json`, `project.json` and records.
- **Stale git locks** (`index.lock`) are cleared only when no git process is running.
- **Commit attribution:** commits are deduplicated by sha per project, and attributed to the session whose tool call ran `git commit`, so two windows on one repo don't both claim the same commits.

## Learning runs

- **Batches.** Learning from a large backlog runs in batches (e.g. 20 sessions or 200 commits), newest first, each with its own cursor commit, so hitting the spending cap loses at most one batch. If the estimate exceeds the cap, `/selfim` offers "learn the most recent part now, the rest later". The estimate notes that only transcripts within `cleanupPeriodDays` still exist.
- **All or nothing per batch.** Changes are applied on a temporary branch, checked, and fast-forwarded only if every check passes; otherwise the branch is discarded. Cursors advance only in the commit that consumes the digests.
- **Failures are visible.** Each run's outcome (ok, not logged in, rate limited, budget hit, invalid output) is recorded in `reviews/` and shown in `/selfim` and as a one-line message at the next session start. Rate limits retry with backoff.
- **The path guard is configured explicitly** for the learning run (not inherited from user settings), and tested to fire.

## Prompt injection

The user: "selfim is interested in workflow and processes not specifically what input were used". So:

- **Capture is content-free.** selfim stores the shape of work (command families, tools, outcomes, sequences, failure→fix pairs) and the user's own corrections and decisions, but not tool output, file contents or argument values ([02-capture.md](02-capture.md#what-selfim-records)). A learning run therefore sees almost no untrusted text.
- **The remaining outside text** is commit messages and repo files (used when learning from history). Records derived from them need the user's approval before they load in every session, go global, or become blocking ([04-review.md](04-review.md#approval-gate)).
- **Lint before render.** Rules and pitfall "instead" lines are checked by a script; anything recommending destructive or exfiltrating actions (`rm -rf`, `curl | sh`, force pushes, permission or hook changes) is rejected.
- **New always-loaded text is visible:** the session-start header names records added since the last session.
- **Pasted text isn't a user decision.** A decision flagged from a long or quoted prompt doesn't go on trial at once.

## Pitfall safety

- **Safe triggers.** Trigger patterns are limited to a safe subset (literal tokens, anchors, alternation, no nested quantifiers), validated at render time, and matched under a time budget; a trigger that overruns is allowed through and quarantined.
- **Override.** A block message names a one-shot override (`selfim allow <id>`, allowed without a permission prompt). Using it lets the call through, marks the record harmful, and takes effect at once. `selfim pause` turns the pitfall check off for the rest of the session.
- **Blocks expire** when a fact they depend on is contradicted.

## Secrets

- Redaction runs in every hook and in `selfim note` before anything is written ([02-capture.md](02-capture.md#secrets-my-call)).
- **Scan before commit:** every selfim commit scans its staged content; a hit blocks the commit.
- **Push is opt-in**, and requires a clean scan of everything not yet pushed.
- **Purge:** `selfim purge <pattern>` is the one documented exception to "nothing is deleted". It rewrites the selfim repo's history, repairs cursors from the logs, and tells the user to rotate the secret and force-push any remote.

## Two machines and history rewrites

- **Merge rules per file type:** JSONL logs merge by union; records resolve by state precedence, then the later change; cursors take the maximum; `registry.json` takes the union.
- An unresolvable conflict aborts the pull, keeps local commits, and shows in `/selfim`.
- A rewritten upstream (non-fast-forward) is never rebased onto automatically.
- Cursors can always be rebuilt from the logs.

## Hand edits

- Dirty record files are committed before learning as `selfim-user-edit` and treated as explicit user statements.
- Rendered files carry a "generated from <record ids>; edit the record" header and a content hash; if a rendered file was edited, `/selfim` warns instead of overwriting.
- A user commit that touches records without selfim trailers is treated as a manual revert or pin.
- If the user deletes the project-skill link, that is recorded as an opt-out and the link isn't recreated.

## Growth

Compacting evidence doesn't shrink git history. `/selfim` reports the repo size; an archive command can move old history to a separate repo when it grows too large.

## Uninstall

`/selfim uninstall` removes the skill links, the `.git/info/exclude` lines and the `selfim note`/`selfim allow` permission rules, then tells the user to remove the plugin. The runtime repo stays. Its location is also stored in `~/.config/selfim/location`, so a reinstall finds the existing repo instead of creating a new one.
