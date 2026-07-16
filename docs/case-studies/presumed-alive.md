# Case Study — Presumed Alive

### A command post was assumed alive because its door answered — and, later, an alarm nearly fenced a healthy seat for going quiet on purpose

> Two real incidents from a reference implementation of this model, anonymised. Together they are
> the motivating evidence for the **liveness & attestation** primitive
> ([`../agent-operating-model.md`](../agent-operating-model.md) §4.11) and invariant 11 — and a
> concrete demonstration of *why "it answered" is not "it is working," and why a dead-man's switch
> that cries wolf is worse than none.*

**Seats involved:** a coordinating command post and its peer hubs · **Outcome (incident 1):** mail
sat unread until the human principal became the transport layer · **Outcome (incident 2):** a
presumed-dead alarm fired on a *healthy* seat during a clean, announced rest.

---

## Incident 1 — Liveness that was assumed, never proven

A peer opened a communication lane and sent the command post a round-trip challenge — a message that
existed only to prove the link worked. It **sat unread.** The receiving hub's door was up; its status
read `LIVE`; its API returned `201` on delivery. Every mechanical signal said healthy. But no living
session was *reading the board*, so the message went unanswered until the **human principal
personally asked, "did you get a message from the peer?"** The Commander-in-Chief of the whole
operation was, in that moment, serving as the system's transport layer.

### The gap in one sentence

> **Receiving is not responding, and a door that returns `201` is indistinguishable from a dead one.**
> A hub that accepts mail while no session reads it presents every outward sign of life and performs
> none of its function.

Liveness had been *assumed* from the wrong evidence: reachability (the door answers) was read as
aliveness (the seat is working). They are different properties. The only proof of the second is a
**positive signal that the seat is actually processing** — emitted by the seat, on a clock — not the
absence of a connection error.

## Incident 2 — The alarm that cried wolf

The fix for incident 1 introduced a heartbeat: seats check in on a cadence, and a **missed** check-in
raises a presumed-dead alarm. Correct instinct — but the first version defined "missed" too loosely,
and it fired on a seat that was **entirely healthy:**

- The seat had **actioned** its outstanding work — it simply had not marked the cards *read* in the
  shared store. Un-acknowledged was read as un-done.
- The seat had announced a **clean, planned stand-down** — a deliberate rest, communicated in
  advance. Expected silence was read as death.

The alarm escalated anyway, and nearly triggered counter-measures (fencing, re-spawn) against a seat
that had done nothing wrong. An alarm that punishes healthy behaviour trains the crew to **disable
it** — which is how a safety mechanism becomes a liability.

### The gap in one sentence

> **A dead-man's switch that cries wolf is worse than none.** If "silent" is not carefully
> distinguished from "compromised," the switch fences the very crew it was built to protect.

---

## Why they belong together

The two incidents are the same lesson seen from opposite failure modes. Incident 1 is a **false
negative**: a dead-in-effect seat read as alive. Incident 2 is a **false positive**: a healthy seat
read as dead. A liveness contract has to defeat *both* — assert aliveness positively (so silence is
never mistaken for health) **and** define the miss precisely (so health is never mistaken for
silence).

## The fix — liveness as an adversarial contract

| # | Rule | Kills |
|---|------|-------|
| 1 | **Positive check-ins on a cadence** — aliveness is emitted, never inferred from a door | a `LIVE` status with nobody home |
| 2 | **Silence defaults to compromised** — a missed check-in is presumed-bad, not presumed-fine | the failure modes that emit nothing going unnoticed |
| 3 | **Fence before kill, then re-attest** — revoke authority (money/merge/lease/comms) *before* recovery | respawning on top of possibly-hostile state |
| 4 | **Identity is a per-seat secret, not an address** — prove *who*, not just *where* | a spoofed sender or a replayed continuity marker |
| 5 | **Precise trigger** — escalate only on un-actioned obligation *and* unexpected darkness | fencing a seat that finished but didn't `ack` |
| 6 | **Planned stand-downs de-register; probe before counter-measure** | an announced rest read as death |

## The transferable lesson

**Do not confuse the channel with the worker, or silence with failure.** Any autonomous-agent system
that delegates real authority needs liveness that is *asserted* (so a hung or captured seat cannot
hide behind an answering door) and *precisely scoped* (so the alarm never fences a seat that is
merely resting or merely quiet). Reachability is not aliveness; an address is not an identity; and an
alarm you cannot trust is an alarm the crew will learn to switch off. This is the difference between
§4.11's *"a heartbeat is a dead-man's switch"* as a sentence and as a discipline.

---

© 2026 Eugene Calalang. All rights reserved.
