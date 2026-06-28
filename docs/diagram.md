# The Agent Operating Model — One-Page Diagram

> Author: **Eugene Calalang** · 2026-06-23 · Diagrams render natively on GitHub (Mermaid).

## A. Chain of command & roles

```mermaid
flowchart TD
    H["👤 Human Principal<br/>(owns intent)"]
    J["🧭 Judgment / Architect Role<br/>session-bound · decides WITH human · no autonomous decisions"]
    O["🛰️ Orchestrator<br/>the ONLY always-on agent · drains dispatch lane"]
    H --> J
    J --> O

    subgraph WORKERS["Worker roles — summoned on trigger, else not running"]
        direction LR
        R1["Event-triggered<br/>(e.g. review on change)"]
        R2["Anomaly-triggered<br/>(e.g. ops on signal)"]
        R3["On-demand<br/>(specialist skills)"]
        R4["Persistent-while-working<br/>(build / incident owner)"]
    end

    O --> R1 & R2 & R3 & R4
```

## B. Two doors into one persona

```mermaid
flowchart LR
    U["👤 Human"] -->|"direct / interactive door"| P
    ORC["🛰️ Orchestrator"] -->|"orchestrated / ephemeral door"| P
    P["🎭 Persona<br/>(one card + one memory namespace)"]
    P --- M[("🧠 Persona memory<br/>hydrate at spawn · persist at release")]
```

## C. Task lifecycle (durable; process is ephemeral)

```mermaid
stateDiagram-v2
    [*] --> queued
    queued --> dispatched: orchestrator claims
    dispatched --> running: persona spawned
    running --> blocked: needs human/async answer<br/>(save state · release process)
    blocked --> ready: answer arrives<br/>(state re-injected)
    ready --> running: re-spawn
    running --> done: success → persist learnings
    running --> failed: error
    failed --> ready: retry
    failed --> [*]: escalate
    done --> [*]
```

## D. Lane addressing — exactly one consumer per message

```mermaid
flowchart TD
    MSG["✉️ Message"] --> L{"lane / address?"}
    L -->|"dispatch"| ORC["🛰️ Orchestrator ONLY<br/>(only it auto-drains)"]
    L -->|"direct:&lt;session-id&gt;"| SES["💬 One specific live session"]
    L -->|"correlation-id"| TASK["🔗 The exact task it answers"]
    ORC -.->|safety net| C["atomic claim + idempotency<br/>(by task id)"]
    SES -.->|safety net| C
    TASK -.->|safety net| C
```

---

**The eight invariants:** ① one auto-drainer per lane (orchestration may nest) · ② never block a
process on an async/human dependency — park the task · ③ exactly one consumer per message · ④ only
the orchestrator auto-drains dispatch · ⑤ memory & journals belong to persona/task, never the
process · ⑥ engine is generic, identity lives in the card · ⑦ autonomy bounded by an explicit
chain of command · ⑧ supervision is free, spend is budgeted, confidential roles are
residency-pinned.

---

© 2026 Eugene Calalang. All rights reserved.
