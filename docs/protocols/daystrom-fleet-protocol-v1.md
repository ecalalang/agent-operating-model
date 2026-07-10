# Daystrom Fleet Protocol (DFP) v1 — hub-to-hub card comms standard

> A vendor-neutral wire-contract for card-based communication between an agent hub
> ("Daystrom") and any non-Daystrom peer — another ship or another hub.
>
> Status: **v1 — ratified 2026-07-10.** Standing standard for ship↔hub / hub↔hub comms.
> This document is **clean-room and codename-only.** No customer, employer, or product is
> named — the human/display label of any ship is deliberately kept out of this canon, which
> is the same privacy rule the protocol itself enforces on the wire (§4). Any concrete
> deployment is a *downstream reference implementation* (§8), not the standard.

---

## 1. Purpose

DFP is a **reusable standard for non-Daystrom → Daystrom card comms** — the message plane by
which a ship talks to a hub, or one hub talks to another. It generalises the point-to-point
federation link into an **N-ship fleet**: every future ship inherits the same wire contract,
and onboarding a new ship is **configuration only** (§7), not bespoke code.

The unit of exchange is a **card** — a durable, addressed, single-consumer message (the AOM
agent-card primitive, [`../agent-operating-model.md`](../agent-operating-model.md) §4). DFP
specifies the card's wire format, its addressing, its transport abstraction, and the
liveness signal that proves the link is healthy.

**Model:** one **hub per customer** — each customer is served by its own Daystrom hub/ship.
Ships and hubs are peers on a shared **message-bus**; they are *not* a shared runtime (§5).

---

## 2. Card wire format

Every card is a row in a `federation_messages` table on the shared bus:

| column        | type            | meaning |
|---------------|-----------------|---------|
| `from_hub`    | TEXT            | sender hub codename (e.g. `DS9`) |
| `to_hub`      | TEXT            | recipient hub codename (e.g. `Enterprise`) |
| `from_persona`| TEXT            | sending persona on the sender (e.g. `zora`) |
| `to_persona`  | TEXT            | target persona on the recipient (e.g. `daystrom`) |
| `cid`         | TEXT **UNIQUE** | correlation id — the idempotency key |
| `subject`     | TEXT            | short human-readable summary |
| `body`        | TEXT            | payload |
| `class`       | TEXT            | message class (see below) |
| `status`      | TEXT            | delivery state — `pending` → `delivered` |
| `created_at`  | TEXT            | UTC timestamp of insert |

**Message class** — `class ∈ { comms, directive, evidence, heartbeat, ack }`.

**Addressing** is `ship.persona` — the pair `(to_hub, to_persona)`. Examples: `DS9.zora`,
`Enterprise.daystrom`. A hub is addressed by its **codename**, never by a customer/display
name (§4).

**Idempotent send.** Writes are `INSERT OR IGNORE` on `cid` (the UNIQUE constraint) — a
resend with the same `cid` is a no-op, so retries and duplicate polls never double-deliver.

**Poll / ingest loop.** A hub consumes its own inbox:

```sql
SELECT * FROM federation_messages WHERE to_hub = <self> AND status = 'pending';
-- ingest each card into local handling, then:
UPDATE federation_messages SET status = 'delivered' WHERE cid = ?;
```

Exactly one consumer per card (single-consumer lane addressing); `delivered` is the terminal
wire state.

---

## 3. Transport abstraction — the reusable core

DFP defines **one Broker interface** with **two interchangeable backends**, selected per link
by `federation_broker.transport`. The card format (§2) and the allowlisted SQL are **identical
on both** — only the connection to the bus differs:

| `transport`     | backend | when |
|-----------------|---------|------|
| `local-sqlite`  | co-located ships share **one local** `fleet/federation.db` | ships on the same machine |
| `subspace-ssh`  | existing cloudflared → remote SQLite over a forced-command SSH broker | ships on separate machines (e.g. a remote hub) |

Both backends execute the **same allowlisted `federation_messages` statements** (insert,
poll, mark-delivered, heartbeat upsert). The Broker interface hides the difference, so hub
logic never branches on transport.

A single hub **may run BOTH backends at once** and poll each independently — e.g. a hub can
serve co-located ships over `local-sqlite` while simultaneously federating to a remote hub
over `subspace-ssh`. Each configured broker is one poll cursor.

---

## 4. Hub identity & privacy

Every hub has:

- a **`hub` codename** — a unique, opaque wire id (e.g. `DS9`). This is what appears in
  `from_hub` / `to_hub`, in addressing, and in logs.
- a **`display`** — the human label for the hub (the customer name). It is used **only** in
  local, human-facing surfaces.

**Privacy invariant:** the **customer / display name never enters the wire or the logs.** The
bus, the card rows, and the audit trail carry codenames only. This is why this canonical
document names no customer either — the codename `DS9` is written here; its `display` is not.

**Personas** on a hub:
- a **resident dispatcher** (e.g. `zora`) — receives inbound cards for the ship,
- an **orchestrator** (e.g. `worf`) — routes/drains work locally,
- **command personas** — named roles addressable as `hub.persona`.

---

## 5. Command-bus isolation

Co-located ships share **only the fleet message-bus** (`fleet/federation.db`). They do **not**
share a runtime:

- Each ship keeps its **own local command / inbox database** — its work, tasks, and internal
  dispatch are private to that ship.
- The fleet bus is the *only* crossing point between ships. It carries cards, nothing else.

The implementation is **one core, per-ship config**: a single shared broker/hub codebase,
parameterised by each ship's own registry (identity, personas, brokers). No ship forks the
core to join the fleet.

---

## 6. Heartbeat

Liveness is proven, not assumed. Each hub **upserts its own `federation_heartbeat` row once
per poll cycle** (piggybacked on the broker poll — no separate timer), stamping its last-seen
time, its auto-poll `enabled` flag, and its poll cursor. A monitor reads **both** hubs' rows
and verifies the link **bilaterally** — a hub that stops upserting, or that disables its own
poll, is detected as stale even though no card was lost.

- A hub writes **only its own** heartbeat row and **reads** the peer's; neither hub mutates
  the other's liveness state.
- The heartbeat is **additive** — it never reads or writes the `federation_messages` delivery
  path.

The `subspace-ssh` reference of this signal (tables, 30s cadence, 90s staleness,
GREEN/DEGRADED/DOWN semantics) is specified in the Enterprise hub's
`federation-heartbeat-contract.md`; DFP adopts its shape as the standard heartbeat for **any**
transport.

---

## 7. Onboarding ship N — the payoff

Adding the Nth ship is a **drop-in registry change**, no bespoke code:

1. Assign a unique **`hub` codename** (+ a local-only `display`).
2. Declare its **personas** (dispatcher, orchestrator, command roles).
3. Declare its **`federation_broker { transport, dsn }`** — `local-sqlite` (point at the
   shared `fleet/federation.db`) or `subspace-ssh` (point at the remote broker).
4. **Enable** it.

The shared core (§5) reads the registry and the ship is on the bus. Because the wire format
(§2), transport abstraction (§3), identity/privacy (§4), and heartbeat (§6) are fixed by this
standard, ship N inherits all of them for free. This is the whole point of DFP: **the fleet
scales by configuration.**

---

## 8. Reference implementation

**Ship #1 — codename `DS9`** — is the first proven instance, communicating bidirectionally
with hub **`Enterprise`** (persona `daystrom`) over a co-located `local-sqlite` fleet bus.
Both directions were exercised on a shared local fleet DB.

- **Fleet bus:** a single local SQLite DB shared by the co-located ships
  (`~/.copilot/fleet/federation.db`).
- **Hub side (`Enterprise`):** a `local` broker entry with `transport: local-sqlite`,
  `peer: DS9`, `from_persona: daystrom`, `to_persona: zora`. The same hub also runs a
  `subspace-ssh` broker to a remote peer — demonstrating the "both backends at once" property
  of §3.
- **Ship side (`DS9`):** its registry declares `hub: DS9` with a local-only `display`, a
  resident dispatcher (`zora`) and orchestrator (`worf`), and a `local-sqlite` broker onto the
  shared fleet bus.

**Provenance (landing commits, 2026-07-10):**

- Hub side — `fc2146a` *"feat(federation): DFP v1 — pluggable broker transport (local-sqlite)
  + DS9 fleet peer"*.
- Ship side — `90f40c1` *"feat(federation): DFP v1 — DS9 as ship #1 on local-sqlite
  transport"*.

The concrete deployment record — display name, absolute paths, and hub-specific wiring — is
kept in the Enterprise hub's decision ledger, not in this vendor-neutral canon.

---

*Daystrom Fleet Protocol v1. Ratified 2026-07-10. Curated by the Chronicler.*
