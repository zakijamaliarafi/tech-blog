---
title: 'Understanding Linux Control Groups (cgroups)'
description: 'A comprehensive guide to resource management and isolation using Linux cgroups, covering cgroups v1 vs v2, implementation, and real-world usage.'
pubDate: 'Apr 18 2026'
---

Linux Control Groups, widely known as cgroups, form the backbone of modern containerization and resource management in Linux environments. From Docker to Kubernetes, cgroups provide the necessary primitives to limit, account for, and isolate resource usage (CPU, memory, disk I/O, network) of a collection of processes.

In this deep dive, we'll explore what cgroups are, how they evolved from version 1 to version 2, and how you can interact with them directly.

## What are cgroups?

At its core, a cgroup is a mechanism for organizing processes hierarchically and distributing system resources along the hierarchy in a controlled and configurable manner. Developed by Google engineers in 2006 (originally named "process containers"), the feature was merged into the mainline Linux kernel in version 2.6.24.

Cgroups allow administrators to:
1. **Resource Limiting:** Restrict the maximum amount of memory or CPU a group of processes can use.
2. **Prioritization:** Give certain groups larger shares of CPU utilization or disk I/O throughput.
3. **Accounting:** Measure the exact resource consumption of a process group for billing or monitoring.
4. **Control:** Freeze and resume execution of all processes in a group.

## The Shift from cgroups v1 to v2

For many years, cgroups v1 was the standard. However, it had architectural limitations. In v1, different resource controllers (like `cpu`, `memory`, `blkio`) were mounted in separate hierarchies. This meant a process could belong to multiple independent hierarchies simultaneously, leading to complex management and inconsistencies.

### Enter cgroups v2

Introduced in kernel 4.5, cgroups v2 brought a unified hierarchy design. 
- **Single Hierarchy:** A process can only belong to one cgroup in the system, and all resource controllers apply to that single node in the tree.
- **Safer Delegation:** v2 makes it much safer to delegate cgroup management to unprivileged processes (a crucial requirement for rootless containers).
- **BPF Integration:** Modern cgroups tightly integrate with eBPF, allowing for highly efficient, programmatic control over network traffic and device access.

## Interacting with cgroups

You don't need Docker to use cgroups; they are built directly into your filesystem under `/sys/fs/cgroup`.

### Checking Your cgroup Version

To see if your system uses v2, you can run:
```bash
mount | grep cgroup
```
If you see `cgroup2` mounted at `/sys/fs/cgroup`, you are running the modern unified hierarchy (standard on most distros since systemd 245).

### Creating a cgroup

Creating a cgroup is as simple as creating a directory:
```bash
sudo mkdir /sys/fs/cgroup/my_app
```
Immediately, the kernel populates this directory with interface files:
```bash
ls /sys/fs/cgroup/my_app
```
You'll see files like `cgroup.procs`, `memory.max`, and `cpu.weight`.

### Applying Limits

To limit our new group to 500MB of memory:
```bash
echo 500M | sudo tee /sys/fs/cgroup/my_app/memory.max
```

To limit it to using only 50% of a single CPU core:
```bash
echo "50000 100000" | sudo tee /sys/fs/cgroup/my_app/cpu.max
```

### Assigning a Process

To apply these limits, simply write a Process ID (PID) to the `cgroup.procs` file:
```bash
echo $$ | sudo tee /sys/fs/cgroup/my_app/cgroup.procs
```
Now, your current shell and any child processes it spawns will be bound by the 500MB memory limit and 50% CPU limit.

## Systemd and cgroups

In modern Linux, `systemd` is the primary cgroup manager. When you create a `systemd` service, it automatically gets its own cgroup. You can easily apply limits in your service unit files:

```ini
[Service]
ExecStart=/usr/bin/my_daemon
MemoryMax=1G
CPUQuota=200%
```

This configuration ensures the daemon uses at most 1GB of RAM and 2 full CPU cores, abstracting away the manual filesystem interactions.

## Conclusion

Understanding cgroups demystifies how containers work. They are not magic; they are simply standard Linux processes bounded by strict resource quotas defined in a virtual filesystem. As eBPF and cgroups v2 continue to mature, the precision and performance of Linux resource management will only improve.
