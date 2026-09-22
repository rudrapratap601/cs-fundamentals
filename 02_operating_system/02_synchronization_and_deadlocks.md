# 2. Synchronization and Deadlocks

[Index](README.md) · [Previous](01_processes_threads_scheduling.md) · [Next](03_memory_management.md)

## Race conditions — core

A race condition occurs when correctness depends on uncontrolled timing. A data race is a more specific language/memory-model concept involving unsynchronized conflicting accesses; not every logical race is a data race.

`counter++` can involve read, increment, and write:

```text
Initially counter = 5
Thread A reads 5
Thread B reads 5
A writes 6
B writes 6
Final result 6, but two increments should produce 7.
```

A **critical section** accesses shared state that requires coordination. Classical goals are mutual exclusion, progress, and bounded waiting. Individual primitives do not necessarily promise fairness or bounded waiting.

## Synchronization primitives

| Primitive | Use | Important limitation |
|---|---|---|
| Mutex | Exclusive ownership of a critical section | Usually only the owner may unlock |
| Semaphore | Count available permits/resources | Does not inherently have mutex-style ownership |
| Binary semaphore | Permit count bounded to 0/1 by usage/design | Not identical to a mutex's ownership semantics |
| Spinlock | Busy-wait for a short critical section | Wastes CPU if wait is long or owner cannot run |
| Condition variable | Wait for a predicate under a lock | Wakeup is not proof that the predicate is true |
| Read-write lock | Concurrent readers or exclusive writer | Starvation/overhead depend on policy |
| Atomic operation | Indivisible operation on specific state | Does not automatically protect multi-variable invariants |

Mutexes may block or use hybrid spinning/blocking internally. Choose based on expected contention and wait duration, not a blanket “one is faster.”

### Condition-variable pattern

```text
lock(mutex)
while queue is empty:
    condition.wait(mutex)  // atomically releases lock; reacquires before return
item = queue.remove()
unlock(mutex)
```

Use `while`, not `if`: wakeups can be spurious, or another consumer may consume the item before this thread reacquires the lock. Producers change the predicate under the appropriate lock and notify waiters according to the condition-variable API's rules.

### Bounded producer-consumer buffer

For capacity N, initialize `empty=N`, `full=0`, and a mutex unlocked.

```text
Producer:                     Consumer:
wait(empty)                   wait(full)
lock(mutex)                   lock(mutex)
insert item                   remove item
unlock(mutex)                 unlock(mutex)
signal(full)                  signal(empty)
```

The permit controls capacity; the mutex protects the buffer's data structure. Waiting for a permit while holding the buffer mutex can prevent the other side from making progress.

## Deadlock — core

Deadlock is a set of tasks waiting indefinitely for events/resources that only other tasks in the set can provide.

The Coffman conditions for resource deadlock are:

1. **Mutual exclusion:** some resource cannot be shared.
2. **Hold and wait:** a task holds resources while waiting for more.
3. **No preemption:** resources cannot be forcibly reclaimed in the relevant model.
4. **Circular wait:** a cycle of tasks waits on one another.

**Example:** A holds L1 and waits for L2; B holds L2 and waits for L1. Requiring every task to acquire L1 before L2 breaks circular wait.

For one instance of each resource type, a resource-allocation graph cycle implies deadlock. With multiple instances, a cycle alone need not prove deadlock.

| Strategy | How it works | Cost/limitation |
|---|---|---|
| Prevention | Break at least one necessary condition | May reduce concurrency or utilization |
| Avoidance | Grant requests only if state stays safe | Requires information about future maximum needs |
| Detection/recovery | Allow, detect, then abort/preempt/roll back | Recovery can be costly or impossible for some resources |
| Ignore in general case | Do not provide general detection | Applications still need handling for relevant cases |

### Safe state and Banker's algorithm — follow-up

A state is safe if some completion sequence lets every task obtain its remaining maximum demand and finish. **Unsafe does not mean already deadlocked.**

For one resource type, suppose 3 units are available:

| Task | Allocated | Maximum | Remaining need |
|---|---:|---:|---:|
| P1 | 1 | 3 | 2 |
| P2 | 2 | 5 | 3 |
| P3 | 3 | 7 | 4 |

P1 can finish: available becomes `3+1=4`. Then P2 can finish: available becomes `4+2=6`. Then P3 can finish. The state is safe. The safety simulation adds the task's existing allocation back when it finishes; its simulated remaining need is borrowed and returned, so the net increase is the old allocation.

## Related failures

- **Starvation:** one task waits indefinitely while others progress.
- **Livelock:** tasks keep responding/changing state but do no useful work, such as endlessly yielding to each other.
- **Priority inversion:** a high-priority task waits on a lock held by a lower-priority task, potentially delayed by medium-priority work. Priority inheritance can mitigate it.

## Interview answers

**Does an atomic counter make a bank transfer safe?** No. A transfer involves a relationship between multiple values; single-variable atomicity is insufficient.

**Does a timeout eliminate deadlock?** It may enable recovery if the task can safely release/roll back; it does not by itself guarantee correctness or prevent repeated conflicts.

**How do you prevent two-lock deadlock?** Use a consistent global acquisition order and release resources on all exit paths. Avoid holding locks across unnecessary slow/blocking operations.
