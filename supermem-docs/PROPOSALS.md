# Proposals

Register of everything Claude proposes. Each proposal is logged in the turn it's made and stays here once resolved.

- **Status:** `open` · `accepted` · `rejected` · `changed` (accepted in modified form) · `withdrawn` (replaced by Claude before the user resolved it) · `deferred` (an implementation detail: remembered, not decided at the current stage, D-18)
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

## P-32 · open · 2026-10-22 · Add three missing connections to the system map
- **Proposal:** the CLI reads git at session start (repo/branch → ticket); the CLI queues the curator; the harness loads SKILL.md files into the agent, which closes the improvement loop.
- **Lands in:** ARCHITECTURE §1

## P-33 · withdrawn · 2026-10-22 · Entity catalog for the data layer (entities only; fields deferred per D-18)
- **Proposal:** 10 core entities, most named by the user (project, epic, ticket, progress item, log entry, fact, limitation, decision, area, skill), plus 5 supporting entities from Claude (evidence, link, session, surfacing, event). Relationships are drawn as an entity diagram without fields.
- **Lands in:** new DESIGN section "Data layer: entities"
- **Withdrawn:** 2026-10-22. The user asked to check for overlap ("see if they're the minimal list of things we need so no overlap or unnecessary"); replaced by P-34.

## P-34 · open · 2026-10-22 · Minimal data model: 4 entities + 1 relation
- **Proposal:**
  - **Work item** (kinds project · epic · ticket · step) as one tree. Scope = any node; global = no node. This covers P-08, and decomposition becomes "add children".
  - **Record** (kinds fact · limitation · decision). The D-09 building blocks are kept as kinds; a finding is the candidate state.
  - **Area**, with an optional pointer to its skill.
  - **Event** (change · note · evidence · surfaced · session), append-only. Covers logs, trim/exclude, evidence, sessions/cost and helpful/harmful.
  - **Link** relation: blocks · relates · supersedes · derived-from.
- **Merged away:** epic, project, progress item → work item; log entry, evidence, session, surfacing → event; skill → area.
- **Costs:** Event is a catch-all and needs derived views. Steps share the work-item entity with tickets. Assumes one skill per area.
- **Lands in:** new DESIGN section "Data layer: entities"; would replace the field-level sketches in DESIGN §3–§5 (P-11–P-15, deferred).
- **Revision 1 (2026-10-22):** the user asked "isnt Area just labels ?". Yes: **Area is dropped and becomes labels**, a plain property on work items and records. The vocabulary is controlled by normalizing on write plus a curator merge (detail deferred). A SKILL.md declares which labels it covers, instead of an area pointing to its skill. Model is now **3 entities + 1 relation**: work item, record, event, plus link. Added risk: label sprawl hurts retrieval. This also changes P-10 ("areas from a fixed list") to "labels with normalization".

## P-35 · open · 2026-10-22 · Session context coverage: fact/limitation/decision + event + link is enough, under two conditions
- **Context:** the user asked "are these fact · limitation · decision enough to capture a task session context together with link and event ?"
- **Proposal:**
  1. **Facts can be unverified.** Hypotheses and assumptions are facts in the candidate state with `source: inferred`.
  2. **Decisions can be open.** An open question is a decision with a question and no choice yet; answering it fills in the choice.
  3. **Procedures (multi-step how-tos) are stored as facts for now** and distilled into skills by the curator. Revisit with a fourth record kind `procedure` if facts and procedures get confused.
- **Revision 1 (2026-10-22):** point 3 replaced after the user said "remember we also have skills and will use skills if that covers many of the areas" (D-20). Procedures go to **skills**, not facts. A procedure found mid-session is captured as an event note with labels, and the curator updates or proposes the matching skill. Further proposals:
  - **Split rule:** area-general know-how → skill; project- or ticket-specific knowledge → record.
  - **No duplication:** records distilled into a skill are linked to it and no longer surfaced separately; pickup points to the skill.
  - **Promotion:** a limitation seen across projects in an area moves into that area's skill.
  - **Skill index:** skills stay as SKILL.md files declaring their labels; the store only reads an index of them.
- **Revision 2 (2026-10-22):** after "What records are for is also to support skill distilation" (D-21), the capture path in revision 1 changes. Area-wide findings and procedures are captured as **records** (candidates with labels), **not event notes**. Further proposals:
  - Distilled records stay **linked to their skill** (`derived-from`) as its provenance and evidence. They're kept in the store but not surfaced separately; pickup points to the skill.
  - The curator **re-distills a skill** when its supporting records are superseded, marked harmful, or new ones pile up.
  - Records may need a **"distilled" marker or state**, so retrieval knows to skip them (exact form deferred).
- **Coverage walk-through:** goal/plan/status → work item; output, evidence and "where was I" → event; stable truths → fact; mid-session instructions and failed attempts → limitation (ticket scope); choices → decision; discovered work → work item + `discovered-from` link.

## P-36 · open · 2026-10-22 · Capture successful trajectories for skill distillation
- **Context:** the user asked "would all our logs and events etc be enough for distilation of skill ?"
- **Gap:** failures are captured automatically but successes aren't, and a skill is mostly the successful path.
- **Proposal:** when a step is marked done, keep a compact trajectory of what worked: the commands and edits between starting and finishing the step, redacted and de-duplicated, without output. Distillation uses successful trajectories for the procedure, failures and limitations for pitfalls, facts for conventions, decisions for rationale. A pattern becomes a skill candidate only once it recurs across two or more tickets or projects (sources: Agent Workflow Memory, Voyager).
- **Open:** should skill distillation be its own "skill agent", separate from the curator's record triage? (The user used the term "skill agent".)

## P-37 · open · 2026-10-22 · Label inference from activity, and when a skill becomes relevant
- **Context:** the user asked "how would it know a git skill for example is now relevant ?"
- **Proposal:**
  - **Automatic labels:** deterministic rules map commands and paths to labels (`git`/`gh` → git, `kubectl` → k8s, `pom.xml`/`*.java`/`mvn` → java, `charts/**`/`helm` → helm). Events and records created during that activity inherit the label, without relying on the agent to tag them.
  - **At distillation:** the curator looks for clusters of confirmed records plus successful trajectories under one label, across two or more tickets. Labels with high cost or many failures go first.
  - **At use:** skills are listed in pickup from ticket labels and repo facts. The pre-tool hook also gives a just-in-time one-line nudge ("skill X covers this") on the first matching command.
  - **Feedback:** skills get helpful/harmful counts when surfaced. A skill that doesn't reduce failures in its area gets flagged for re-distillation.

## P-38 · open · 2026-10-22 · Skills start at planning as drafts; work confirms them
- **Context:** the user asked "So a skill can be build at planning stage if done right ?"
- **Proposal:**
  - **Skill coverage check at planning:** the manager compares the labels the tickets need against existing skills to get a skill gap list.
  - **Draft skills for gaps,** seeded from same-label records in other projects, official docs or public skills, and this project's decisions. They start small, marked unverified, and are loaded at lower trust.
  - **Skill-forming tickets:** the first ticket in an uncovered area gets richer capture (P-36) and a skill pass right after it closes.
  - **Confirmation by outcomes:** matching successful trajectories, helpful counts, fewer failures. Distillation refines the draft with itemized updates, never a rewrite.
- **Lifecycle:** plan (draft) → work (evidence) → distill (confirmed) → use (feedback, re-distill on drift).
- **Risk:** generic doc-derived content is the noise the anti-goals warn about (P-28); hence keep drafts small and confirm them by evidence.

## P-39 · open · 2026-10-22 · Atomicity rules for work items; steps are the atomic unit
- **Context:** the user asked "Are suggested workitem atomic ?" P-34 defined the entity but not how big an item may be.
- **Proposal:**
  - **Project and epic** are containers, not atomic.
  - **A ticket** is atomic for claiming and acceptance (one owner, one deliverable), but not for execution; it can span sessions.
  - **A step** is atomic. It has one outcome stated as a result, one check with evidence, fits in one session, has no hidden sub-steps, and ideally maps to one commit.
  - **Decomposition rule:** break down until each step fits in one session and has one check. A step that doesn't finish in one session gets split at the checkpoint. This answers the user's deferred question (D-09) with a rule.
  - **Definition of Ready** checks that steps meet the criteria.
- **Effects:** resume at step granularity, with only the in-flight step needing a "where was I" note; trajectories (P-36) align with steps.
- **Tension:** this strengthens the case for steps as a separate lighter entity (P-34's open one-tree question).
- **Revision 1 (2026-10-22):** after the user asked "what is a step ? a broken down task ?", a step is defined as **one entry in a ticket's plan, not a task**. It is the user's own "work items in a task and status for each" (D-09, progress). Unlike a sub-task (child ticket), a step has no separate owner, no own acceptance criteria, can't be done in parallel by another agent, and has no steps of its own. Rule of thumb: could it be handed to another agent and done on its own? Then it's a sub-ticket; otherwise a step.
- **Proposed consequence for P-34:** keep steps as their own lighter entity ("progress item"), not a fourth work-item kind. The tree is project › epic › ticket (tickets can have sub-tickets), and each ticket holds its steps. The model becomes 4 entities + link: work item, progress item, record, event.
- **Revision 2 (2026-10-22):** "ideally maps to one commit" is too strict (see P-40). A code-changing step ends in **at least one** commit tagged with its step. A step with no code change carries evidence only.

## P-40 · open · 2026-10-22 · Align work items with code delivery; git drives state and evidence
- **Context:** the user asked "then it doesnt align well with code delivery no ?"
- **Proposal:**
  - **Mapping:** epic ↔ milestone/release · ticket ↔ one branch → one PR (PR-sized, checked by the Definition of Ready) · sub-ticket ↔ its own PR · code-changing step ↔ one or more commits tagged `Ticket: T · Step: n` · step with no code change ↔ evidence only.
  - **Git and PR events drive ticket state:** branch pushed → in_progress; PR opened → review; CI green → acceptance evidence; merged → done; closed unmerged → in_progress or dropped.
  - **Tickets spanning repos:** one ticket per repo/PR, grouped under a parent ticket or epic, ordered with `blocks` links.
  - **Stacked PRs:** sub-tickets plus `blocks` links for now.
  - **Work with no PR** (`helm upgrade`, `kubectl apply`): the ticket closes on evidence, not merge. With GitOps it becomes a PR.
- **Principle:** planning (epic › ticket › step) stays agent-friendly; delivery (release › PR › commit) is the source of truth for state and evidence.

## P-41 · open · 2026-10-22 · Steps are a checklist on the ticket, not an entity
- **Context:** the user asked "Why is step needed ?"
- **Why the concept is needed:** D-09 (progress = "work items in a task and status for each … resume at any point"), and the user's handoff scenario ("done steps and remaining steps"). Without a structured list of remaining work, handoff falls back to free-text notes.
- **Options:** (1) no steps: progress = event notes + commits + acceptance checks; (2) **an ordered checklist of `{text, status}` on the ticket**, every change an event, and evidence/trajectories as events pointing to item n; (3) a separate progress-item entity.
- **Proposal:** option 2. A step never exists outside its ticket, has the ticket's owner, and has no relations of its own, so it's a property of the ticket.
- **Consequences:** the model goes back to **3 entities + link** (work item, record, event). The tree is project › epic › ticket (› sub-ticket). This replaces P-39 revision 1's "separate progress-item entity" and settles P-34's one-tree question. The atomicity criteria in P-39 and the delivery mapping in P-40 still apply to checklist items.

## P-31 · changed · 2026-10-22 · Diagram tools: Mermaid in the repo, plus an interactive step-through page for the flows
- **Resolution:** 2026-10-22, "go with mermaid for now then" → Mermaid only, for now. The interactive page wasn't taken up. → D-19
- **Proposal:** Mermaid as the source of truth in `docs/` (no tooling, renders on GitHub and in editors, agents write it well). Add an interactive HTML page that steps through each swimlane flow and highlights the active lane and message. Alternatives: D2 (better layout, multi-step boards, needs a CLI to render), LikeC4/Structurizr (model the system once, generate many views), PlantUML (true activity swimlanes, Java dependency, dated look), Excalidraw/draw.io (hand-made, poor diffs).
- **Refines:** P-30 format You raised Dolt yourself, but the engine is an implementation detail once the history/audit need is agreed.
