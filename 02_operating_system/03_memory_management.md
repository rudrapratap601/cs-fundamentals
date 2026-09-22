# 3. Memory Management

[Index](README.md) · [Previous](02_synchronization_and_deadlocks.md) · [Next](04_files_io_system_calls.md)

## Virtual memory — core

Virtual memory gives processes virtual address spaces mapped to physical memory and other backing as needed. It supports isolation, controlled sharing, sparse address spaces, and demand loading. It is more than “using disk as extra RAM.”

Typical regions include executable code, static data, heap allocations, mapped files, and thread stacks. Exact layout and growth direction are implementation-dependent.

| Stack | Heap |
|---|---|
| Activation frames/local execution state in a typical implementation | Dynamically allocated objects/storage |
| Often automatic scope-based frame management | Managed by allocator, garbage collector, or explicit ownership |
| Limited per-thread size in many environments | Shared address-space region; allocation/lifetime rules vary |

Do not generalize language implementation details into “all objects are always on the heap” or “all local variables are always on the stack.” Optimizers and runtimes can change physical placement.

## Paging and address translation

Paging divides virtual memory into fixed-size **pages** and physical memory into equal-size **frames**. Page tables map virtual page numbers to frames and record properties such as permissions, presence, and dirty/reference state.

```text
Virtual address = virtual page number + page offset
Physical address = mapped frame number × page size + same offset
```

**Worked example:** Page size is 4 KiB = 4096 bytes. Virtual address 13,000 gives page number `13000 // 4096 = 3` and offset `13000 % 4096 = 712`. If page 3 maps to frame 9, physical address is `9 × 4096 + 712 = 37,576`.

For a 32-bit virtual address and 4 KiB pages, 12 bits are offset and 20 bits are page number. A single-level table with 4-byte entries needs `2^20 × 4 = 4 MiB` per full table, motivating hierarchical or other compact structures.

## TLB versus page fault — core

The **TLB** caches address translations.

- **TLB hit:** use the cached mapping, subject to permissions.
- **TLB miss:** perform a page-table lookup/walk; the page may already be in RAM.
- **Page fault:** translation/access requires OS handling, for example a nonresident page, copy-on-write, or an invalid/protected access.

Not every TLB miss causes a page fault, and not every page fault requires disk I/O. Some faults can be resolved by zero-filling a new page or changing a copy-on-write mapping; invalid accesses may terminate the process.

**Simplified effective access time:** Assume sequential TLB lookup, one-level table, no page faults, TLB lookup 10 ns, memory access 100 ns, and hit ratio 0.9.

```text
Hit cost = 10 + 100 = 110 ns
Miss cost = 10 + 100 + 100 = 210 ns
EAT = 0.9 × 110 + 0.1 × 210 = 120 ns
```

Actual systems use multi-level walks, caches, and overlapping operations; the formula depends on stated assumptions.

## Demand paging and replacement

On a resolvable nonresident-page fault, the OS validates access, finds a frame, possibly evicts a victim, writes back dirty data if necessary, loads/maps the needed page, updates translation state, and restarts the faulting operation.

| Algorithm | Victim selection | Main point |
|---|---|---|
| FIFO | Oldest loaded page | Simple; can show Belady's anomaly |
| Optimal | Page used farthest in the future | Benchmark; requires future knowledge |
| LRU | Least recently used page | Uses locality; exact tracking can be expensive |
| Clock/second chance | Approximate recency using reference bits | Practical compromise |

**Belady's anomaly:** More frames can increase faults under some policies, such as FIFO. It does not occur for stack algorithms such as LRU and Optimal.

### Worked FIFO trace

Three initially empty frames; references `1, 2, 3, 1, 4, 2, 5`. List frames in load order, oldest first.

| Reference | Frames afterward | Result |
|---|---|---|
| 1 | 1 | Fault |
| 2 | 1,2 | Fault |
| 3 | 1,2,3 | Fault |
| 1 | 1,2,3 | Hit; FIFO order unchanged |
| 4 | 2,3,4 | Fault; evict 1 |
| 2 | 2,3,4 | Hit |
| 5 | 3,4,5 | Fault; evict 2 |

Total: **5 faults**. Under LRU, hits change recency order, so the final evictions differ.

## Fragmentation and segmentation

- **Internal fragmentation:** wasted space inside an allocated unit, such as the unused end of a page.
- **External fragmentation:** free memory exists but is split into holes that cannot satisfy a contiguous request.
- **Paging:** avoids external fragmentation for page-frame allocation but can cause internal fragmentation and page-table overhead.
- **Segmentation:** variable-sized logical regions with base/limit-style translation in the classical model; can suffer external fragmentation.

**Thrashing:** Excessive paging dominates useful execution when active working sets do not fit available memory. Reducing multiprogramming, improving locality, or providing more memory can help; merely raising CPU frequency does not solve the cause.

**Copy-on-write:** Share physical pages until a write requires a private copy. It reduces copying for operations such as `fork()`, but is not ordinary permanently shared writable memory.

## Interview answers

**Can two processes use the same virtual address?** Yes; their page tables can map it to different physical frames.

**Is a page fault an error?** Sometimes; many are normal demand-allocation/loading events. Invalid or forbidden accesses are different cases.

**Why is locality important?** Temporal and spatial reuse improve cache/TLB hit rates and reduce working-set pressure.
