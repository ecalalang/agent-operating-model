# Prior Art — an honest map

The Agent Operating Model (AOM) does **not** claim to invent agent orchestration or durable
execution. Those are active, fast-moving, well-funded fields. This document maps the adjacent
work honestly and states precisely what AOM adds. Intellectual honesty *strengthens* ownership:
the defensible asset is the **synthesis and framing**, not a claim that nothing came before.

> Snapshot date: 2026-06-23 (fact-checked). The agent-framework field moves quarterly; this is a
> living map. Where the field has caught up to a primitive AOM once foregrounded, that is recorded
> below rather than hidden — see "Where the field has caught up."

## Adjacent work

| Project / pattern | Current state (2026) | Overlap with AOM |
|---|---|---|
| **OpenAI Agents SDK** | Production successor to **Swarm** (Swarm deprecated Mar 2025). v2 (2026): handoffs, sessions/memory, guardrails, tracing, Temporal integration | Hand-off between roles; session memory; durable workflow hooks |
| **Microsoft Agent Framework (MAF)** | Unification of **AutoGen + Semantic Kernel** (both in maintenance mode since Oct 2025; MAF GA ~Q1 2026). Declarative multi-agent graphs, MCP/A2A native, OpenTelemetry, Entra | Multi-role coordination; tool use; observability/governance |
| **CrewAI** | Role/goal/skill crews **+ Flows**: event-driven orchestration with checkpointing, persistent state, replay/fork; AMP enterprise suite, A2A | Persona-card idea; role teams; **durable/resumable tasks (Flows)** |
| **LangGraph** | Graph of nodes with state/cycles **+ durable execution**: checkpointers (thread state) and **Stores (long-term cross-thread memory)**, time-travel/replay, auto-persistence on Agent Server | Stateful task flow; step memory; **resumable tasks; long-term memory** |
| **AWS Strands + Bedrock AgentCore** | Durable, framework-agnostic agent runtime: session isolation (microVM), persistent filesystem across stop/resume, up to 8h async, **episodic memory** (cross-session learning), policy enforcement, MCP/A2A | **Parked/resumable tasks; persona-scoped learning; claim/isolation; policy/autonomy guardrails** |
| **Anthropic Claude Agent SDK** | Orchestrator–worker **subagents**, tool use, MCP-native; published orchestrator-worker multi-agent patterns | One always-on orchestrator delegating to workers |
| **Durable execution** (Temporal; OpenAI/cloud durable workflows) | Crash-tolerant, pause/resume, persistent state, retries — now commonly embedded *under* the agent frameworks above | Parked-task / resumable task lifecycle |
| **Interoperability standards** — **A2A** (Google, Apr 2025; Linux Foundation) + **MCP** (Anthropic) | A2A = horizontal agent↔agent delegation with a published **"Agent Card"** capability descriptor; MCP = vertical agent↔tool. Joint spec targeted ~2026 | **"Agent Card" ≈ persona card; capability advertisement + task delegation ≈ lane addressing** |
| **Message queues / consumer groups** (SQS, Kafka) | At-least/exactly-once delivery; claims; idempotency | Claim + idempotency safety net |
| **Actor model** (Erlang/Akka) | Addressable, isolated units with mailboxes | Lane addressing; isolation |

## What is genuinely shared (and reused on purpose)

- **Durable, resumable tasks** — durable-execution engines (Temporal) and now the agent
  frameworks themselves (CrewAI Flows, LangGraph checkpointers, AgentCore persistent runtime)
  solve pause/resume/retry. AOM's *parked task* is this idea, applied to human/async escalation.
- **Role/persona definitions** — CrewAI/MAF/Strands model agents as roles with skills; A2A even
  standardises an **"Agent Card"**. AOM's *persona card* is in the same family.
- **Persona/cross-session memory** — LangGraph Stores and AgentCore **episodic memory** already
  persist learning across sessions. AOM's *persona-scoped learning loop* is the same idea.
- **Exactly-once delivery** — claims + idempotency are classic queue discipline. AOM reuses
  them as a *safety net*, not a novelty.

## Where the field has caught up (intellectual honesty)

When AOM was first framed, several of its primitives were not foregrounded by mainstream
frameworks. As of this snapshot, the field has converged on a number of them. Recording this
openly is the point — it keeps the *real* contribution defensible:

- **Capability cards** — now an industry standard via A2A's **Agent Card**. AOM's *persona card*
  is no longer distinctive as a mechanism.
- **Durable / parked tasks** — now table-stakes (CrewAI Flows, LangGraph, AgentCore, Temporal).
  AOM's *parked task* is a framing of a solved mechanism, not a new engine.
- **Cross-session persona memory** — now offered directly (LangGraph Stores, AgentCore episodic
  memory). AOM's *learning loop* is the same capability, applied per persona.
- **Orchestrator–worker delegation** — standard (Claude subagents, CrewAI manager agents).

What this leaves is the part that the engineering frameworks above deliberately do **not** take a
position on: *how an agent workforce is organised, governed, and trusted by the humans who
delegate to it.*

## What AOM contributes (narrowed and defended)

The contribution is the **operating model** — the governance/employment lens — plus a few
specific contracts the surveyed frameworks still do not prescribe:

1. **The employment operating-model framing.** A complete, one-to-one mapping from employment
   concepts to technical primitives, chosen because it makes an agent workforce *legible and
   trustworthy to the humans who must delegate real work to it.* The frameworks above are
   engineering libraries and runtimes; AOM is a governance/operating lens over any of them.
2. **An explicit role + trigger taxonomy — "who is allowed to be awake and why."** Exactly one
   always-on orchestrator; everyone else event-, anomaly-, on-demand-, or task-duration-triggered;
   and a **session-bound judgment role that is forbidden from autonomous decisions by design.**
   Most frameworks let any agent run; AOM prescribes who may run and who may not.
3. **The "two doors" identity model with take-over semantics.** The same persona (same card, same
   memory) is reachable both as a live human-driven session and as an orchestrated ephemeral
   spawn, and control can pass between them. Frameworks treat interactive vs autonomous as
   different modes; AOM treats them as two doors into one employee.
4. **A mandated human-readable working journal (handover), not just persisted state.** Workers
   journal *working* context as they go — "what happened and why" — so any later instance *or a
   human* can reconstruct it. Durable engines persist machine *state*; AOM additionally mandates a
   human-legible handover. (Note: episodic memory stores persist outcomes/facts; AOM's journal is
   specifically the human-readable narrative.)
5. **A bounded chain of command.** Autonomy as an explicit, per-role dial inside a stated
   hierarchy, rather than an implicit default — closer to AgentCore's policy layer in spirit, but
   expressed as an org chart rather than a rule engine.

> Net: the mechanical primitives (cards, durable tasks, cross-session memory, handoffs) are now
> commodity. AOM's defensible core is the **operating/governance model layered over them**: the
> employment framing, the awake/forbidden trigger taxonomy, two-doors take-over, the human
> handover mandate, and the explicit chain of command.

## Honest bottom line

If you need an engineering library or runtime to *build and run* a crew, the projects above are
excellent and AOM sits happily on top of any of them (CrewAI, LangGraph, MAF, OpenAI Agents SDK,
Strands/AgentCore). If you need a **model for how an agent workforce is organised, governed, and
trusted** — who may run, how they avoid colliding, how they escalate and hand over to a human, how
they learn, and who answers to whom — that organising model is what AOM names and what this
repository captures.

---

© 2026 Eugene Calalang. All rights reserved.
