# 4. Indexes, Storage, and Scaling

[Index](README.md) · [Previous](03_transactions_and_concurrency.md) · [Next](05_interview_practice.md)

## Indexes — core

An index is an auxiliary access structure that helps locate rows without scanning the whole table. It consumes storage and increases write work because inserts, deletes, and relevant updates must maintain it.

| Structure | Useful for | Trade-off |
|---|---|---|
| B+ tree | Equality, range scans, ordered access | Updates/splits and additional storage |
| Hash index | Equality lookup | Generally unsuitable for ordered range access |

B+ trees have high fan-out, reducing levels/page reads. Internal nodes guide searches and leaves hold indexed entries; linked leaves support ranges. Actual entry contents depend on engine and index type.

**Clustered organization** stores/organizes table data around an index key in systems that support it. **Secondary indexes** point to rows or primary/clustered keys. “Primary key always means physically clustered” is not portable. Even clustered storage does not guarantee query output order without ORDER BY.

## Composite and covering indexes

For an index on `(department_id, salary)`:

```sql
CREATE INDEX idx_employee_department_salary
ON Employee(department_id, salary);
```

It often helps `WHERE department_id = ?` and `WHERE department_id = ? AND salary >= ?`. A salary-only predicate generally cannot use the leading key in the same way; engines may still perform an index scan, skip scan, or other optimization. Avoid “can never use this index.”

**Leftmost-prefix intuition:** Entries are first grouped by department, then ordered by salary within a department. Equality on leading columns plus a range/order on the next column is a common useful shape.

A **covering index** contains everything needed by a query, potentially avoiding table lookups. Index-only behavior can also depend on visibility checks and engine storage rules. Extra included/indexed columns consume space and add maintenance costs.

## Query performance workflow

1. Confirm the query's intended result, row counts, and selectivity.
2. Inspect a plan with the engine's EXPLAIN tooling. Execution-analysis variants can actually run the query, including writes; use them appropriately.
3. Compare estimated and actual rows where safely available; stale statistics can mislead the optimizer.
4. Look for large scans, join multiplication, sorting, temporary spills, and repeated lookups.
5. Consider query shape, appropriate indexes, statistics, and data layout.
6. Measure the target workload and include write/storage cost.

**Sargability:** Predicates that match an index's searchable form can be easier to optimize. For a timestamp column, a range such as `created_at >= :start AND created_at < :end` is often more index-friendly than applying a date-extraction function to every row, unless an appropriate expression index exists. Placeholder/date syntax is engine-specific.

The optimizer may correctly choose a table scan when most rows qualify, the table is small, or indexed lookups would create expensive random I/O. An unused index is not automatically an optimizer bug.

## Join algorithms — follow-up

| Algorithm | Typical fit | Cost factors |
|---|---|---|
| Nested loop | Small outer input or efficient indexed inner lookup | Outer rows × inner lookup cost |
| Hash join | Equality joins with suitable memory | Build/probe work; spills when memory is insufficient |
| Sort-merge join | Ordered inputs or large sortable sets | Sorting cost and sequential merge |

SQL join semantics and physical join algorithms are different concepts. An INNER JOIN does not specify whether the engine uses hashing or nested loops.

## Storage, OLTP, and OLAP

- **Row-oriented storage:** keeps a row's fields near each other; often fits transactional point reads/writes.
- **Column-oriented storage:** groups column values; often fits analytical scans and compression when reading a few columns across many rows.
- **OLTP:** many short operational transactions.
- **OLAP:** analytical aggregations over substantial datasets.
- **LSM-based storage, follow-up:** buffers writes and merges sorted structures through compaction. It can improve write handling but introduces read, write, and space amplification trade-offs.

These are workload tendencies, not universal product labels.

## Replication, partitioning, and sharding

| Technique | Meaning | Main concern |
|---|---|---|
| Replication | Keep copies of data | Lag, failover, consistency, write coordination |
| Horizontal partitioning | Split rows into subsets | Partition key and pruning |
| Vertical partitioning | Split columns into related structures | Reconstruction and access patterns |
| Sharding | Distribute partitions across nodes | Cross-shard joins/transactions and hot shards |

Replication does not automatically scale writes or replace backups: accidental deletion can replicate too. Asynchronous replicas can serve stale data; read-after-write requirements may need routing/coordination. Shard keys should consider distribution and query locality, not only uniqueness.

## CAP and database families — follow-up

During a network partition, a distributed system cannot guarantee both linearizable consistency and availability as defined by CAP for all relevant requests. “Pick any two forever” is misleading; partition handling forces the specific trade-off. CAP availability means every request received by a non-failing node eventually gets a response under the model, not merely a high uptime percentage.

Relational, document, key-value, graph, and wide-column models suit different access/relationship needs. “NoSQL has no schema, no joins, and no transactions” is an overgeneralization; features differ. Choose based on data relationships, access patterns, consistency, scale, and operational constraints.

## Interview answers

**Why not index every column?** Storage and write amplification, maintenance overhead, and limited usefulness for some predicates.

**Why can low-selectivity predicates favor scans?** Retrieving a large fraction of rows through an index can cost more than sequentially scanning them.

**Replication versus sharding?** Copies of data versus distribution of data subsets. Systems often combine both.
