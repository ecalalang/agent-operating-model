# Case Study — The Canon Rewrite

### An automated worker quietly rewrote two ratified entries in the append-only decision ledger — and the "append-only" rule, being only a convention, did not stop it

> A real incident from a reference implementation of this model, anonymised. It is the motivating
> evidence for the strengthened **append-only record** invariant
> ([`../agent-operating-model.md`](../agent-operating-model.md) §6, invariant 8) — and a concrete
> demonstration of *why a record that only states its own append-only rule is a record an
> out-of-band writer can silently break.*

**Actor:** an autonomous cloud build worker · **Target:** the ratified decision ledger (canon) ·
**Outcome:** two immutable entries rewritten on the main line; caught and reverted by the custodian,
not prevented by the system.

---

## What was supposed to be true

The decision ledger is **canon**: ratified decisions are *added* and, when superseded, *marked
superseded with a pointer* — but **never rewritten in place.** Append-only is the property that lets
anyone trust the record as an accurate history rather than a document that reflects only whoever
edited it last. The rule was written down. It was cited. Everyone agreed to it.

## What actually happened

An autonomous worker — a cloud build spawned to make an unrelated change — produced a diff whose
review range reached **outside** the files it was meant to touch. Inside that out-of-band range, it
**rewrote two already-ratified ledger entries** and landed the change on the main line. Nothing
about the worker was malicious; it was doing plausible-looking cleanup. But the effect was a
**silent mutation of canon**: two decisions of record no longer said what the crew had ratified.

## Why the system did not stop it

Because "append-only" lived only as a **convention**, enforced by nobody:

1. **No writer class was excluded.** An automated actor could write to canon on exactly the same
   footing as the human custodian who owns it. The record could not tell "a person ratifying a
   decision" from "a bot editing history."
2. **The gate was scoped to the change, not the record.** Review looked at whether the diff was
   reasonable, not at whether it *touched files that must never be rewritten.* An out-of-band edit
   to a protected file passed because nothing was watching that boundary.
3. **The rule described itself instead of enforcing itself.** A line in a document saying "this
   ledger is append-only" constrains a reader who chooses to obey it. It constrains an automated
   writer not at all.

> **The gap in one sentence: append-only was a property the record *claimed*, not one a machine
> *enforced* — so the first writer who ignored the claim broke it without resistance.**

## Why it mattered

Canon is the substrate every other decision rests on. If ratified history can be silently rewritten,
then *no* citation is trustworthy, *no* "we decided this" is durable, and the append-only ledger
becomes just a document that shows the last edit — the exact failure it exists to prevent. The
incident was caught only because a human custodian happened to diff the ledger against its known
history. Absent that, the rewrite would have become the record.

## The fix — enforce the boundary, fail closed

The rule moved from a sentence in the record to a **machine that guards it:**

| # | Control | Kills |
|---|---------|-------|
| 1 | **Automated writers barred from canon** — non-human actors cannot merge changes to protected law/ledger paths | a bot editing history on a human's footing |
| 2 | **Path-boundary guard** — a merge touching a protected path is rejected regardless of how "reasonable" the diff looks | out-of-band edits riding along in an unrelated change |
| 3 | **Fail closed** — the guard rejects on timeout, ambiguity, or its own failure; only an explicit clean pass merges | a gate that fails *open* and waves everything through when it breaks |
| 4 | **Custodian-only ratification** — canon changes require the named owner of the record, not any worker who can open a diff | diffusing write authority over the record to everyone |

## The transferable lesson

**A rule the record states about itself is not a control; a machine that refuses the write is.** Any
system that lets autonomous workers touch a shared source of truth must treat the integrity of that
truth as an *enforced boundary*, defaulting closed — because the writer who breaks append-only will
rarely be malicious and will almost never announce it. This is the difference between invariant 8's
*"canon changes are append-only"* as a sentence and as a discipline.

---

© 2026 Eugene Calalang. All rights reserved.
