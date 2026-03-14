# schedx-xv6
<div align="center">

<br/>

```
 ___     _          _ _  __
/ __| __| |_  ___ __| \ \/ /
\__ \/ _| ' \/ -_) _` |>  <
|___/\__|_||_\___\__,_/_/\_\
```

# xv6-schedx

**A more capable xv6 — built by Team SchedX**

[![Status](https://img.shields.io/badge/status-in%20progress-orange?style=flat-square)](https://github.com)
[![xv6](https://img.shields.io/badge/base-xv6%20RISC--V-blue?style=flat-square)](https://github.com/mit-pdos/xv6-riscv)
[![Course](https://img.shields.io/badge/course-Operating%20Systems-green?style=flat-square)](https://github.com)
[![License](https://img.shields.io/badge/license-MIT-gray?style=flat-square)](LICENSE)

*Operating Systems Design Course Project — Spring 2026*

<br/>

</div>

---

## Overview

This project extends the [MIT xv6 RISC-V](https://github.com/mit-pdos/xv6-riscv) operating system with two foundational features absent from the original: a **Multi-Level Feedback Queue (MLFQ) CPU scheduler** and a **kernel thread subsystem**. Together, they give xv6 a scheduling model and a concurrency primitive that meaningfully reflect how modern operating systems like Linux actually operate.

xv6 is intentionally minimal — it is built for teaching, not production. We took two of its most significant gaps and closed them properly, touching the scheduler loop, the timer interrupt, the process table, the context-switch path, and the memory allocator along the way.

---

## Features

### 🗂️ Feature 1 — Multi-Level Feedback Queue Scheduler

The default xv6 scheduler is pure round-robin. It treats a shell waiting for a keypress the same as a background number-crunching job. MLFQ fixes that.

**How it works:**
- New processes enter at **Queue 0** (highest priority, shortest quantum)
- If a process exhausts its quantum without yielding, it gets **demoted** to the next queue — the scheduler infers it is CPU-bound
- Processes that yield early (I/O-bound, interactive) **stay near the top** and remain responsive
- A **periodic priority boost** every `S` ticks resets all processes back to Queue 0, preventing starvation

This is the exact MLFQ algorithm described in the [OSTEP textbook](https://pages.cs.wisc.edu/~remzi/OSTEP/), implemented from scratch in xv6's scheduler loop.

**New system calls:**

| System Call | Description |
|---|---|
| `setpriority(pid, queue)` | Manually assign a process to a queue level (0–3) |
| `getpriority(pid)` | Read the current queue level of a process |

**Files modified:** `proc.h`, `proc.c`, `trap.c`, `syscall.c`, `sysproc.c`, `defs.h`

---

### 🧵 Feature 2 — Kernel Thread Support (kthread)

xv6 has no concept of threads. Every process is isolated, and there is no way for two units of execution to share memory. This feature adds that capability.

**How it works:**
- `kthread_create()` spawns a new thread that **shares the calling process's page table** — same virtual address space, same heap, same globals
- Each thread gets its own **private kernel stack** and its own slot in the thread table
- Threads are scheduled by MLFQ alongside regular processes — they are first-class schedulable entities
- `kthread_join()` blocks until the target thread finishes
- `kthread_exit()` tears down the thread cleanly without disturbing the rest of the process

**New system calls:**

| System Call | Description |
|---|---|
| `kthread_create(func, arg)` | Spawn a new kernel thread in the current process's address space |
| `kthread_join(tid)` | Block until the specified thread exits |
| `kthread_exit()` | Terminate the calling thread |

**Files modified:** `proc.h`, `proc.c`, `vm.c`, `kalloc.c`, `syscall.c`, `sysproc.c`, `defs.h`

---

## System Calls Summary

| # | System Call | Feature | Description |
|---|---|---|---|
| 1 | `setpriority(pid, queue)` | MLFQ | Set a process's queue level |
| 2 | `getpriority(pid)` | MLFQ | Get a process's current queue level |
| 3 | `kthread_create(func, arg)` | Kthread | Spawn a thread in shared address space |
| 4 | `kthread_join(tid)` | Kthread | Wait for a thread to finish |
| 5 | `kthread_exit()` | Kthread | Terminate the calling thread |

---

## Team

| Member | Role |
|---|---|
| **MD Sakib Sarker** | MLFQ Scheduler — `proc.c`, `trap.c`, system calls |
| **Ahnaf Tausif Nehal** | Kernel Threads — `vm.c`, `kalloc.c`, thread table, system calls |

---

## Project Structure

```
xv6-schedx/
├── kernel/
│   ├── proc.h          # Process + thread table structs (modified)
│   ├── proc.c          # MLFQ scheduler + thread table logic (modified)
│   ├── trap.c          # Timer interrupt — demotion & priority boost (modified)
│   ├── vm.c            # Shared page table + thread stack allocation (modified)
│   ├── kalloc.c        # Physical memory allocator (modified)
│   ├── syscall.c       # System call dispatch table (modified)
│   └── sysproc.c       # System call implementations (modified)
├── user/
│   ├── mlfqtest.c      # MLFQ scheduler test — CPU time distribution
│   └── kthreadtest.c   # Kthread test — shared counter with 4 threads
└── README.md
```

---

## Build & Run

> **Prerequisites:** RISC-V toolchain and QEMU. Follow the [xv6 setup guide](https://pdos.csail.mit.edu/6.828/2023/tools.html) if you haven't already.

```bash
# Clone the repo
git clone https://github.com/Ahnaf72/xv6-schedx.git
cd schedx-xv6

# Build and run in QEMU
make qemu
```

---

## Testing

```bash
# Test MLFQ — spawns processes at different queue levels,
# measures CPU time distribution over 10 seconds
$ mlfqtest

# Test Kthread — spawns 4 threads incrementing a shared counter,
# joins all threads, verifies final value
$ kthreadtest
```

> ⚠️ Tests will be added as implementation progresses.

---

## Implementation Progress

- [ ] MLFQ — Queue structure in `proc.h`
- [ ] MLFQ — Rewrite `scheduler()` in `proc.c`
- [ ] MLFQ — Demotion logic in timer interrupt (`trap.c`)
- [ ] MLFQ — Periodic priority boost
- [ ] MLFQ — `setpriority()` and `getpriority()` system calls
- [ ] MLFQ — `mlfqtest` user program
- [ ] Kthread — Thread table in `proc.h` and `proc.c`
- [ ] Kthread — Shared page table + private kernel stack (`vm.c`, `kalloc.c`)
- [ ] Kthread — `kthread_create()`, `kthread_join()`, `kthread_exit()` system calls
- [ ] Kthread — `kthreadtest` user program
- [ ] Integration — MLFQ scheduling threads and processes together
- [ ] Final testing and cleanup

---

## References

- [xv6 RISC-V Source](https://github.com/mit-pdos/xv6-riscv)
- [xv6 Book (MIT)](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)
- [OSTEP — Scheduling: MLFQ (Chapter 8)](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf)
- [OSTEP — Concurrency: Threads (Chapter 26)](https://pages.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)

---

<div align="center">

*Team SchedX &nbsp;·&nbsp; Operating Systems, Spring 2026*

</div>
