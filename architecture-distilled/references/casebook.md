# AOSA decision casebook

Source check: 2026-09-23. This document contains original, compressed case notes based on the authors' historical accounts in *The Architecture of Open Source Applications*. It is not a description or audit of current project releases. **Mechanism** and **Trade-off** summarize the cited chapter; **Ask**, **Fit**, **Avoid**, and **Check** are our transferable engineering interpretation, not quotations or claims that the author recommends the same design for every system.

### nginx — Bound the work inside the event loop

Source: Andrew Alexeev, [“nginx,” AOSA Volume 2, §§14.2–14.4](https://aosabook.org/en/v2/nginx.html). The contents page renders the author's first name as Andrey.

- **Mechanism:** A master manages configuration and workers; each worker multiplexes many connections through a nonblocking event loop. Request phases and filters provide extension points.
- **Trade-off:** Low per-connection overhead requires every handler to cooperate. The chapter identifies disk I/O and embedded scripts as potential worker-wide blockers.
- **Ask:** Which operation can monopolize a worker, and how many requests share its failure domain?
- **Fit:** Numerous mostly waiting connections and short, bounded callbacks.
- **Avoid:** Treating event-driven execution as a cure for unbounded CPU work or blocking dependencies.
- **Check:** Inject a slow handler and measure unrelated requests' tail latency. Isolate, bound, or offload the blocker before adding workers.

### LLVM — Make the intermediate contract complete

Source: Chris Lattner, [“LLVM,” AOSA Volume 1, §§11.3–11.6](https://aosabook.org/en/v1/llvm.html).

- **Mechanism:** Front ends, optimization passes, and back ends exchange a self-contained intermediate representation. Libraries let clients select the passes they need.
- **Trade-off:** Shared infrastructure reduces repeated implementation, but the representation must express source semantics and target requirements; specialized passes remain necessary.
- **Ask:** Can a downstream stage operate from the documented contract alone, without reaching into upstream internals?
- **Fit:** Several real producers or consumers that benefit from shared transformations.
- **Avoid:** Inventing a universal representation for a single fixed transformation, or erasing domain semantics to force reuse.
- **Check:** Serialize a minimal intermediate input and test one transformation in isolation. If the test requires unrelated stages, inspect the boundary.

### Git — Separate immutable history from mutable names

Source: Susan Potter, [“Git,” AOSA Volume 2, §§6.3–6.6](https://aosabook.org/en/v2/git.html).

- **Mechanism:** Immutable objects form content and history graphs; mutable references identify selected commits. Unchanged objects can be reused, and local commits precede optional publication.
- **Trade-off:** Independent histories support offline work and flexible sharing, but add synchronization, merge, and storage complexity.
- **Ask:** Which state needs durable identity, which names must move, and who resolves divergent updates?
- **Fit:** Versioned artifacts, reproducible snapshots, and workflows needing independent histories.
- **Avoid:** Assuming a content hash authenticates its author, resolves conflicts, or supplies transactional coordination by itself.
- **Check:** Verify unchanged content is reused, references move independently, and divergent edits require an explicit reconciliation policy.

### Twisted — Give asynchronous results a lifecycle

Source: Jessica McKellar, [“Twisted,” AOSA Volume 2, §21.2](https://aosabook.org/en/v2/twisted.html).

- **Mechanism:** A reactor dispatches events. Deferred objects collect success and error handlers for a result that fires once; protocols are separated from transports.
- **Trade-off:** Cooperative execution avoids much shared-state locking, but callback order and failure propagation become explicit design work.
- **Ask:** Where is completion owned, how does failure propagate, and can protocol behavior be tested without a live socket?
- **Fit:** Composable I/O workflows and protocols reused over multiple transports.
- **Avoid:** Letting blocking or long CPU operations occupy the event loop; assuming one-shot completion guarantees a remote action occurs exactly once.
- **Check:** Exercise success, failure, and duplicate completion with a fake transport. Use the platform's existing future/promise abstraction before building a new one.

### ZeroMQ — Keep ownership explicit across threads and libraries

Source: Martin Sústrik, [“ZeroMQ,” AOSA Volume 2, §§24.1–24.2 and 24.7–24.8](https://aosabook.org/en/v2/zeromq.html). The chapter heading renders the surname as Süstrik.

- **Mechanism:** Messaging lives in a library with explicit contexts. Internal objects belong to worker threads and communicate through asynchronous messages; ownership trees coordinate shutdown.
- **Trade-off:** Removing shared mutable access reduces lock contention but introduces event-ordering, fairness, and shutdown obligations.
- **Ask:** Who owns each mutable object, and what must finish before its owner can stop?
- **Fit:** Embedded communication components with bounded handlers and explicit ownership.
- **Avoid:** Treating a messaging library as an automatic substitute for a durable broker or assuming every workload benefits from more threads.
- **Check:** Instantiate independent contexts and test shutdown with pending work. Specify delivery and persistence requirements separately.

### Bash — Carry semantic context across stages

Source: Chet Ramey, [“The Bourne-Again Shell,” AOSA Volume 1, §§3.4–3.6](https://aosabook.org/en/v1/bash.html).

- **Mechanism:** Parsing produces command and word structures; flags preserve quoting, assignment, and expansion information for later stages. Context-sensitive grammar requires cooperation between lexer and parser.
- **Trade-off:** Preserving historical language behavior creates cross-stage coupling, recursive parser-state management, and difficult edge cases.
- **Ask:** What meaning will be lost if a boundary passes only a plain string?
- **Fit:** Interpreters, template engines, and transformations whose later stages depend on earlier semantic decisions.
- **Avoid:** Copying Bash's compatibility complexity into a new language or using string substitution where structured data is needed.
- **Check:** Follow quoted emptiness, nested substitutions, and assignment context through every stage; test semantic preservation at the boundary.

### Graphite — Buffering trades I/O pressure for memory and freshness

Source: Chris Davis, [“Graphite,” AOSA Volume 1, §§7.6–7.8](https://aosabook.org/en/v1/graphite.html).

- **Mechanism:** Carbon queues points by metric and writes batches to Whisper. Queries combine persisted points with queued points to expose recent data.
- **Trade-off:** Batching reduces write operations, but growing queues consume memory; memory pressure can further slow writes. Queue and operation limits make overload explicit.
- **Ask:** Does the bottleneck concern bytes, operations, or latency, and what happens when producers outrun storage?
- **Fit:** Telemetry workloads with declared freshness and loss policies.
- **Avoid:** Unbounded queues or treating query visibility as proof of durable storage.
- **Check:** Slow storage, fill queues, and verify memory bounds, recent-read behavior, and the chosen reject/drop policy. Measure before changing implementation language.

### Berkeley DB — Make durability ordering cross the cache boundary

Source: Margo Seltzer and Keith Bostic, [“Berkeley DB,” AOSA Volume 1, §§4.5–4.6](https://aosabook.org/en/v1/bdb.html).

- **Mechanism:** Separate buffer, lock, log, and transaction subsystems cooperate. Before flushing a dirty page, the buffer manager ensures the log record identified by that page's log sequence number is durable.
- **Trade-off:** Shared on-disk/in-memory page representation avoids conversion, but traversal and pinning pay buffer-management overhead even when data fits in RAM.
- **Ask:** Which ordering must survive a crash, and which component enforces it?
- **Fit:** Persistent transactional storage with a precisely defined acknowledgment and recovery contract.
- **Avoid:** Copying WAL machinery into ephemeral data paths or assuming layering removes cross-component invariants.
- **Check:** Crash between log and page writes, then recover. Verify the acknowledged durability contract with the real storage engine.

### HDFS — Separate metadata decisions from bulk data transfer

Source: Robert Chansler, Hairong Kuang, Sanjay Radia, Konstantin Shvachko, and Suresh Srinivas, [“The Hadoop Distributed File System,” AOSA Volume 1, §§8.1–8.2](https://aosabook.org/en/v1/hdfs.html).

- **Mechanism:** A NameNode manages namespace and block placement; clients transfer content directly to DataNodes. Metadata changes enter a synchronized journal before acknowledgment and replay from a checkpoint.
- **Trade-off:** Separating control and data traffic supports large transfers, while the chapter's single NameNode concentrates metadata capacity and availability concerns.
- **Ask:** Must bulk bytes pass through the coordinator, and what limits the metadata plane?
- **Fit:** Large-block storage and workloads benefiting from data locality.
- **Avoid:** Assuming the historical topology describes present HDFS, or importing it unchanged for small-object, low-latency workloads.
- **Check:** Test metadata recovery, DataNode loss, and coordinator unavailability separately. Replication does not replace a metadata recovery plan.

### SQLAlchemy — Make state scope and dependency order explicit

Source: Michael Bayer, [“SQLAlchemy,” AOSA Volume 2, §§20.8–20.9](https://aosabook.org/en/v2/sqlalchemy.html).

- **Mechanism:** An explicit Session scopes identity and pending changes. The unit of work constructs persistence actions and topologically orders dependencies before executing a flush.
- **Trade-off:** Coordinated persistence handles related objects, but requires visible session ownership and complex ordering logic. The chapter recounts the limitations of an earlier implicit global/thread-local scope.
- **Ask:** Which changes belong together, who owns their lifecycle, and which actions must precede others?
- **Fit:** Related mutations whose constraints require coordination within a transaction.
- **Avoid:** A global session shared across unrelated workflows or a dependency planner for a single straightforward write.
- **Check:** Insert related rows, exercise cyclic dependencies, and force failure. Verify scope, ordering, and rollback using the actual database.


## Attribution

Adapted from the credited AOSA chapters, edited by Amy Brown and Greg Wilson, under [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/). Original condensed wording, questions, applicability boundaries, and checks by the Architecture Distilled contributors. No endorsement is implied. [Source license](https://aosabook.org/en/license.html).
