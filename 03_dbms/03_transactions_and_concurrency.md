# 3. Transactions and Concurrency

[Index](README.md) · [Previous](02_sql_and_query_patterns.md) · [Next](04_indexes_storage_scaling.md)

## ACID — core

A transaction groups operations into a unit with defined commit/rollback behavior.

| Property | Meaning | Transfer example |
|---|---|---|
| Atomicity | All transactional changes take effect or none do | Debit and credit succeed together |
| Consistency | Correct transactions preserve declared/domain invariants | Valid balances and accounting rules remain valid |
| Isolation | Controls interference between concurrent transactions | Concurrent transfers do not corrupt updates |
| Durability | A successful commit survives failures covered by the system's guarantees | Committed transfer survives recovery |

The database cannot infer every business rule. Constraints and correct transaction logic are needed for consistency. Durability depends on configured persistence and the failure model, not merely calling a function named commit.

## Transfer example

Assume an Account table with primary key `account_id` and non-NULL numeric `balance`. Transaction-start and row-locking syntax vary by engine.

```sql
BEGIN;

UPDATE Account
SET balance = balance - 100
WHERE account_id = 1 AND balance >= 100;

-- Application: require exactly one affected row; otherwise ROLLBACK.

UPDATE Account
SET balance = balance + 100
WHERE account_id = 2;

-- Application: require exactly one affected row; otherwise ROLLBACK.
-- COMMIT only after both required updates succeed.
COMMIT;
```

The comments are required application control flow, not executable checks. A transaction alone does not make an unchecked missing-account update correct. Concurrent transfers can deadlock; consistent lock ordering and retrying an aborted transaction help. State isolation assumptions when reasoning about concurrent behavior.

## Concurrency anomalies

| Anomaly | Example |
|---|---|
| Dirty read | T2 reads T1's uncommitted balance; T1 rolls back |
| Non-repeatable read | T1 reads one row twice; T2 commits a change between reads |
| Phantom | Repeating a predicate query sees a changed matching row set |
| Lost update | Two tasks read 10, each writes computed value 11, losing one increment |
| Write skew | Transactions read overlapping state and update different rows, jointly breaking an invariant |

**Write-skew example:** Two doctors are on call. Each transaction observes the other on call and marks itself off call. Each writes a different row; together they leave nobody on call. Snapshot isolation alone can allow this.

## Isolation levels

The table describes classic minimum expectations; engines can provide stronger behavior or use different implementations.

| Level | Dirty reads | Non-repeatable reads | Phantoms |
|---|---|---|---|
| Read uncommitted | May occur | May occur | May occur |
| Read committed | Prevented | May occur | May occur |
| Repeatable read | Prevented | Prevented | May occur under the classic definition |
| Serializable | Prevented | Prevented | Prevented |

Real engines differ: repeatable read may use a snapshot and also prevent ordinary repeated-read phantoms while still permitting other serialization anomalies. Do not infer all anomaly behavior from the level name alone.

**Serializable** means committed concurrent execution is equivalent to some serial ordering. It need not literally execute one transaction at a time. It may block or abort conflicting transactions, so applications must be prepared to retry.

## Locks, 2PL, MVCC — core and follow-up

- Shared locks typically support reads; exclusive locks protect writes. Compatibility depends on lock type/granularity.
- Row locks allow more concurrency than coarse table locks but increase tracking overhead.
- Predicate/range locking can protect a set of possible matching rows, including insertion gaps where supported.
- **Two-phase locking (2PL):** growing phase acquires locks; shrinking phase releases them and acquires no new locks. Under its assumptions it provides conflict serializability but can deadlock.
- **Strict 2PL:** retains exclusive locks until commit/abort, preventing reads/writes of uncommitted writes and simplifying recovery. Some descriptions use stronger all-locks-until-end variants; state the convention.
- **MVCC:** retains row versions so reads can see an appropriate snapshot. Readers often avoid blocking writers, but writes still need coordination and old versions require cleanup.

**Optimistic concurrency:** Read a version, then conditionally update:

```sql
UPDATE Document
SET content = :new_content, version = version + 1
WHERE document_id = :id AND version = :old_version;
```

This uses named placeholder notation; driver syntax differs. Zero affected rows means a missing row or concurrent change and requires handling. Optimistic does not mean “no concurrency control.”

### Serializability graph

For conflict serializability, create an edge Ti → Tj when Ti's operation precedes and conflicts with Tj's on the same item (different transactions, at least one write). A cycle means the schedule is not conflict-serializable; acyclicity gives a serial order.

Example: `R1(X), W2(X), R2(Y), W1(Y)` creates T1 → T2 on X and T2 → T1 on Y, so the graph has a cycle.

## Deadlocks and recovery

Transactions can deadlock when each waits for locks held by another. The database may detect a wait-for cycle and abort a victim. The application should retry the entire appropriate transaction, not assume every partial statement can be safely repeated independently.

**Write-ahead logging:** Relevant log records reach durable storage before modified data pages are persisted in a way that requires those records for recovery. A durable commit can rely on persisted log information even if data pages are flushed later.

Recovery may redo committed changes and undo incomplete work according to the engine's design. Checkpoints reduce recovery work; they do not mean the log is unnecessary or that every transaction has finished. Backups and replication solve different failure problems.

## Interview answers

**ACID consistency versus CAP consistency?** ACID consistency concerns invariants; CAP consistency refers to a single-copy/linearizable view in its distributed-system model.

**Does MVCC eliminate locks?** No. It changes how versions are read and reduces some conflicts; write coordination and other locks remain.

**What if the client loses connection during commit?** Outcome may be uncertain. Reconcile using application identifiers/status and design retries to avoid duplicate business effects.
