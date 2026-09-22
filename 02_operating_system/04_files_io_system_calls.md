# 4. Files, I/O, and System Calls

[Index](README.md) · [Previous](03_memory_management.md) · [Next](05_interview_practice.md)

## System-call boundary

A library function is not necessarily a system call. For example, a buffered output function can accumulate data in user space before issuing an OS write. A system call crosses into the kernel to request services such as opening files, creating processes, mapping memory, or using sockets.

An interrupt can notify the kernel of device completion. **DMA** lets a device transfer data to/from memory without the CPU copying each byte; the CPU still configures operations and handles coordination/completion.

## File systems — core

A file system organizes names, metadata, file contents, free space, and access rules on storage.

In a typical Unix-like model:

- A directory maps names to file identifiers such as inode numbers.
- An inode stores metadata and references to data, not normally the filename itself.
- A file descriptor is a process-local integer handle.
- Descriptors reference kernel open-file state, which includes information such as offset and flags. Duplicated or inherited descriptors may share that state.

**Example:** `open()` resolves a pathname and returns a descriptor; `read()` uses that descriptor and typically advances its offset; `close()` releases the reference. Exact APIs differ across operating systems.

## Links and deletion

| Hard link | Symbolic link |
|---|---|
| Another directory entry for the same underlying file | Separate object containing a target path |
| Usually limited to the same file system | Can point across file systems |
| Target data persists while links/open references require it | Can dangle if its target path disappears |
| Directory hard links are generally restricted | Can reference directories |

Removing a filename is not always immediate deletion of underlying contents. On Unix-like systems, an unlinked file can remain accessible through an open descriptor until remaining references are released. Windows sharing/deletion semantics differ; state the platform when discussing this behavior.

## Allocation and storage

| Allocation approach | Benefit | Cost |
|---|---|---|
| Contiguous | Efficient sequential/random access | Growth and external-fragmentation issues |
| Linked | Flexible growth | Poor direct access; pointer overhead |
| Indexed | Locate blocks through an index | Index storage and lookup overhead |
| Extents | Track runs of contiguous blocks | More complex allocation; fragmentation still possible |

HDDs have seek and rotational costs. Classical disk schedulers include FCFS, SSTF, SCAN, and C-SCAN. SSTF can starve distant requests; SCAN sweeps in one direction and then reverses; C-SCAN services one direction and returns to the start. SSDs have no mechanical seek, so HDD scheduling intuitions do not directly transfer.

## Buffering, caching, and durability

- **Buffering:** temporarily holds data to accommodate transfer sizes/rates.
- **Caching:** retains copies to avoid repeated expensive retrieval.
- **Spooling:** queues work for a device/service, such as print jobs.
- **Page cache:** OS-managed cached file data, often used by ordinary reads/writes.

A successful `write()` may mean data reached a kernel cache, not durable storage. APIs such as `fsync()` request persistence according to platform/file-system guarantees. Correct crash-safe updates can require syncing file data and relevant directory metadata; one generic sequence is not universal across all systems.

**Journaling** records intended file-system changes so recovery can restore consistency after a crash. What is journaled—metadata, data, or both—depends on the mode. A consistent file system does not automatically guarantee that the application's latest data is durable or that a multi-file operation is atomic.

## Blocking, nonblocking, asynchronous I/O

- **Blocking:** the calling thread waits when the operation cannot progress.
- **Nonblocking:** the call returns promptly, potentially indicating “would block”; readiness mechanisms can tell the program when to retry.
- **Asynchronous:** submission and completion are separated; the application receives completion later.

These are related but not interchangeable concepts. Nonblocking I/O alone does not guarantee that an entire application never waits.

## Interview answers

**Why does a second file read often run faster?** Data or metadata may be cached; measure before assuming storage speed changed.

**File descriptor versus inode?** A descriptor is a process handle to an open resource; an inode represents file metadata in an inode-based file system.

**Does closing a file guarantee durability?** Do not assume it does. Durability requires the relevant OS/file-system guarantees and synchronization operations.
