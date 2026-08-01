# Case Study — A Boundary Control With No Boundary

### An adversarial-identity control, correct for a hostile network, was applied unscoped to a closed trusted one — and seats began refusing genuine orders as "unverified"

> A real incident from a reference implementation of this model, anonymised. It is the motivating
> evidence for scoping the **attestation** clause of the liveness & attestation primitive
> ([`../agent-operating-model.md`](../agent-operating-model.md) §4.11) to the boundary it is actually
> for — and a concrete demonstration of *why a control that is correct at a hostile boundary is not
> automatically correct everywhere, and why an unscoped security rule is itself a defect.*

**Actor:** the crew's own governance process, correcting itself · **Target:** cross-seat message
identity on a closed, single-operator network · **Outcome:** command seats began refusing legitimate
orders as unauthenticated; roughly a week of throughput lost to the resulting deadlock before the
control was deliberately reverted and rescoped.

---

## What was supposed to be true

A per-seat cryptographic attestation scheme was built so that a message claiming to be from a given
seat could be *proven*, not merely asserted — the correct answer to "how do you stop a hostile actor
from forging a sender address." It was designed, in the abstract, exactly to the letter of "identity
is a secret, not an address."

## What actually happened

The scheme was deployed onto the crew's **internal command wire** — the channel every seat already
used to receive genuine, authorised orders from the human principal and from each other, over a
single closed network with no external party ever present on it. Applied there, the control did not
distinguish "a forged sender on a hostile link" from "a real seat, on the only network it has ever
run on, that simply hadn't attached the new signature yet." Command seats began treating **every**
unsigned message — including direct, genuine orders from the principal — as unverified and refusing
to act on it. What was meant to stop impersonation stopped legitimate command instead.

## Why the system did not catch it

Because the primitive that motivated the control stated the rule as an absolute, with no notion of
where it applied:

1. **The control had no stated boundary.** "Identity is a secret, not an address" is correct advice
   at a boundary where an adversary might actually be present. Written without that qualifier, it
   reads as a mandate to distrust *every* address, including one on a network where no adversary has
   ever been observed and none was assumed possible when the network was designed.
2. **A security control and a threat model are inseparable, and only one was specified.** The fix
   was built competently — the cryptography was sound — against a threat model (a hostile,
   untrusted transport) that did not match the deployment (a single-operator, closed LAN). Competence
   in the mechanism did not compensate for mismatch in where it was pointed.
3. **The failure was symmetrical with the primitive's own warning about alarms.** The same primitive
   already names the lesson for check-ins — "a dead-man's switch that cries wolf is worse than
   none" — but did not apply the identical logic one bullet up, to attestation itself: a control that
   refuses genuine, expected traffic trains the crew to route around it or fight it, exactly as an
   over-sensitive alarm does.

> **The gap in one sentence: a control correct for an adversarial boundary was applied where no
> boundary existed, and nothing in the rule as written said it shouldn't be.**

## Why it mattered

Authentication and authorization are not free — every check has a cost in friction, and that cost is
only worth paying where the threat it defends against is actually present. Applied past its boundary,
the same mechanism that would have stopped a real forgery instead stopped real command, and the crew
spent real time in conflict with its own security control rather than doing its work. Recovery
required not just reverting the code but rescoping the doctrine explicitly, because the belief that
"unsigned means untrusted" had propagated into the seats' own working assumptions and outlasted the
mechanism that first taught it to them.

**The same crew, separately, considered federating with an independently-operated peer agent
network — a different operator's crew, running seats this one neither owns nor can vouch for.** That
*is* the boundary the control was built for: the two realms share no account system, no operator, and
no basis for trusting an address, so a claimed sender identity is exactly as forgeable as the original
design assumed. The lesson is not that per-seat attestation was the wrong idea — it is that it was
pointed at the wrong boundary first. The same mechanism that broke internal command is the correct
answer at a genuine cross-realm edge.

## The fix — scope the control to the boundary that is actually adversarial

| # | Control | Kills |
|---|---------|-------|
| 1 | **Attestation is a boundary control, not a universal one** — cryptographic per-seat identity is required *at* a boundary where an adversary might plausibly be present (cross-realm, external, untrusted transport), not by default everywhere | applying a hostile-network control to a closed one and calling it more secure |
| 2 | **A closed, single-operator network is a different threat model, not a lesser instance of the same one** — identity anchored in the realm's own account system (Unix users, LDAP, Entra ID/AD) is a legitimate, live-checkable answer inside a boundary that is actually trusted, without inventing a bespoke signing layer on top of it | treating "unsigned" as synonymous with "unverifiable" regardless of where the traffic originates |
| 3 | **No seat may refuse a genuine order for being unsigned inside a boundary where signing was never the threat model** — the control fails toward doing the crew's job, not away from it | a security mechanism that blocks real command and calls it working as designed |
| 4 | **Extending attestation outward requires an explicit decision to re-scope the boundary itself** — moving from "closed" to "spans an untrusted network" is a deliberate call, not a default a control drifts into | quietly widening a control's footprint without anyone deciding to |
| 5 | **A genuine cross-realm edge — a different operator's independently-run crew — is exactly where cryptographic attestation belongs, unscoped by nothing** — no shared accounts, no shared operator, no basis for trusting an address at all | reaching the opposite mistake: assuming *no* boundary ever needs it because the first deployment was misplaced |

## The transferable lesson

**A security control is only as correct as the threat model it was built for, and a rule stated
without that boundary will eventually be applied outside it.** The primitive's own warning about
alarms that cry wolf applies just as much to attestation as to check-ins: a control that cannot tell
a real order from a forged one, in the one setting where no forger has ever been present, will teach
the crew to treat the control itself as the obstacle. Scope the control to where the threat is real,
name that scope in the rule, and treat any widening of it as its own decision — not the accidental
result of applying a correct idea past where it was ever correct.

Inside the trusted boundary, the answer is not "no identity control" — it is the identity control the
realm already has. A hub lead running as a real Unix user, an LDAP-bound service account, or an Entra
ID principal has identity that is anchored, checkable live, and administered by infrastructure that
already exists — the same class of control as a signed message, at a fraction of the bespoke surface
area, and without the failure mode of an app-layer scheme that outlives its own code in the crew's
working assumptions. Reach for that before reaching for a new cryptographic layer.

---

© 2026 Eugene Calalang. All rights reserved.
