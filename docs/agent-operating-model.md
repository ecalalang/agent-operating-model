# The Agent Operating Model

> A vendor-neutral framework for running AI agents as a managed, accountable workforce.
>
> Author: **Eugene Calalang** · First captured: **2026-06-23** · Status: Draft v0.1

---

## 1. Premise — the agent as an employee

A useful agent is not a function you call; it is a **worker you employ**. The shift from
"call a model" to "employ an agent" reframes the entire problem. Functions are stateless,
anonymous, and fire-and-forget. Employees have identity, memory, assignments, the ability
to escalate, and accountability. The Agent Operating Model (AOM) takes the employee
metaphor seriously and shows that it maps **one-to-one** onto concrete technical
primitives. The metaphor is not decoration — it is the design.

## 2. The employment mapping

Every concept a human organisation needs to coordinate workers has a precise technical
counterpart in the AOM:

| Employment concept | AOM primitive |
|---|---|
| Job description | **Persona card** — role, skills, tools, model/cost budget |
| Training that sticks | **Learning loop** — durable persona memory loaded at start, written at end |
| A manager assigns work | **Dispatch** — the orchestrator hands a task to a role |
| Asking the manager a question | **Parked task** — async escalation without blocking a process |
| No two people grab the same ticket | **Claim + lane addressing** — exactly one consumer per message |
| "What did you do on this?" / handover notes | **Working journal** — per-task play-by-play |
| Talk to them live, *or* let them work async | **Two doors** — direct session and orchestrated spawn |
| Reports to a boss, who reports up | **Chain of command** — bounded autonomy |

The completeness of this mapping is itself evidence the model is sound: a real worker needs
all eight, and all eight fall out of asking "but what happens when…?".

## 3. Roles, triggers, and tiers

### 3.1 One always-on auto-drainer per lane
Exactly **one** orchestrator persistently watches each **dispatch lane** and assigns its work.
Everyone else is **summoned on a trigger** and is otherwise not running. The invariant is one
auto-drainer **per lane**, not one per system: a simple crew runs a single orchestrator, while a
larger crew **nests** — a sub-crew may run its own orchestrator over its own lane, queue, and
polling cadence (e.g. a fast operational loop beneath a slower strategic one). Within any one lane
there is still a **single dispatcher, not a swarm of pollers** competing for the same queue. This
keeps cost bounded and coordination deterministic while letting orchestration scale hierarchically.

### 3.2 Trigger taxonomy
Roles are woken by *event type*, not kept idling:

- **Event-triggered** — e.g. a review role woken by a change/PR event.
- **Anomaly-triggered** — e.g. an operations role woken by an infrastructure signal.
- **On-demand** — most specialist roles, spawned when their skill is needed.
- **Persistent-while-working** — a role stays resident *only* for the duration of a long
  task it owns (a build, an incident), then releases. Persistence is a property of an
  active task, not of the role.
- **Session-bound by design** — a **judgment/architect role** that must never decide
  autonomously. It exists only inside a human-coupled session, because its job is to loop
  decisions through a human. This is a deliberate constraint, not a limitation.

### 3.3 Tiers
The model supports two deployment tiers that share the same primitives:

- **Unmanaged tier** — no always-on orchestrator; roles are invoked directly by a human or
  a host runtime. Lighter, simpler, no infrastructure.
- **Managed tier** — a resident orchestrator drives dispatch, claims, and journals. Heavier,
  but enables autonomous, durable, multi-step work.

A crew can graduate from unmanaged to managed without changing its persona cards or
learning loops.

## 4. The nine primitives

### 4.1 Persona card (job description)
A declarative definition of a role: its name, the skills/tools it may use, its model and
cost budget, its trigger, and its memory namespace. A persona is a *card the runtime
wears* — the runtime engine stays generic; identity lives in the card. This separates the
**engine** (reusable) from the **employee** (configured).

### 4.2 Learning loop (training that sticks)
Each persona owns a durable memory namespace. At spawn it **hydrates** (loads prior
learnings); at release it **persists** (writes new learnings). Learning is attached to the
*persona*, not to the ephemeral process that happens to be running it — so a lesson learned
by one instance is available to the next. Without this, every spawn is a new hire with
amnesia.

### 4.3 Dispatch (assignment)
The orchestrator is the single component that converts incoming work into an assignment: it
selects the right persona, opens a task, and spawns (or attaches to) an instance. Because
dispatch is centralised, there is exactly one decision-maker for "who does this," which is
what prevents double-action at the source.

### 4.4 Parked task (async escalation)
**Rule: never block a live process on a human or asynchronous dependency.** When a worker
needs an answer it cannot get immediately, it does *not* sit and wait. It:
1. writes its current working state to the task,
2. marks the task `blocked` (recording *what* it is waiting on and *who* owes the answer),
3. **releases the process.**
The answer, when it arrives, flips the task `blocked → ready` and **re-spawns** the
persona with its saved state injected. The *process* is ephemeral; the *task* is durable
and resumable. This is how an agent "asks a question and comes back to it" without holding
a process hostage.

### 4.5 Claim + lane addressing (no double-work)
Two mechanisms guarantee **exactly one consumer** per unit of work:

- **Lane addressing (primary).** Every message carries an address:
  - `dispatch` — consumable only by the orchestrator;
  - `direct:<session-id>` — destined for one specific live session;
  - a `correlation-id` — a reply keyed to the exact task it answers.
  Addressing makes the consumer *deterministic by construction*.
- **Atomic claim (safety net).** Before working an item, a worker performs a conditional
  claim (`set state = claimed where id = ? and state = unclaimed`) and proceeds only if it
  won the row. Combined with **idempotency keyed by the task id**, a duplicated delivery
  cannot cause duplicated work.

A critical corollary: **only the orchestrator auto-drains the dispatch lane.** Direct,
human-driven sessions must *not* scrape the shared queue — if every instance drains it, you
manufacture the very race you are trying to prevent.

### 4.6 Working journal (handover / context continuity)
Context is reconstructed from **three layers**, all hydrated at spawn:
1. **Persona memory** — long-term learnings (cross-task).
2. **Per-task working journal** — a play-by-play of *this* task, keyed by task id. Workers
   must journal their *working context as they go*, not just a final summary.
3. **Conversation thread** — the message history for the task.
Because of (2), any later instance — or a human who opens the task directly — can
reconstruct "what happened so far." Handover is automatic, not heroic.

### 4.7 Two doors (direct + orchestrated)
The same persona can be entered two ways:
- **Direct / interactive** — a human opens the persona and works with it live.
- **Orchestrated / ephemeral** — the orchestrator spawns it to run a dispatched task.
Both doors open onto the **same card and the same memory namespace.** A direct session is
simply "an ephemeral spawn that a human drives and keeps open." Take-over is supported: a
human can attach to an in-flight task; the autonomous worker yields; single-flight is
preserved and context transfers through the journal.

### 4.8 Chain of command (bounded autonomy)
Authority is explicit and layered: **Human principal → judgment/architect role →
orchestrator → worker roles.** Workers execute; the orchestrator coordinates; the judgment
role designs and decides *with* the human; the human owns intent. No layer exceeds its
mandate. Autonomy is a dial set per role, not a default.

### 4.9 Cost & residency governance (economic accountability)
An employed workforce has a budget and a confidentiality boundary; so does an agent crew. Three
contracts make the economics and data-residency of the crew first-class, not incidental:

- **Zero-cost supervision.** The always-on layer — the orchestrator/auto-drainer — must impose
  **no per-decision model cost.** Watching for work is mechanical: either a model-less daemon, or a
  model-backed role that idles on a non-billed wait and only consumes budget when actual work
  arrives. *Who is awake must be cheap to keep awake.*
- **Per-role spend budget, enforced.** Each persona card declares a model/cost tier
  (e.g. `light`/`heavy`/`maximum`). The tier is **resolved from configuration, never hardcoded**,
  and the highest tier requires explicit human approval per task. Routine roles run on the cheapest
  capable model; scarce expensive capacity is spent only where it changes the outcome.
- **Residency pin with a hard canary.** A role that handles confidential data is **pinned to a
  residency boundary** (e.g. on-device / local inference). The dispatcher must **fail closed** —
  refuse, not warn — when routing such a role would cross that boundary. Residency is an invariant
  of the card, enforced at dispatch, not a guideline.

## 5. Task lifecycle

A task is the durable unit of work. Its state machine:

```
queued ──▶ dispatched ──▶ running ──▶ done
                              │
                              ├──▶ blocked(awaiting:<who>) ──▶ ready ──▶ running
                              │            (answer arrives, state re-injected)
                              └──▶ failed ──▶ (retry | escalate)
```

- **queued** — created, addressed to the `dispatch` lane.
- **dispatched** — claimed by the orchestrator, persona selected.
- **running** — an instance is live and working; it journals as it goes.
- **blocked** — parked on a dependency; process released, state saved.
- **ready** — dependency satisfied; eligible for re-spawn.
- **done / failed** — terminal; learnings persisted to the persona.

The process may start and stop many times; the *task* persists across all of them.

## 6. Design invariants

1. **One auto-drainer per lane** — each dispatch lane has exactly one always-on orchestrator;
   orchestration may nest (a sub-crew owns its own orchestrator, queue, and cadence). Everyone
   else is triggered.
2. **Never block a process on an async/human dependency** — park the task instead.
3. **Exactly one consumer per message** — by address first, claim second, idempotency third.
4. **Only the orchestrator auto-drains the dispatch lane.**
5. **Memory and journals belong to the persona/task, never the process.**
6. **Engine is generic; identity lives in the card.**
7. **Autonomy is bounded by an explicit chain of command.**
8. **Supervision is free; spend is budgeted; confidential roles are residency-pinned.** The
   always-on layer costs nothing per decision; each role's model spend is declared and enforced
   from configuration (top tier human-gated); and a confidential role fails closed rather than
   leave its residency boundary.

Hold these eight invariants and a crew of agents behaves like a well-run team rather than a
race condition.

## 7. Relationship to existing fields

This model deliberately *reuses* hard-won ideas from adjacent disciplines rather than
reinventing them — see [`prior-art.md`](prior-art.md). Its contribution is the **operating
model**: the unification of these mechanisms under an employment metaphor that makes the
whole system legible to humans who must trust agents with real work.

## 8. Reference implementations

Concrete systems (resident orchestrators, message bridges, dashboards, persona libraries)
are **downstream reference implementations** of this model. They validate the framework but
do not define it. The framework is the portable asset; implementations are interchangeable.

---

© 2026 Eugene Calalang. All rights reserved.
