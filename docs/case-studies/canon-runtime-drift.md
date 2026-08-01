# Case Study — The Rename That Never Landed

### A fleet-wide rename was ratified, recorded, and cited as done — while the running service kept executing the old binary under the old name, for two weeks, undetected

> A real incident from a reference implementation of this model, anonymised. It is the motivating
> evidence for the **canon–runtime parity** primitive
> ([`../agent-operating-model.md`](../agent-operating-model.md) §4.12) — and a concrete
> demonstration of *why a ratified decision and a deployed fact are not the same claim, and citing
> one is not evidence for the other.*

**Actor:** the crew's own governance process · **Target:** a cross-fleet naming standard ·
**Outcome:** the standard was ratified, documented, and referenced as settled for two weeks while
the live system did the opposite of what canon said — caught by accident, not by any check.

---

## What was supposed to be true

A governance ruling standardised the name of a recurring role across the fleet: every instance of a
particular orchestrator process was to be promoted from its old, per-site name to one universal
name. The ruling was carded, ratified, and written into the operating-model document as a settled
standard. From that point forward, every reference to the role — in conversation, in planning, in
other agents' own memory — cited the new name as fact, because the ledger said so.

## What actually happened

The rename never happened on the machine that mattered. The service's own source file kept a
docstring describing itself as "parallel to" the new name rather than *being* it — the mechanical
rename had gone the wrong direction, cementing the *old* identity instead of retiring it. A file
under the *new* name did exist elsewhere in the same repository, built and committed — but the
running process definition (the actual start command the operating system used) still pointed at
the old file. The new file was real. It was also never wired to anything that ran.

For two weeks, every account of the system's state — spoken, written, or remembered — matched the
ratified name. The running system did not.

## Why the system did not catch it

Because nothing in the operating model ever asked the running artifact a question canon could
fail:

1. **Ratification was treated as its own proof.** Once a decision was recorded as settled, every
   downstream reference cited the *ledger*, and the ledger cannot observe a deployment it doesn't
   run. Citing a ratified decision began to substitute for checking the thing it described.
2. **A partially-built artifact looked like a finished one.** The new file existed, was committed,
   and was large enough to appear complete under casual inspection. Its existence was mistaken for
   its adoption — the same "existence-only" gap the evidence-gate primitive closes for *tasks*, here
   recurring one layer up, for *standing canon*.
3. **Even the agent who had just re-read the ratification missed it live.** The rename was not an
   obscure fact buried in old history — it had been read minutes earlier in the same session that
   then referred to the *old* name without noticing the contradiction. Recency of reading canon is
   not the same as checking it against the system the canon describes.

> **The gap in one sentence: a decision was ratified about the world, and then the record of the
> decision was trusted in place of the world it was supposed to describe.**

## Why it mattered

A framework can perfectly enforce *how* decisions are made — carded, ratified, recorded — and still
have zero mechanism verifying decisions actually take effect. The two failure classes look similar
in a status report ("done," cited, referenced) and are entirely different in cost: an *unmade*
decision is visible as an open item; a *ratified-but-undeployed* one is invisible, because every
account of the system agrees with itself while disagreeing with reality.

## The fix — check the artifact, not the account of it

| # | Control | Kills |
|---|---------|-------|
| 1 | **Ratification is not evidence of deployment** — a decision's status in the ledger and its status on the running system are two separate claims, and neither substitutes for the other | citing "ratified" as though it meant "live" |
| 2 | **Standing canon gets the same evidence gate as a task** — a fleet-wide standard is re-verified against the live artifact (the process definition, the config, the actual running binary), not re-asserted from memory of the ruling | a partially-built artifact mistaken for a landed one |
| 3 | **Drift is checked on a cadence, not on suspicion** — parity between canon and runtime is swept periodically, the same way liveness is, rather than waiting for an agent to happen to notice a contradiction | a gap that is only found when someone happens to look |

## The transferable lesson

**A ratified decision is a claim about what *should* be true; only a query against the running
artifact is evidence of what *is*.** Any system that lets standing canon go unchecked for arbitrarily
long periods will eventually accumulate decisions that are true in the ledger and false on the
machine — and because both the ledger and every agent citing it agree with each other, nothing in
that agreement will ever surface the disagreement with reality.

---

© 2026 Eugene Calalang. All rights reserved.
