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

## Provenance — how this came about

I didn't set out to write a framework, and I didn't start from the literature. This came
out of practice — and it started on **my own personal project**, before I ever applied any
of it to my day job. I was building and running a multi-agent crew of my own, and I kept
hitting the same wall — not *can* the agents run, but *who is allowed to be awake, who picks
up which task, what happens when an agent needs to ask a human mid-task, and how does the
next instance know what the last one was doing.* Solving those problems one at a time, the
same shape kept appearing: I was, in effect, **employing** the agents — job descriptions, a
manager, handovers, a chain of command. The operating model is that shape, written down. The
crew I run now is not the first; it's a later application of a model I'd already arrived at
on my own.

I arrived at it **independently**. I hadn't read the internals of the existing agent
frameworks when the concept formed — it came from the problems in front of me, not from
copying a design. *Afterwards*, I deliberately mapped it against the field (CrewAI,
LangGraph, the Microsoft Agent Framework, OpenAI's Agents SDK, AWS AgentCore, the A2A/MCP
standards) to be honest about what overlaps and what doesn't. That map is in
[`docs/prior-art.md`](docs/prior-art.md), and it openly states which mechanics are now
industry-standard.

So I make a deliberately narrow claim. The *mechanics* — durable tasks, agent cards,
cross-session memory — are not new, and I say so. What is mine is the **independent
derivation from practice** and the **employment operating-model framing**: a way to make an
agent workforce legible, governable, and trustworthy to the humans who delegate to it. See
[`docs/agent-operating-model.md`](docs/agent-operating-model.md) for the full model and
[`docs/diagram.md`](docs/diagram.md) for the one-page visual.

For the discipline seen in practice, [`docs/case-studies/fabricated-completion.md`](docs/case-studies/fabricated-completion.md)
walks through a real incident where an existence-only completion gate accepted a fabricated result —
and the six-point provenance contract that now prevents it.

This document is captured here, dated and attributed, as portable personal IP.

---

© 2026 Eugene Calalang. All rights reserved.
