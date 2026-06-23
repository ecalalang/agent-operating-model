# Prior Art — an honest map

The Agent Operating Model (AOM) does **not** claim to invent agent orchestration or durable
execution. Those are active, well-populated fields. This document maps the adjacent work
honestly and states precisely what AOM adds. Intellectual honesty *strengthens* ownership:
the defensible asset is the **synthesis and framing**, not a claim that nothing came before.

> Snapshot date: 2026-06-23. This is a working map to be revisited; it reflects the state of
> the field as understood at capture time.

## Adjacent work

| Project / pattern | What it provides | Overlap with AOM |
|---|---|---|
| **CrewAI** | Role/goal/skill agent definitions; stepwise task execution; human-in-the-loop | Persona-card idea; role teams |
| **AutoGen** (Microsoft) | Declarative agent graphs; conversational multi-agent; tool invocation | Multi-role coordination; tool use |
| **LangGraph** | Graph of nodes (agents/tools) with state and cycles | Stateful task flow; step memory |
| **OpenAI Swarm** | Lightweight handoff-based multi-agent coordination | Hand-off between roles |
| **Durable execution** (Temporal; OpenAI durable workflows) | Crash-tolerant, pause/resume, persistent state, retries | Parked-task / resumable task lifecycle |
| **Message queues / consumer groups** (e.g. SQS, Kafka) | At-least/exactly-once delivery; claims; idempotency | Claim + idempotency safety net |
| **Actor model** (Erlang/Akka) | Addressable, isolated units with mailboxes | Lane addressing; isolation |

## What is genuinely shared (and reused on purpose)

- **Durable, resumable tasks** — durable-execution engines (Temporal et al.) already solve
  pause/resume/retry. AOM's *parked task* is this idea, applied to human/async escalation.
- **Role/persona definitions** — CrewAI/AutoGen already model agents as roles with skills.
  AOM's *persona card* is in the same family.
- **Exactly-once delivery** — claims + idempotency are classic queue discipline. AOM reuses
  them as a *safety net*, not a novelty.

## What AOM contributes

The contribution is the **operating model** — the unification — plus a few specific contracts
that the surveyed frameworks do not foreground:

1. **The employment operating-model framing.** A complete, one-to-one mapping from
   employment concepts to technical primitives, chosen because it makes an agent workforce
   *legible and trustworthy to the humans who must delegate real work to it.* The frameworks
   above are engineering libraries; AOM is a governance/operating lens over them.
2. **An explicit role + trigger taxonomy** — exactly one always-on orchestrator; everyone
   else event-, anomaly-, on-demand-, or task-duration-triggered; and a **session-bound
   judgment role that is forbidden from autonomous decisions by design.** Most frameworks
   let any agent run; AOM prescribes *who is allowed to be awake and why.*
3. **Lane addressing as a first-class delivery contract** — `dispatch` / `direct:<session>` /
   `correlation-id`, with the rule that **only the orchestrator auto-drains dispatch.** This
   prevents double-action *by construction*, upstream of claims.
4. **The "two doors" identity model** — the same persona (same card, same memory) is
   reachable both as a live human-driven session and as an orchestrated ephemeral spawn,
   with take-over semantics. The frameworks treat interactive vs autonomous as different
   modes; AOM treats them as two doors into one employee.
5. **Persona-scoped learning as a first-class loop** — memory bound to the *persona*, not the
   process, hydrated at spawn and persisted at release, so successive ephemeral instances
   accumulate expertise like a tenured employee.
6. **Context continuity via a mandated working journal** — workers journal *working* context
   as they go (not just outcomes), so any later instance or human reconstructs "what
   happened." Durable engines persist *state*; AOM additionally mandates a human-readable
   *handover*.
7. **A bounded chain of command** — autonomy as an explicit, per-role dial inside a stated
   hierarchy, rather than an implicit default.

## Honest bottom line

If you need an engineering library to *build* a crew, the projects above are excellent and
AOM sits happily on top of any of them. If you need a **model for how an agent workforce is
organised, governed, and trusted** — who may run, how they avoid colliding, how they escalate
and hand over, how they learn, and who answers to whom — that organising model is what AOM
names and what this repository captures.

---

© 2026 Eugene Calalang. All rights reserved.
