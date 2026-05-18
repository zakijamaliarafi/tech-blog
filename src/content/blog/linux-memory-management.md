---
heroImage: '/linux-memory-management.svg'
title: 'Demystifying Linux Memory Management and Swap'
description: 'Understand how the Linux kernel manages RAM, handles page caching, utilizes swap space, and resolves Out-Of-Memory (OOM) situations.'
pubDate: 'Apr 29 2026'
---

"Why does Linux use all my RAM?" This is one of the most common questions from new Linux users. When running `free -h`, it often looks like the system is out of memory, even when barely any applications are running. 

Understanding Linux memory management is crucial for diagnosing performance issues and properly sizing your servers.

## Virtual Memory and Paging

Linux uses a **Virtual Memory** system. When a process asks for memory, the kernel doesn't immediately hand over physical RAM. Instead, it provides a virtual memory address. Only when the process actually tries to write to that address does the kernel allocate a physical "page" of RAM (typically 4KB in size) and maps it to the virtual address.

This allows the system to overcommit memory safely and ensures processes are isolated from each other.

## The Page Cache: "Free Memory is Wasted Memory"

If you read a file from the disk, Linux copies the file's contents into RAM. If you close the file, Linux *does not* clear that RAM immediately. It keeps the data in memory as a **Page Cache**. 

If you open the file again, it reads instantly from RAM rather than the slow disk. If an application suddenly needs memory, the kernel instantly drops the oldest page cache to free up space.

This is why `free -h` might show very little "free" memory. You should look at the "available" column, which represents memory available for starting new applications (free + reclaimable cache).

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:            15Gi       3.2Gi       1.1Gi       120Mi        11Gi        11Gi
Swap:          2.0Gi       0.0Ki       2.0Gi
```
In this example, 11GB is being used for cache, but 11GB is also instantly available if applications need it.

## Swap Space: The Safety Net

Swap is space on a disk drive used as an extension of physical RAM. When physical RAM is filling up, the kernel looks for inactive memory pages (data that hasn't been accessed recently) and "swaps" them out to the disk.

### The Swappiness Parameter

You can control how aggressively Linux swaps using the `vm.swappiness` sysctl parameter (value from 0 to 100).
- `100`: Aggressively swap inactive memory to disk to keep RAM free for page caches.
- `0` or `1`: Only swap as an absolute last resort to prevent crashing.
- `60`: The default on most distros.

For database servers (like PostgreSQL or MySQL), it is highly recommended to set swappiness to a low value (like 1 or 10) to ensure database indexes stay in fast physical RAM:
```bash
sudo sysctl vm.swappiness=10
```

## The OOM Killer

What happens when you run completely out of physical RAM and Swap? The system could lock up entirely. To prevent this, Linux invokes the **OOM (Out-Of-Memory) Killer**.

The OOM Killer scans all running processes and calculates an `oom_score`. It then brutally kills the process with the highest score to free up memory instantly. The score is largely based on how much memory the process is using, but also on how long it has been running and its privilege level.

You can adjust an application's susceptibility to the OOM killer using the `oom_score_adj` file in the `/proc` filesystem.

## Tools for Memory Inspection

1. `top` / `htop`: Good for real-time process monitoring.
2. `vmstat 1`: Displays virtual memory statistics, swapping activity, and CPU usage every 1 second.
3. `smem`: A tool that accurately calculates PSS (Proportional Set Size) to show exactly how much memory a process is truly using, accounting for shared libraries.

By understanding how caching, swapping, and the OOM killer interact, you can tune your Linux servers to run workloads smoothly without panic when RAM usage hits 95%.
