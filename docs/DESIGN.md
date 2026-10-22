# Design

Status: draft.

Legend: ✅ `D-NN` = agreed (DECISIONS.md) · `P-NN` = proposal (PROPOSALS.md), either open or deferred as an implementation detail (D-18). **Content without a ✅ is a proposal.**
Last updated: 2026-10-22

## 1. Overview

✅ Our own agent-optimized ticketing system (D-03), in Go (D-12), with capture enforced by hooks + rules (D-06).

One local store holds **tickets** (work) and **knowledge** (facts, limitations, decisions) `P-02`, in SQLite `P-03` at `~/.supermem/supermem.db` `P-27`. Agents reach it through a CLI, `tk` as a working name `P-04` `P-27`. Hooks call the same CLI to capture data and inject context at lifecycle moments. Retrieval returns a small layer-0 **pickup bundle** with drill-down by ID.

```
agent ──CLI──┐
hooks ──CLI──┼──> single write path (redact → event) ──> SQLite: events + projected tables + FTS
git hooks ───┘                                                │
curator (offline LLM pass) ── itemized events ────────────────┘
```

## 2. Scope

- Chain: **global › project › epic › ticket** `P-08`. Retrieval for a ticket walks ticket → epic → project → global.
- **Areas** (git, k8s, helm, java-test, ...) are tags that cut across scopes. They come from a fixed list per project, which stops label sprawl from breaking retrieval `P-10`.

## 3. Knowledge records (fact, limitation, decision)

✅ The kinds themselves (D-09), pin modes (D-14 A) and helpful/harmful counters (D-14 C).

Shared envelope `P-11`:

| Field | Meaning |
|---|---|
| `id` | short hash with a type prefix: `F-3a1c`, `L-12b0`, `D-01f4` |
| `kind` | fact · limitation · decision |
| `scope` | project, epic?, ticket?, areas[] |
| `state` | candidate → confirmed → superseded / retired. Decisions: proposed → accepted → superseded. A "finding" is the candidate state `P-05`. |
| `pin` | always · trigger · search (D-14 A) |
| `title` | one line, ≤100 chars, the only text shown in layer 0 |
| `body` | detail, shown only on `show` |
| `source` | observed · user · inferred (trust) |
| `verified_at`, `origin_ticket`, `origin_session` | provenance |
| `helpful`, `harmful`, `surfaced_count`, `last_surfaced_at` | outcome feedback (D-14 C) |

Fields specific to each kind `P-12`:

| Kind | Fields |
|---|---|
| fact | `evidence` (command output, path, link) |
| limitation | `rule`, `reason`, `severity: block\|warn`, `triggers[]` (`cmd:` regex, `path:` glob, `tool:`, `area:`) |
| decision | `question`, `choice`, `rationale`, `alternatives[]`, `revisit_when`, `decided_by: user\|agent` |

## 4. Work records

✅ Progress = a task's work items with a status each, resumable at any point (D-09).

**ticket** `P-13`
- id `T-…` / `E-…`, type (epic · feature · bug · spike · chore), parent, priority
- goal, `acceptance[]` 🔒, `out_of_scope[]` 🔒, constraints
- where: repo, dir, branch, `target_context` (kube context/namespace, cloud profile/subscription, DB host), risk (read-only · mutating · destructive)
- `skills_load[]`, `skills_skip[]`, `files_expected[]`
- `smoke_check`, `budget{tokens, turns}`
- status: draft → ready → claimed → in_progress ↔ blocked | waiting_human → review → done | dropped

**progress_item** `P-14`: ticket, seq, text, status (todo · doing · done · blocked · failed · dropped), note, `failed_reason` (dead-ends are kept: D-14 H).

**evidence**: attaches to an item or record. It is a commit sha, check output or path. A commit trailer `Ticket: T-…` creates one automatically.

🔒 = guarded field (§6).

## 5. Supporting tables `P-15`

| Table | Purpose |
|---|---|
| `link` | from, to, type: blocks · parent · discovered_from · relates · supersedes |
| `surfacing` | record, session, ticket, layer, outcome (helpful · harmful · unused). Feeds the counters. |
| `session` | harness, model, ticket, tokens, turns, tool_failures. Cost per ticket and area decides which skills to write first. |
| `skill` | name, path to SKILL.md, areas[], derived_from[record ids]. A pointer only; skills stay normal SKILL.md files. |
| `event` | ts, actor, entity, op, delta. **The source of truth.** Tables are rebuilt from it. `op` ∈ create · update-field · transition · supersede · exclude · merge. There is no wholesale rewrite (D-14 B). |

## 6. Write permissions `P-16` `P-06`

✅ Principle: an offline curator, with workers only adding candidates (D-14 D), and field-level write guards (D-14 E).

| Actor | May | May not |
|---|---|---|
| worker | add progress items, set item status, attach evidence, create candidates, transition its own claimed ticket | edit acceptance or out-of-scope; change confirmed records or accepted decisions |
| hook | create candidates, record surfacing outcomes, write session metrics | anything else |
| curator | confirm, merge, supersede, retire, change pin modes, propose skills | close tickets |
| manager | create and shape tickets, edit guarded fields, set priority | — |
| user | everything. The only actor who can accept `decided_by: agent` decisions. | — |

Every write goes through one path: **redact secrets → append event → rebuild the affected tables**.

## 7. Retrieval: `tk pickup <ticket> [--budget N]` `P-17`

✅ Principles: deterministic, cache-friendly output (D-14 I); task-shaped tools, concise/detailed, readable IDs (D-14 J).

1. Ticket header and the next open item.
2. Last N log events and any `failed` items.
3. Records whose scope is on the chain, where `pin=always` or a trigger matches the ticket's areas, tools or files. Decisions whose `revisit_when` matches are added too.
4. Ranking: `state × source × (helpful − harmful) × recency`, filled by priority until the budget is reached.
5. Deterministic order (kind, score, id), with no timestamps at the top, so the output stays cache-friendly (D-14 I).
6. A `more` line with drill-down commands (`show`, `log`, `epic`).

Three layers: `pickup` (index) → `log` (timeline) → `show <id>` (detail). Output is compact text, with `--json` for scripts and a concise/detailed switch (D-14 J).

Example layer 0 (illustrative, about 350 tokens):

```
T-4f2a  feat  "Add PodDisruptionBudget to api chart"   epic E-91 (3/7 done)
goal    Prevent all api pods evicting during node drain
accept  helm template renders PDB; helm lint clean; ct lint passes
out     no HPA changes; no other charts
where   repo:infra-charts  dir:charts/api  branch:feat/T-4f2a
target  kube:kind-dev ns:api        risk:mutating(local only)
skills  load: authoring-helm        skip: authoring-alerts
next    step 2/4 — add minAvailable to values.yaml
after   T-3c10 (done)   blocks: T-5d77
files   templates/pdb.yaml, values.yaml   (no conflicts)
limits  L-12 defaults in values.yaml, never in templates
        L-31 always pass --kube-context/-n explicitly
facts   F-07 helm 3.15, chart-testing via ct
past    T-2b81 similar PDB on worker chart → see F-19
failed  —
budget  40k tok / 30 turns   model: small
more    tk show <id> | tk log T-4f2a | tk epic E-91
```

## 8. Lifecycle triggers

✅ Hooks + rules enforce capture (D-06).

Rule: hooks are deterministic and fast (under about 20 ms, no LLM). Only the curator uses an LLM `P-09`.

### Harness hooks (Claude Code first) `P-18`

| Event | Action | Writes |
|---|---|---|
| SessionStart `startup` | `tk prime` prints the short workflow instructions, plus the pickup bundle if repo/branch/cwd maps to an active ticket | opens session |
| SessionStart `compact` / `resume` | re-inject the pickup bundle, so compaction becomes a reload from the store | — |
| UserPromptSubmit | every N prompts: recite the next open item (D-14 G). Optional: a pattern match on corrections creates a candidate limitation with `source=user` | candidate |
| PreToolUse | match the command, path or tool against limitation triggers. `block` → deny with the reason and L-id. `warn` → inject one line | surfacing |
| PostToolUse (failure) | log the failure (redacted). A failure followed by a similar successful call becomes a candidate "use B, not A" | candidate, tool_failures |
| PreCompact | checkpoint: if nothing has been logged since the last checkpoint, prompt to log first | event |
| Stop | if the ticket is in_progress and nothing has been logged for X tool calls, block the stop once (never twice in a row) | — |
| SessionEnd | close the session row, work out surfacing outcomes, queue the curator if inbox ≥ N | metrics |
| SubagentStop (worker phase) | the report must reference progress or record IDs | — |

### Git hooks (`tk setup git`) `P-19`

| Event | Action |
|---|---|
| post-commit | `Ticket: T-…` (+ optional `Step: n`) attaches evidence and checks off the item |
| post-checkout | a branch `…/T-xxxx` sets the repo's active ticket |
| post-merge to main | `review → done` if DoD checks pass, otherwise flag it |

### CLI transitions (guards) `P-20`

| Command | Guard |
|---|---|
| `claim` | ticket must be `ready` (Definition of Ready: a cold agent could start from it). WIP=1: an in-progress ticket gets checkpointed first. Atomic. |
| `item done` | needs evidence, or `--no-evidence "reason"`, which is logged |
| `close` | every acceptance check has evidence, a 2–3 line retro exists (it creates candidate limitations), and findings are logged or explicitly "none" |
| `block --ask` | moves to `waiting_human` and joins the escalation queue |

### Scheduled and threshold jobs `P-21` (values: `P-22`)

| Trigger | Job |
|---|---|
| Curator: SessionEnd with inbox ≥ N, or daily (launchd) | small-model LLM pass with a budget: de-duplicate, confirm, merge, drop. Itemized events only. |
| harmful > helpful, or unused for 60 days | demote the pin mode, then retire |
| fact `verified_at` older than 90 days | mark stale (`?` in pickup) and add to the re-verify queue |
| area with ≥ K confirmed records not covered by a skill | propose a skill draft for user review |
| limitation surfaced across ≥ 3 projects | propose widening its scope to global |
| new ticket matches a decision's `revisit_when` | surface that decision in pickup |
| ticket budget at 80% / 100% | 80%: checkpoint and warn. 100%: stop and escalate. |

**Auto-confirm** happens only when `source=observed`, the same candidate was seen twice independently, and none were marked harmful. Agent decisions stay `proposed` until the user accepts them.

## 9. Productivity ideas from Jira and similar tools `P-23`

Definition of Ready and Done, acceptance criteria as checks, minimal templates per ticket type, transition guards, WIP limit of 1, blocked/waiting-on-human with an escalation queue, time-box (token/turn budget), triage inbox, duplicate detection on write, blameless retro on close, auto-archive of stale records, components plus a fixed label list, saved filters (named queries), smart commits, standup-style log entries (done/next/blocked), and actual cost per ticket by area.
Dropped: story points, sprints, burndown charts, detailed priority schemes (P0–P3 only), boards.

## 10. Borrowed ideas

### From similar tools `P-24` (reference only, no fork: `P-07`)

| Idea | Source |
|---|---|
| Append-only event log as source of truth; context rebuilt from it; single write path with secret redaction; `precheck_file` gate; token-budgeted `get_context` | PROJECTMEM (arXiv 2606.12329) |
| Supersession, validity windows, trust fields; Claude Code hook installer (start, end, pre-compact) | ipiton/agent-memory-mcp (Go) |
| Three-layer retrieval (index → timeline → detail) | claude-mem |
| Definition of Done defaults; the tool serves its own agent instructions | Backlog.md |
| confidence: confirmed vs inferred | xChuCx/agent-memory |
| Short hash IDs, ready/claim, link types, `prime` | Beads |

**Avoid:** an LLM processing every observation (claude-mem), injecting all memories at start (Beads), markdown files as the database (Backlog.md), large tool lists (41 tools), depending on an embedding provider in v1.

### From big-lab research · ✅ D-14

| # | Idea | Source |
|---|---|---|
| A | Pin modes always/trigger/search | Devin Knowledge triggers and pinning; OpenHands repo.md vs keyword microagents |
| B | Itemized edits only; avoid "context collapse" from repeated summarizing | ACE (arXiv 2510.04618) |
| C | Helpful/harmful counters | ACE; ExpeL (from memory, unverified) |
| D | Offline curator separate from the worker | Letta sleep-time agents; ACE generator/reflector/curator |
| E | Field-level write guards | Anthropic long-running harness (JSON feature list, only `passes` editable) |
| F | Check before done; smoke-test at start | Anthropic long-running harness |
| G | Recitation | Manus todo.md |
| H | Keep failures | Manus |
| I | Cache-stable deterministic output | Manus KV-cache lesson |
| J | Task-shaped tools, concise/detailed, readable IDs, actionable errors | Anthropic "Writing tools for agents" |
| K | Worker brief: objective, output format, tools, boundaries; outputs to the store | Anthropic multi-agent research system |
| L | Single-threaded by default | Cognition "Don't build multi-agents" |
| M | Evaluate from about 20 real cases, judging the end state | Anthropic multi-agent research system |

OpenAI's Codex ExecPlan sections (Progress, Surprises & Discoveries, Decision Log, Outcomes & Retrospective) map onto progress, candidate, decision and retro. That supports the choice of building blocks.

Sources: see §13.

## 11. Go building blocks `P-25` (✅ Go itself: D-12)

- SQLite: `modernc.org/sqlite` (pure Go, bundles sqlite-vec for later). Fallback: `ncruces/go-sqlite3`.
- MCP (later): `modelcontextprotocol/go-sdk`.
- CLI: cobra. Migrations: goose with embedded SQL. (Claude's call.)

## 12. Phasing `P-26`

**v1:** the four building blocks, tickets and progress, event log and single write path, CLI with compact output and `pickup --budget`, plain-text ranking, Claude Code hooks, git hooks, guarded transitions, curator pass, session metrics, eval with about 20 handoffs.
**Later:** vector search (sqlite-vec), MCP wrapper, Codex/Cursor/Gemini adapters, manager/worker orchestration, multi-machine sync.

## Open questions

Open proposals are tracked in PROPOSALS.md. These are unknowns that aren't proposals yet:

- Check whether full-text search is enabled in `modernc.org/sqlite`'s default build (5-minute spike).
- Gemini CLI pre-tool hook (`BeforeTool`): sources conflict; check against the official docs.
- Which model the curator uses, and its budget.
- Whether a task needs breaking down further (deferred by the user, D-09).

## 13. Sources

- Beads: https://github.com/gastownhall/beads · beads-mcp: https://github.com/gastownhall/beads/tree/main/integrations/beads-mcp · memory docs: https://mintlify.wiki/steveyegge/beads/concepts/memory
- agent-memory-mcp: https://github.com/ipiton/agent-memory-mcp
- PROJECTMEM: https://arxiv.org/html/2606.12329v1
- claude-mem: https://www.mintlify.com/thedotmack/claude-mem/architecture/overview
- Backlog.md: https://cdn.jsdelivr.net/gh/MrLesk/Backlog.md@main/README.md
- xChuCx/agent-memory: https://github.com/xChuCx/agent-memory
- Anthropic long-running harness: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic context engineering: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic multi-agent research: https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic writing tools: https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic code execution with MCP: https://www.anthropic.com/engineering/code-execution-with-mcp
- Claude Code memory: https://code.claude.com/docs/en/memory
- OpenAI ExecPlans: https://developers.openai.com/cookbook/articles/codex_exec_plans
- OpenAI long-horizon Codex: https://developers.openai.com/blog/run-long-horizon-tasks-with-codex
- Manus: https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
- Cognition: https://cognition.com/blog/dont-build-multi-agents
- Devin Knowledge: https://docs.devin.ai/product-guides/knowledge.md
- OpenHands microagents: https://docs.all-hands.dev/usage/prompting/microagents-keyword
- ACE: https://huggingface.co/papers/2510.04618
- Letta sleep-time: https://letta.com/blog/sleep-time-compute
- Hooks comparison: https://lilting.ch/en/articles/gemini-cli-hooks-research
- Codex hooks: https://codex.danielvaughan.com/2026/05/07/codex-cli-v0129-hooks-browser-compaction-hooks-runtime-management/
- Go MCP SDK design: https://go.googlesource.com/tools/+/master/internal/mcp/design/design.md
- modernc sqlite vec: https://pkg.go.dev/modernc.org/sqlite@v1.59.0/vec
