# The Agent Operating Model (AOM)

**A vendor-neutral framework for running AI agents as a managed, accountable workforce.**

Author: **Eugene Calalang**
First captured: **2026-06-23**
Status: Draft v0.1 — original framework, captured for provenance.

---

## What this is

Most AI-agent work today produces *a* clever agent — a single assistant that answers
prompts or runs a tool. The Agent Operating Model is one level up: it describes how a
**crew of agents** is hired, trained, assigned work, coordinated, and held accountable —
the *organisational* layer for an agent workforce.

The core idea: **treat an AI agent like an employee, not a function call.** An employee
has a job description, retains what they learn, is assigned work by a manager, escalates
questions, never duplicates a colleague's ticket, leaves handover notes, can be spoken to
directly or left to work asynchronously, and reports up a chain of command. Every one of
those is a concrete technical primitive — and together they form an operating model.

This document is **clean-room and vendor-neutral**. It names no employer, customer,
product, or proprietary tool. Any specific implementation (orchestrators, bridges,
dashboards) is a *downstream reference implementation* of the model described here — not
the model itself.

## Why it matters

As agents become capable enough to do real work, the hard problems stop being "can the
model answer?" and become **coordination problems**: Who picks up this task? What happens
when an agent needs to ask a human something mid-task? How do two agents avoid doing the
same job? How does a new instance know what the last one was doing? These are the same
control-plane problems that distributed systems, message queues, and durable-execution
engines have always had to solve — re-encountered in an agentic world, and best expressed
in the familiar language of *employment*.

## Provenance

This framework was synthesised by Eugene Calalang from first principles while designing a
real multi-agent crew. It is captured here, dated and attributed, as portable personal IP.
Related fields exist (multi-agent orchestration, durable execution); the contribution of
this work is the **synthesis and the employment operating-model framing** — see
[`docs/agent-operating-model.md`](docs/agent-operating-model.md) for the full model,
[`docs/diagram.md`](docs/diagram.md) for the one-page visual, and
[`docs/prior-art.md`](docs/prior-art.md) for an honest map of adjacent work.

---

© 2026 Eugene Calalang. All rights reserved.
