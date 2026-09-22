# Revision Plan

[Home](README.md)

## Fourteen study sessions

Allow roughly 60–90 minutes per session; split sessions if a topic is unfamiliar.

| Session | Study | Recall task |
|---|---|---|
| 1 | OOP chapters 1–2 | Explain the four pillars using one example |
| 2 | OOP chapters 3–4 | Design a notification service |
| 3 | OOP practice | Explain trade-offs without notes |
| 4 | DBMS chapter 1 | Normalize a table and identify keys |
| 5 | DBMS chapter 2 | Write joins, aggregates, and ranking queries |
| 6 | DBMS chapters 3–4 | Explain a transaction and an index choice |
| 7 | DBMS practice | Solve SQL tasks on paper |
| 8 | OS chapters 1–2 | Calculate scheduling metrics and fix a race |
| 9 | OS chapters 3–4 | Trace address translation and a file read |
| 10 | OS practice | Analyze deadlock and page replacement |
| 11 | CN chapters 1–2 | Draw the layers and calculate a subnet |
| 12 | CN chapters 3–4 | Explain TCP and a browser request |
| 13 | CN practice | Diagnose protocol scenarios |
| 14 | Mixed mock interview | Trace an order request across all four subjects |

## Active recall

After each session, record three questions you could not answer. Revisit them after about one day, three days, and one week. Spend more time reconstructing explanations than rereading them.

Score answers: **0** = cannot explain; **1** = definition only; **2** = mechanism and example; **3** = trade-off and follow-up. Revisit scores 0–1 first.

## Final checklist

### Computer Networks

- [ ] Map protocols to layers and distinguish frames, packets, and segments.
- [ ] Compute subnet ranges and explain local versus routed delivery.
- [ ] Distinguish ARP, DHCP, DNS, and NAT.
- [ ] Explain TCP setup, reliability, flow control, and congestion control.
- [ ] Trace HTTPS, caching, and browser session state.

### Operating Systems

- [ ] Compare processes and threads, including sharing and isolation.
- [ ] Calculate response, waiting, and turnaround times.
- [ ] Fix races and explain deadlock handling.
- [ ] Translate virtual addresses and distinguish TLB misses from page faults.
- [ ] Explain page replacement, file descriptors, and durability.

### DBMS

- [ ] Identify keys and normalize through BCNF.
- [ ] Write SQL with joins, NULLs, grouping, and window functions.
- [ ] Explain ACID, isolation anomalies, locking, and MVCC.
- [ ] Choose an index and explain its write cost.
- [ ] Compare replication, partitioning, and sharding.

### OOP

- [ ] Explain the four pillars with examples.
- [ ] Distinguish overloading, overriding, and field access.
- [ ] Compare interfaces, abstract classes, inheritance, and composition.
- [ ] Apply SOLID without merely expanding the acronym.
- [ ] Explain equality, immutability, copying, and resource ownership.
- [ ] Justify a design pattern with a concrete problem.

## Mixed mock interview

**Prompt:** A user clicks “Place order.” Explain the system from browser to storage.

A good answer connects DNS and HTTPS; transport; server processes and threads; collaborating application objects; validation and authorization; a database transaction for orders and inventory; constraints and concurrent updates; indexes and storage; and the response.

**Follow-ups and checkpoints:**

- Two users buy the last item: coordinate the update transactionally; do not rely on an unchecked read-then-write.
- The process crashes after commit but before replying: the order may exist; a timeout does not imply failure. Consider an idempotency key and a status lookup.
- DNS succeeds but a connection fails: resolution and transport are separate steps; investigate route, endpoint, port, and filtering.
- A request is slow: separate network delay, CPU scheduling, application work, database execution, and storage I/O.

For last-minute revision, use subject practice files and your incorrect answers. Rehearse one subnet calculation, one scheduling example, one normalization example, one SQL query involving ties, and one object design.
