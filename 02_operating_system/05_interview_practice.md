# 5. OS Interview Practice

[Index](README.md) · [Previous](04_files_io_system_calls.md)

## Rapid recall

| Question | Answer checkpoint |
|---|---|
| Process versus thread? | Address-space isolation versus execution streams sharing process resources |
| Mode versus context switch? | Privilege transition versus changing execution context |
| Mutex versus semaphore? | Exclusive ownership versus permits/counting |
| Deadlock versus starvation? | Circular dependency with no progress versus one task repeatedly denied progress |
| TLB miss versus page fault? | Translation-cache miss versus OS-handled memory fault |
| Internal versus external fragmentation? | Waste within allocation versus separated free holes |
| Zombie versus orphan? | Uncollected termination status versus exited parent |
| Is `write()` durable? | Not necessarily; data may only be cached |

## Worked problems

### Round robin

All processes arrive at 0 in queue order P1, P2, P3. Bursts: 5, 3, 1. Quantum: 2. No I/O or switching overhead.

```text
0–2 P1 | 2–4 P2 | 4–5 P3 | 5–7 P1 | 7–8 P2 | 8–9 P1
```

| Process | Completion/turnaround | Waiting | Response |
|---|---:|---:|---:|
| P1 | 9 | 4 | 0 |
| P2 | 8 | 5 | 2 |
| P3 | 5 | 4 | 4 |

Average waiting is `13/3 = 4.33`; average response is `6/3 = 2`. Response and waiting time are not the same once tasks can be preempted.

### Address translation

**Question:** With 1 KiB pages, virtual address 2500, and virtual page 2 mapped to frame 7, find the physical address.

**Answer:** Offset `2500 − 2048 = 452`; physical address `7 × 1024 + 452 = 7620`.

### LRU replacement

**Question:** Three empty frames, references `1,2,3,1,4,2,5`. How many LRU faults?

**Answer:** Six. First three fault; 1 hits; 4 evicts 2; 2 evicts 3; 5 evicts 1. Unlike FIFO, a hit updates recency.

### Two-lock freeze

**Question:** A locks X then Y; B locks Y then X. Diagnose and fix.

**Answer:** Each can hold one lock while waiting for the other. Enforce one global acquisition order, such as X then Y, on every path. Use structured release so exceptions do not leak locks.

### Consumer wakes but queue is empty

**Question:** Why must a condition wait recheck the predicate?

**Answer:** Spurious wakeups or another consumer can invalidate the condition before the lock is reacquired. Test in a loop while holding the lock.

### Heavy disk activity, low useful CPU work

**Question:** What might be wrong when too many processes are running?

**Answer:** Thrashing is one possibility: active working sets exceed available memory. Check memory pressure and page-fault/I/O behavior rather than concluding from CPU utilization alone.

## Follow-up explanations

**Why doesn't shared memory remove IPC complexity?** It removes some copying/communication overhead, but participants must coordinate access and lifetime.

**Why doesn't more RAM solve a deadlock?** The issue can be a cycle involving locks or other resources, not lack of memory capacity.

**What happens on a page fault?** Validate access, allocate/find a frame, load or create content if appropriate, update mappings, restart; invalid access follows an error path.

**What makes an OS answer strong?** Distinguish abstraction from implementation, state scheduling/memory assumptions, and explain where waiting or overhead actually occurs.
