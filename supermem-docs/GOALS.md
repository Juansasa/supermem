# Goals

Last updated: 2026-10-22

Everything here is from the user's own statements, except items marked with a `P-NN` (open proposal, see PROPOSALS.md).

## Intent

Agent sessions keep losing what they learned. Compaction summarizes the current ticket, but by then work has usually moved to another ticket, so the summary has little value, while the findings made along the way are lost:

- tools repeatedly run with the same bad arguments
- commands hit the wrong cluster because the context wasn't checked
- environment facts are discovered and then forgotten

There is also no way to hone working areas (git, gitops, Java testing, ...) into skills that load when needed. Today that knowledge either disappears or becomes always-on rules in CLAUDE.md, which grows and eats context in every session.

supermem is a Jira-like system for agents: projects, epics, tickets, statuses, labels and logs, built to track **findings**. The findings serve three purposes:

1. **Handoff**: a new session picks up a ticket with what it needs.
2. **Continuous improvement**: mistakes and facts stick and resurface when relevant.
3. **Skill distillation**: findings that pile up in one area become skills.

## Goals

- Track decisions, progress/planning, limitations and facts as structured, token-efficient records.
- Make capture automatic and enforced (hooks + rules), not dependent on the agent remembering.
- Store and retrieve relevant information with minimal token use, so an agent can easily offload knowledge and access what it needs when it starts a ticket.
- Replace blind compaction with curated progress that can be added to, trimmed or excluded.
- Distill skills from accumulated findings per area.
- Make tickets self-contained enough that a cold agent can start from one alone.

## Success criteria

1. **Cold start**: a new session on a new task starts with everything it needs, or knows exactly where to find it.
2. **Resume instead of compacting**: when context fills, a fresh session resumes from the store with little or no compaction, because findings were logged as work went on and at hook checkpoints.

## Future use (after the foundation is proven)

- **Manager/worker**: the main session acts as planner/manager only. Worker agents claim tickets, load a `dev/<role>` skill, work like a developer and report back on the ticket.

## Non-goals (for now)

- Multi-user or team sharing. It's single user, on this machine (D-04).
- Story points, sprints, burndown charts, boards. `P-23`

## Anti-goals (failure even if it technically works) · `P-28`

- The pickup bundle or tool schemas grow until loading them costs as much as the context they save.
- The knowledge base fills with duplicate or stale records, so retrieval returns noise and misleads small models.
- Agents spend noticeable effort on logging instead of working.
- CLAUDE.md-style growth just moves into the store, with everything injected at every session start.
