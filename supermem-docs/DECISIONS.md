# Decisions

Agreed decisions only. Each has a **Basis** quoting the user. Append-only: a change creates a new entry that supersedes the old one.
Proposals that haven't been agreed live in PROPOSALS.md. Gaps in numbering are entries that were moved there.

---

## D-01 · 2026-10-22 · Findings are the core unit; the Jira-like structure exists to track them
- **Basis:** "jira-like is for keeping track of findings which is missing for both hand-off and continuous improment and skill distillation"

## D-02 · 2026-10-22 · CLAUDE.md is fine but not everything belongs there
- **Basis:** "claude is fine but it grows and eats up context always so not everything belong there"

## D-03 · 2026-10-22 · Build our own agent-optimized ticketing system; don't build on Beads
- **Basis:** "building a ticketing system optimized for agents is perhaps a more sound strategy since beads feels like a deadend as it is"
- **Context (Claude's assessment of Beads, not agreed):** `bd remember` is a flat key-value list that `bd prime` injects in full; it's scoped per repo; it has no typed knowledge, no scoped retrieval, and no path from findings to skills.

## D-04 · 2026-10-22 · Single user, this machine
- **Basis:** "start with just for me in this machine"

## D-05 · moved to P-02 (never agreed)

## D-06 · 2026-10-22 · Capture enforced by hooks plus rules
- **Basis:** "using hook in combination with rules to ensure dev and manager always take care of logging of findings, status, facts so they can be used to improve the knowledge base"

## D-07 · moved to P-03 (never agreed)

## D-08 · moved to P-04 (never agreed)

## D-09 · 2026-10-22 · Building blocks: fact, limitation, progress, decision
- **Basis (user's definitions):**
  - facts = "factual findings about systems, tech use for example that is always true for a certain project"
  - limitations = "what we should not do for a project for example never make changes to project-b when working in project-a task"
  - progress = "current work items in a task and status for each etc so it can be resume at any point"
  - decisions: "Yes we need decisions for example for things we decided along the way like for example why we dont use Beads"
- **Deferred by the user:** "if a task is enough or if it can be broken down further is a different story"

## D-10 · moved to P-05 (never agreed)

## D-11 · moved to P-06 (never agreed)

## D-12 · 2026-10-22 · Language: Go
- **Basis:** "go is fine then" (after Claude recommended Go if the user wasn't going to review Rust closely).

## D-13 · moved to P-07 (never agreed)

## D-14 · 2026-10-22 · Adopt ideas A–M from the big-lab research
- **Basis:** "looks good to mae, many points i never have thought of"
- Full list and sources: DESIGN.md §10 "From big-lab research".
- **A** Pin modes always/trigger/search · **B** Itemized edits only · **C** Helpful/harmful counters · **D** Offline curator, workers only add candidates · **E** Field-level write guards · **F** Check before done, smoke-test at pickup · **G** Recitation · **H** Keep failures · **I** Deterministic cache-friendly output · **J** Task-shaped tools, concise/detailed, readable IDs, actionable errors · **K** Worker brief = objective, output format, tools, boundaries · **L** Single-threaded by default · **M** Evaluate from about 20 real handoffs

## D-15 · moved to P-08 (never agreed)

## D-16 · moved to P-09 (never agreed)

## D-17 · 2026-10-22 · Every proposal is logged, tracked and resolved in PROPOSALS.md
- **Basis:** "Everytime you propose something i want it to be logged, tracked and resolved in these document", then "add it" (P-01)

## D-18 · 2026-10-22 · Current stage: high-level decisions only; implementation details are deferred, not decided
- **Basis:** "At this stage we work on high-level decisions not low-level implementation details, those we add to remember not to decide on"
- **Consequence:** implementation-level proposals get status `deferred` in PROPOSALS.md and are revisited when design moves to implementation.

## D-19 · 2026-10-22 · Diagrams in Mermaid, for now
- **Basis:** "go with mermaid for now then" (P-31)
- **Revisit when:** Mermaid's layout can't hold the system map, or an interactive walkthrough is wanted.

## D-20 · 2026-10-22 · ⛔ superseded by D-21 · Skills hold area-wide know-how; procedures go to skills, not records
- **Basis:** "remember we also have skills and will use skills if that covers many of the areas" (in reply to where procedures should live, P-35)
- **Consequence:** no `procedure` record kind. How records relate to skills is in P-35 revision 1.

## D-21 · 2026-10-22 · Records serve two roles: knowledge surfaced at pickup, and raw material for skill distillation
- **Basis:** "remember we also have skills and will use skills if that covers many of the areas", then "What records are for is also to support skill distilation"
- **Supersedes:** D-20. Claude's headline there, "procedures go to skills, not records", was too strong.
- **Consequence:** area-wide know-how, procedures included, is captured as records and distilled into skills. Skills remain the home for area-wide know-how. Still no `procedure` record kind.
