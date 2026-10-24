# Architecture

Status: **draft for review** (P-30). Diagrams in Mermaid (D-19). High-level only (D-18): no field lists or threshold values.

Parts that rest on open proposals carry their P-id:
- one store for tickets and knowledge `P-02`
- CLI as the agents' interface `P-04`
- hooks never call an LLM; only the curator does `P-09`
- the user reviews at promotion, not at logging `P-06`
- epic as a scope level `P-08`

Lanes, in the same order in every flow:
**You · Agent · Hooks (harness) · tk CLI · Store · Git · Curator**

---

## 1. System map

```mermaid
flowchart LR
    you(["You"])

    subgraph harness["Harness · Claude Code now, others later"]
        agent["Agent<br/>(manager / worker)"]
        hooks["Hook adapter"]
    end

    subgraph tk["tk CLI · P-04"]
        prime["prime / pickup"]
        work["log · claim · close"]
        gate["limitation gate"]
        wp["single write path<br/>redact → event"]
        builder["bundle builder<br/>(budgeted, layered)"]
    end

    store[("Store · P-02<br/>event log + current state + search<br/>tickets · progress · facts ·<br/>limitations · decisions")]

    subgraph git["Git repos"]
        repo["commits / branches"]
        ghooks["git hooks"]
    end

    curator["Curator<br/>offline LLM pass · P-09"]

    subgraph outputs["Durable outputs"]
        skills["SKILL.md files"]
        claudemd["CLAUDE.md (small core)"]
    end

    you -- "tickets, answers, reviews" --> agent
    agent -- "CLI calls" --> work
    agent -- "drill-down by ID" --> builder
    hooks -- "lifecycle events" --> prime
    hooks -- "pre-tool check" --> gate
    hooks -- "failures, checkpoints" --> wp
    prime --> builder
    builder -- "query" --> store
    gate -- "match triggers" --> store
    work --> wp
    wp -- "append" --> store
    repo --> ghooks -- "evidence, active ticket" --> wp
    curator -- "read candidates + outcomes" --> store
    curator -- "itemized events" --> wp
    curator -- "proposals" --> you
    you -- "accept / reject · P-06" --> curator
    curator -- "promote" --> skills
    curator -- "promote" --> claudemd
    builder -- "bundle" --> hooks
    hooks -- "inject" --> agent
```

---

## 2. Cold start / pickup

```mermaid
sequenceDiagram
    autonumber
    actor You
    participant Agent
    participant Hooks
    participant tk as tk CLI
    participant Store
    participant Git

    You->>Agent: start session (or work on ticket T)
    Hooks->>tk: session start
    tk->>Git: which repo, branch, cwd?
    Git-->>tk: branch maps to ticket T
    tk->>Store: ticket T + open items + recent log + failed attempts
    tk->>Store: records on scope chain (ticket → epic → project → global)<br/>pinned "always" or matching triggers
    Store-->>tk: candidates for the bundle
    tk-->>Hooks: layer-0 bundle within budget + workflow instructions
    Hooks-->>Agent: inject
    Agent->>tk: show ID (only what it needs)
    Agent->>Agent: run ticket smoke check
    Agent->>tk: claim T
    tk->>Store: event: T claimed
```

---

## 3. Work loop

```mermaid
sequenceDiagram
    autonumber
    participant Agent
    participant Hooks
    participant tk as tk CLI
    participant Store
    participant Git

    loop each tool call
        Agent->>Hooks: about to run a command / edit a file
        Hooks->>tk: check against limitations
        tk->>Store: match triggers
        alt block
            tk-->>Agent: denied + reason (L-id)
        else warn
            tk-->>Agent: one-line warning, proceed
        else no match
            tk-->>Agent: proceed (silent)
        end
        Agent->>Agent: run tool
        opt tool failed
            Hooks->>tk: failure (redacted)
            tk->>Store: candidate finding
            Note over tk,Store: failure followed by a working variant<br/>becomes "use B, not A"
        end
    end

    Agent->>tk: log progress (item done + evidence / new finding / decision)
    tk->>Store: itemized events

    Agent->>Git: commit "Ticket: T"
    Git->>tk: git hook
    tk->>Store: evidence attached, item checked off

    Hooks-->>Agent: periodically recite the next open item
```

---

## 4. Checkpoint → resume (instead of compaction)

```mermaid
sequenceDiagram
    autonumber
    actor You
    participant Agent
    participant Hooks
    participant tk as tk CLI
    participant Store
    participant Curator

    Note over Agent: context filling up, or session ending
    Hooks->>tk: checkpoint (before compact / stop)
    tk->>Store: anything unlogged since last checkpoint?
    alt unlogged work
        tk-->>Agent: log progress first
        Agent->>tk: progress + findings
        tk->>Store: events
    end
    Hooks->>tk: session end
    tk->>Store: session metrics, surfacing outcomes
    tk-->>Curator: queue if inbox is large

    Note over You,Store: later: fresh session or after compaction
    You->>Agent: continue
    Hooks->>tk: session start
    tk->>Store: rebuild bundle for ticket T
    tk-->>Agent: same layer-0 bundle, next open item first
    Agent->>Agent: continue from the next item
```

---

## 5. Curation → promotion

```mermaid
sequenceDiagram
    autonumber
    actor You
    participant Curator
    participant tk as tk CLI
    participant Store

    Note over Curator: runs after sessions or on a schedule
    Curator->>Store: candidates, surfacing outcomes, stale records
    Curator->>tk: de-duplicate, merge, confirm, retire
    tk->>Store: itemized events only (no wholesale rewrite)
    Curator->>Curator: spot areas with enough confirmed knowledge
    Curator->>You: proposals: new or updated skill,<br/>CLAUDE.md line, agent decisions to accept
    alt accept
        You->>Curator: accept
        Curator->>tk: promote
        tk->>Store: records linked to skill / pinned "always"
        Note right of Curator: writes SKILL.md / CLAUDE.md line
    else reject
        You->>Curator: reject + reason
        tk->>Store: kept as rejected (reason retained)
    end
    Note over Store: next pickups surface the confirmed knowledge
```

---

## 6. Lifecycle states

### Ticket

```mermaid
stateDiagram-v2
    [*] --> draft
    draft --> ready: self-contained enough for a cold agent
    ready --> claimed: claim
    claimed --> in_progress
    in_progress --> blocked
    blocked --> in_progress
    in_progress --> waiting_human: question for you
    waiting_human --> in_progress: answered
    in_progress --> review: acceptance has evidence + retro logged
    review --> done
    review --> in_progress: changes needed
    draft --> dropped
    in_progress --> dropped
    done --> [*]
    dropped --> [*]
```

### Knowledge record (fact, limitation)

```mermaid
stateDiagram-v2
    [*] --> candidate: logged by agent, hook or retro
    candidate --> confirmed: curator or you
    candidate --> retired: dropped in triage
    confirmed --> superseded: newer record replaces it
    confirmed --> retired: stale or harmful
    superseded --> [*]
    retired --> [*]
```

### Decision

```mermaid
stateDiagram-v2
    [*] --> proposed: by agent
    [*] --> accepted: by you
    proposed --> accepted: you accept
    proposed --> rejected: you reject (reason kept)
    accepted --> superseded: new decision replaces it
```

---

## 7. Manager / worker dispatch (later)

Not drawn yet. Future use per GOALS.md.
