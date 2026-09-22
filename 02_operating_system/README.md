# Operating Systems

[Home](../README.md)

**Objective:** Explain how an OS manages execution, concurrency, memory, and persistent storage.

## Reading order

1. [Processes, threads, and scheduling](01_processes_threads_scheduling.md)
2. [Synchronization and deadlocks](02_synchronization_and_deadlocks.md)
3. [Memory management](03_memory_management.md)
4. [Files, I/O, and system calls](04_files_io_system_calls.md)
5. [Interview practice](05_interview_practice.md)

Prioritize process/thread differences, context switches, scheduling metrics, races, mutexes/semaphores, deadlocks, paging, virtual memory, and page faults.

**Mental model:** The OS provides abstractions—processes, virtual address spaces, files—while multiplexing hardware and enforcing protection. An abstraction can hide mechanics but cannot remove the cost of CPU work, synchronization, or I/O.
