# Proposals

Register of everything Claude proposes. Each proposal is logged in the turn it's made and stays here once resolved.

- **Status:** `open` · `accepted` · `rejected` · `changed` (accepted in modified form) · `deferred` (an implementation detail: remembered, not decided at the current stage, D-18)
- **Resolution:** date, the user's words quoted, and where the outcome landed (`D-NN` in DECISIONS.md, or a ✅ part of DESIGN.md / GOALS.md).
- A rejected proposal keeps its reason. It is never deleted.

---

## P-01 · accepted · 2026-10-22 · Proposal register in `docs/PROPOSALS.md`
- **Proposal:** Every proposal gets a P-NN and is resolved here. DECISIONS.md holds agreed decisions only. DESIGN.md marks open content with P-ids inline.
- **Resolution:** 2026-10-22, "add it" → D-17

## P-02 · open · 2026-10-22 · Tickets and knowledge in one store
- **Proposal:** One store, so findings link directly to their ticket, epic, area and tool. (Formerly D-05.)
- **Lands in:** DESIGN §1

## P-03 · deferred · 2026-10-22 · SQLite with full-text search and an append-only event log, not Dolt
- **Proposal:** One machine, a single file, no server. The event log gives history without Dolt. Revisit if branching or multi-machine sync is needed. (Formerly D-07.)
- **Lands in:** DESIGN §1, §5

## P-04 · open · 2026-10-22 · CLI-first; MCP only as a thin wrapper for harnesses without a shell
- **Proposal:** Beads' docs report about 1–2k tokens for CLI + hooks versus 10–50k for MCP schemas, and hooks can't call MCP tools. If MCP is added later, at most 6 tools via `modelcontextprotocol/go-sdk`. (Formerly D-08.)
- **User's stated position:** "an mcp running at global scope that any code cli harness can offload to ... is desired if this improve token and context use"
- **Lands in:** DESIGN §1, §11

## P-05 · open · 2026-10-22 · A "finding" is the candidate state of any record, not its own type
- (Formerly D-10.) **Lands in:** DESIGN §3

## P-06 · open · 2026-10-22 · Review happens at promotion, not at logging
- **Proposal:** Logging is cheap and automatic. The user reviews where knowledge becomes durable or always-on: confirmation, promotion to a skill or CLAUDE.md, merging, retiring. (Formerly D-11.)
- **Lands in:** DESIGN §6, §8

## P-07 · open · 2026-10-22 · Build fresh; existing projects as reference only, don't fork
- **Proposal:** No project covers tickets plus typed knowledge plus budgeted retrieval. Reshaping agent-memory-mcp (232 commits, 41 tools) would cost more than a small new core. (Formerly D-13.)
- **User's direction so far:** "borrow the best technical features from these tools to help us guide our own implementation"
- **Lands in:** DESIGN §10

## P-08 · open · 2026-10-22 · Epic is a scope level
- **Proposal:** The scope chain is ticket → epic → project → global, so a decision in one ticket reaches its sibling tickets. (Formerly D-15.)
- **Lands in:** DESIGN §2

## P-09 · open · 2026-10-22 · Hooks stay deterministic and fast (under about 20 ms, no LLM); only the curator uses an LLM
- (Formerly D-16.) **Lands in:** DESIGN §8

## P-10 · open · 2026-10-22 · Areas are tags from a fixed list per project
- **Proposal:** Areas (git, k8s, helm, java-test, ...) cut across scopes and are the keys for scoping retrieval. A fixed list stops label sprawl.
- **Lands in:** DESIGN §2

## P-11 · deferred · 2026-10-22 · Shared envelope for knowledge records
- **Proposal:** id (type prefix + short hash), kind, scope, state, pin, title (≤100 chars), body, source (observed/user/inferred), verified_at, origin, helpful/harmful/surfaced counters.
- **Lands in:** DESIGN §3

## P-12 · deferred · 2026-10-22 · Fields specific to each kind
- **Proposal:** fact: evidence. limitation: rule, reason, severity block/warn, triggers (cmd regex, path glob, tool, area). decision: question, choice, rationale, alternatives, revisit_when, decided_by.
- **Lands in:** DESIGN §3

## P-13 · deferred · 2026-10-22 · Ticket fields and status machine
- **Proposal:** type, parent, priority, goal, guarded acceptance/out-of-scope, where (repo, dir, branch, target_context, risk), skills load/skip, files_expected, smoke_check, budget. Status: draft → ready → claimed → in_progress ↔ blocked/waiting_human → review → done/dropped.
- **Lands in:** DESIGN §4

## P-14 · deferred · 2026-10-22 · Progress items and evidence
- **Proposal:** progress_item (seq, text, status todo/doing/done/blocked/failed/dropped, note, failed_reason). Evidence (commit/check output/path) attaches to items or records. A commit trailer `Ticket: T-…` creates evidence automatically.
- **Lands in:** DESIGN §4

## P-15 · deferred · 2026-10-22 · Supporting tables; the event log is the source of truth; single write path with secret redaction
- **Proposal:** link, surfacing, session, skill (pointer only), event. Tables are rebuilt from events. Ops are itemized only. Every write does redact → event → rebuild.
- **Lands in:** DESIGN §5

## P-16 · deferred · 2026-10-22 · Write-permission matrix by actor
- **Proposal:** worker / hook / curator / manager / user, each with allowed and forbidden writes. Only the user accepts decisions made by agents.
- **Lands in:** DESIGN §6

## P-17 · deferred · 2026-10-22 · Pickup assembly, ranking and three retrieval layers
- **Proposal:** `pickup --budget` (header, next item, recent log, failed items, scope-chain records matched by pin/trigger, ranked state × source × (helpful − harmful) × recency, deterministic order, `more` line) → `log` → `show`. Includes the example layer-0 bundle.
- **Lands in:** DESIGN §7

## P-18 · deferred · 2026-10-22 · Harness hook actions
- **Proposal:** SessionStart (prime/pickup; reload after compact), UserPromptSubmit (recitation, optional correction capture), PreToolUse (limitation gate), PostToolUse failure (failed-then-succeeded pairs → candidates), PreCompact checkpoint, Stop guard (blocks once), SessionEnd metrics + curator queue, SubagentStop report check.
- **Lands in:** DESIGN §8

## P-19 · deferred · 2026-10-22 · Git hooks
- **Proposal:** post-commit (trailer → evidence + check off item), post-checkout (branch with ticket ID → active ticket), post-merge to main (review → done if DoD passes).
- **Lands in:** DESIGN §8

## P-20 · deferred · 2026-10-22 · CLI transition guards
- **Proposal:** claim (Definition of Ready, WIP=1, atomic), item done (needs evidence), close (acceptance evidence + retro + findings or "none"), block --ask (waiting_human + escalation queue).
- **Lands in:** DESIGN §8

## P-21 · deferred · 2026-10-22 · Scheduled and threshold jobs; auto-confirm rule
- **Proposal:** curator pass, demote/retire, stale facts, skill proposals, widening scope to global, revisit_when matching, budget warnings. Auto-confirm only when observed, seen twice independently, and never harmful.
- **Lands in:** DESIGN §8

## P-22 · deferred · 2026-10-22 · Threshold values
- **Proposal:** recitation every N prompts, retire after 60 days unused, stale after 90 days, skill proposal at K records, curator when inbox ≥ N, Stop guard after X tool calls, budget warnings at 80% / 100%.
- **Lands in:** DESIGN §8

## P-23 · open · 2026-10-22 · Productivity ideas from Jira and similar tools
- **Proposal:** the 16 ideas (Definition of Ready/Done, acceptance as checks, templates, transition guards, WIP=1, waiting-on-human, time-box, triage inbox, dedupe on write, retro, auto-archive, components + fixed labels, saved filters, smart commits, standup entries, cost per ticket by area). Drop story points, sprints, burndown, boards, and detailed priorities (P0–P3 only).
- **Lands in:** DESIGN §9, GOALS non-goals

## P-24 · deferred · 2026-10-22 · Ideas borrowed from similar tools, plus the avoid list
- **Proposal:** the PROJECTMEM, agent-memory-mcp, claude-mem, Backlog.md, xChuCx and Beads ideas in DESIGN §10, and avoiding LLM-per-observation, inject-all, markdown-as-DB, large tool lists, and an embedding dependency in v1.
- **Lands in:** DESIGN §10

## P-25 · deferred · 2026-10-22 · Go libraries
- **Proposal:** `modernc.org/sqlite` (fallback `ncruces/go-sqlite3`), cobra, goose, and `modelcontextprotocol/go-sdk` later.
- **Lands in:** DESIGN §11

## P-26 · open · 2026-10-22 · v1 / later split
- **Proposal:** v1 = building blocks, tickets/progress, event log, CLI + pickup, text ranking, Claude Code + git hooks, guards, curator, metrics, eval. Later = vector search, MCP, other harness adapters, manager/worker, sync.
- **Lands in:** DESIGN §12

## P-27 · deferred · 2026-10-22 · Store location and CLI name
- **Proposal:** `~/.supermem/supermem.db`, with CLI name `tk` (placeholder).
- **Lands in:** DESIGN §1

## P-28 · open · 2026-10-22 · Anti-goals
- **Proposal:** bundle or schemas costing as much as they save, duplicate/stale noise, logging overhead, CLAUDE.md growth moving into the store.
- **Lands in:** GOALS

## P-29 · open · 2026-10-22 · Which proposals are high-level (decide now) and which are deferred
- **Proposal (Claude's sorting under D-18):**
  - **Decide now:** P-02 one store · P-04 CLI-first vs MCP · P-05 finding = candidate state · P-06 review at promotion · P-07 build fresh, no fork · P-08 epic scope · P-09 no LLM in hooks · P-10 areas from a fixed list · P-23 which Jira practices · P-26 v1/later split · P-28 anti-goals
  - **Deferred:** P-03 storage engine · P-11–P-16 record fields, tables, permission matrix · P-17 pickup mechanics · P-18–P-21 hook, guard and job specifics · P-22 threshold values · P-24 borrowed tool features · P-25 Go libraries · P-27 store path and CLI name
- **Borderline:** P-03 (SQLite vs Dolt).

## P-30 · open · 2026-10-22 · System map: 6 views as Mermaid diagrams in `docs/ARCHITECTURE.md`
- **Proposal:** (1) a system map of components, boundaries and interfaces; (2)–(5) four swimlane flows: cold start/pickup, work loop, checkpoint → resume, curation → promotion; (6) lifecycle states for tickets and knowledge records. Later: (7) a manager/worker dispatch flow. Diagrams are Mermaid, kept in git next to the logs; an HTML artifact view is optional for review. Components that rest on open proposals carry their P-id.
- **Lands in:** new `docs/ARCHITECTURE.md` (drafted 2026-10-22, awaiting review)

## P-31 · changed · 2026-10-22 · Diagram tools: Mermaid in the repo, plus an interactive step-through page for the flows
- **Resolution:** 2026-10-22, "go with mermaid for now then" → Mermaid only, for now. The interactive page wasn't taken up. → D-19
- **Proposal:** Mermaid as the source of truth in `docs/` (no tooling, renders on GitHub and in editors, agents write it well). Add an interactive HTML page that steps through each swimlane flow and highlights the active lane and message. Alternatives: D2 (better layout, multi-step boards, needs a CLI to render), LikeC4/Structurizr (model the system once, generate many views), PlantUML (true activity swimlanes, Java dependency, dated look), Excalidraw/draw.io (hand-made, poor diffs).
- **Refines:** P-30 format You raised Dolt yourself, but the engine is an implementation detail once the history/audit need is agreed.
