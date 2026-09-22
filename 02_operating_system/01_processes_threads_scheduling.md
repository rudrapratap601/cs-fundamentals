# 1. Processes, Threads, and Scheduling

[Index](README.md) · [Next](02_synchronization_and_deadlocks.md)

## OS and kernel — core

The OS manages CPU time, memory, devices, files, and protection. The kernel is its privileged core. User programs request privileged services through system calls.

- **User mode:** restricted execution; cannot directly perform arbitrary privileged operations.
- **Kernel mode:** privileged execution for kernel work.
- **System call:** deliberate request for a kernel service.
- **Interrupt:** commonly an asynchronous hardware event, such as device completion.
- **Exception:** synchronous event caused by an instruction, such as a fault. Terminology such as “trap” varies by architecture.

A monolithic kernel places many services in kernel space. A microkernel keeps a smaller privileged core and moves more services into user-space processes. The trade-offs include isolation, communication overhead, and implementation complexity.

## Program, process, thread

A program is code/data stored as an executable; a process is a running instance with resources and an address space; a thread is an execution stream within a process.

| Aspect | Process | Thread within a process |
|---|---|---|
| Address space | Normally isolated from other processes | Shares process address space |
| Resources | Own process-level resources | Shares code, heap, and many open-resource handles |
| Execution state | Contains one or more threads | Own registers, instruction pointer, stack, and thread-local state |
| Communication | IPC mechanisms | Shared memory plus synchronization |
| Failure isolation | Usually stronger | Memory corruption can affect the whole process |
| Switching | May require address-space changes | May avoid address-space changes; still has cost |

Threads are not always faster. Shared state adds synchronization, contention, and debugging costs. Separate thread stacks are not an access-control barrier: threads share the address space and can access memory if they have a valid reference/address.

## Process state and context switches

```text
New → Ready → Running → Terminated
        ↑       |
        |       ├─ preempted → Ready
        |       └─ waits for event → Blocked
        └──────────── event completes
```

A process/thread control structure records identity, execution context, scheduling state, and resource metadata. A context switch saves one execution context and restores another. It consumes CPU time and can disturb caches/TLBs.

**Mode switch versus context switch:** A system call enters kernel mode but may return to the same thread. A change of privilege does not necessarily switch the running thread.

**Concurrency** means tasks make overlapping progress; **parallelism** means tasks execute simultaneously on multiple execution units.

## CPU scheduling — core

Preemptive scheduling can interrupt a running task; non-preemptive scheduling waits until it yields, blocks, or finishes its burst.

| Algorithm | Main idea | Strength | Trade-off |
|---|---|---|---|
| FCFS | Arrival order | Simple | Convoy effect behind long jobs |
| SJF | Shortest next burst first | Minimizes average waiting under classic known-burst assumptions | Bursts must be estimated; long jobs may starve |
| SRTF | Preemptive shortest remaining time | Favors short remaining work | Preemption and estimation costs |
| Round robin | Time quantum per ready task | Responsiveness/fair sharing | Small quantum increases switching overhead |
| Priority | Highest-priority eligible task | Expresses urgency | Starvation; aging can help |
| Multilevel feedback queue | Move tasks among queues based on behavior | Balances interactive and CPU-heavy work | Policy/parameter complexity |

```text
Turnaround = completion time − arrival time
Response = first CPU start − arrival time
Waiting = time spent in ready queue
```

For problems with one CPU burst and no I/O, `waiting = turnaround − burst`. With I/O, also account for blocked time; do not count it as ready-queue waiting.

### Worked FCFS example

Assume one CPU, no I/O, and zero context-switch overhead.

| Process | Arrival | Burst | Start | Finish | Waiting | Response | Turnaround |
|---|---:|---:|---:|---:|---:|---:|---:|
| P1 | 0 | 5 | 0 | 5 | 0 | 0 | 5 |
| P2 | 1 | 3 | 5 | 8 | 4 | 4 | 7 |
| P3 | 2 | 1 | 8 | 9 | 6 | 6 | 7 |

```text
0        5      8  9
|   P1   |  P2  |P3|
```

Average waiting: `(0 + 4 + 6)/3 = 3.33` time units. Average turnaround: `19/3 = 6.33`.

## Process creation and IPC — follow-up

On Unix-like systems, `fork()` creates a child with a logically separate address space, often implemented using copy-on-write. `exec()` replaces the current process image; it does not create a new process by itself. `wait()` lets a parent collect child termination status.

- **Zombie:** terminated child whose status has not been collected.
- **Orphan:** process whose parent exited; reparenting details depend on the OS.
- **Pipe:** byte-stream IPC, often between related processes.
- **Message queue:** structured message exchange.
- **Shared memory:** direct shared data access; requires synchronization.
- **Socket:** local or network communication endpoint.
- **Signal:** event notification with constrained handling rules.

## Interview answers

**Why are context switches expensive?** State saving/restoring plus indirect cache/TLB effects; exact cost depends on hardware, OS, and workload.

**Can one core run concurrent tasks?** Yes, by interleaving. It cannot execute those tasks in parallel on that single execution resource.

**What causes starvation?** Repeatedly favoring other tasks. Aging raises waiting tasks' priority over time; it addresses starvation, not every deadlock.
