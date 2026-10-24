# 07 · Setup and runtime data

## Runtime repo

One global git repo, local, pushable later. The user: "we can keep everything in one global git repo for selfim so it can be kept local and later pushed if needed".

Suggested layout **(my call)**:

```
<selfim-repo>/
  rules.md                 # selfim rule file (capped)
  machine/                 # machine-scope facts
  skills/selfim-<area>/    # area skills (linked into the skills dir where selfim is active)
  projects/<project>/      # project-scope facts, decisions, index
  log/                     # queue + digests (JSONL), redacted
  stats/                   # rebuilt from log/; gitignored, not committed
```

## Setup (first invoke)

The user: "When user import/invoke this skill it should prompt user about where to store its runtime data", and "user decide if they want to add it to their hook skill should prompt this for setup". On first invoke, the skill asks, one prompt each, and records the answers in the repo:

1. **Where to store runtime data** (creates and `git init`s the repo).
2. **Which hook groups to install** (each shown with what it does; see [02-capture.md](02-capture.md#hooks)):
   - **core capture:** SessionStart (incl. loading), Stop checkpoint, SubagentStop, PreCompact, SessionEnd. Without it, capture runs only when the skill is invoked by hand.
   - **signals:** correction flags (UserPromptSubmit), failure/fix pairs (PostToolUse).
   - **prompted notes:** the agent is asked for a structured note at moments worth recording ([02-capture.md](02-capture.md#prompted-notes-high-precision-when-applicable)).
   - **pitfall check:** warn/block before commands ([06-outputs.md](06-outputs.md#pre-command-pitfall-check)).
   - **git:** post-commit hook tagging commits with their session, optional.
3. **Where selfim is active:** this project only, or every session on the machine. This scopes the hooks **and** where the `selfim-*` skills are linked, so their descriptions don't load in sessions where selfim is off. Suggested default: this project only **(my call)**.
4. **What starts the review step:** by hand (`/selfim review`), with a one-line reminder from the session-start hook when the queue passes a threshold; or automatically in the background after a session ends. Suggested default: by hand **(my call)**, since background runs spend tokens unattended. Either way, once a review runs its changes apply automatically; with the by-hand default, nothing improves until a review is started.
5. **Backfill** from existing transcripts ([02-capture.md](02-capture.md#backfill-my-call)). Default: off.

Every answer can be changed later by re-running setup.

## Where the source lives

Design and implementation of the skill live in `supermem/self-improvements/`. supermem's own docs (`supermem-docs/`) are for a different system and don't apply here.
