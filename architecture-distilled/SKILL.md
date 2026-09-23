---
name: architecture-distilled
description: Design and review software architectures using distilled lessons from AOSA open-source systems. Use for component boundaries, concurrency, storage, extensibility, reliability, architecture trade-offs, and ADRs grounded in explicit constraints.
license: CC-BY-3.0
---

# Architecture Distilled

Turn a system's constraints into a small, defensible design. Use historical open-source mechanisms as evidence, not as a shopping list of technologies. Respond in the user's language.

## Working method

1. **Frame the decision.** Establish workload, latency/availability targets, consistency, team/operations budget, existing components, and the requested scope. Label unknowns and assumptions; ask only when an answer would change the decision. For existing software, trace the actual data path and ownership first.
2. **Choose relevant precedents.** Read only the applicable cards in [references/casebook.md](references/casebook.md). Usually 2–4 are enough. Match the constraint and mechanism; explain where the analogy stops. Preserve chapter attribution. Historical descriptions do not establish current product capabilities.
3. **Compare feasible options.** Include the smallest option that can meet the constraints and a credible alternative. Compare failure behavior, operational cost, reversibility, and workload fit. Reject an alternative for a specific constraint, not a slogan.
4. **Make ownership and state explicit.** Define component responsibilities, key records/interfaces, source of truth, the request/data path, and important state transitions. For asynchronous boundaries, specify acceptance/durability, capacity, overload response, retries, idempotency, cancellation, and recovery. Include trust boundaries where relevant.
5. **Close the decision.** State the chosen design, its costs, how to test the risky assumptions, a migration/rollback path where applicable, and measurable conditions that would justify revisiting it. Distinguish measured results from proposed targets.

## Precedent router

| Pressure | Cards to consult |
| --- | --- |
| Many concurrent slow clients | nginx, Twisted |
| Messaging throughput or queues | ZeroMQ, Graphite |
| Multiple frontends/backends or extension boundaries | LLVM, SQLAlchemy, Bash |
| Revisions, immutable artifacts, audit history | Git |
| Local durable state or embedded transactions | Berkeley DB |
| Large sequential files and distributed storage | HDFS |

Use a direct source chapter when the card lacks enough evidence. Verify current documentation before recommending a contemporary API, limit, or deployment configuration. If no precedent fits, say so and reason from the constraints.

## Output contract

Scale the answer to the request. A substantial design should contain:

- **Decision and assumptions:** what is being decided, hard constraints, unknowns.
- **Design:** responsibilities, key structures, flow, state ownership, and a small diagram when useful.
- **Trade-offs:** plausible alternative, rejection reason, cost accepted by the chosen design.
- **Failure and verification:** crash/overload behavior, testable criteria, recovery or rollback, revisit triggers.
- **Evidence:** chapter links and a clear separation between source facts and your own transfer/inference.

For a review, lead with concrete risks, their affected path, and the smallest remedy. Avoid rewriting the architecture unless needed. Use [references/decision-template.md](references/decision-template.md) only when a written ADR is useful.

## Judgment rules

- An event loop moves waiting; it does not make blocking work disappear.
- A queue moves pressure; define what happens when it is full.
- A durable log does not make an external side effect exactly once.
- Distribution adds coordination and recovery costs; justify it against the single-process or single-store baseline.
- Reuse an existing boundary before adding a framework or service. A shared representation earns its cost when variation actually exists.
- Never present invented measurements, source claims, or a project's past architecture as current verified fact.

## Example prompts

> Design webhook delivery for a three-person team, 100 requests/s average and 1,000 peak, with PostgreSQL already available. Compare a database-backed work queue with adding a broker. Define delivery states and a benchmark plan.

> Review this upload pipeline. Trace where bytes and metadata become durable, identify unbounded queues, and propose the smallest fix with an AOSA precedent.

> Our compiler supports two languages and three targets. Use LLVM as a precedent to assess a shared IR, including the abstraction cost and cases where it would not help.
