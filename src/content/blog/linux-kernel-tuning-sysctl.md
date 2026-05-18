---
heroImage: '/linux-kernel-tuning-sysctl.svg'
title: 'Linux Kernel Tuning with sysctl for High Performance'
description: 'A comprehensive guide on tweaking Linux kernel parameters via sysctl to optimize network, memory, and disk performance.'
pubDate: 'May 11 2026'
---

The default Linux kernel settings are designed to be general-purpose, balancing performance, compatibility, and resource usage. For high-load servers, you can achieve significant performance gains by tuning kernel parameters using `sysctl`.

## How sysctl Works

`sysctl` allows you to read and modify kernel parameters at runtime. Parameters are organized hierarchically, corresponding to the `/proc/sys/` filesystem.

To view all current parameters:
```bash
sysctl -a
```

To make changes permanent across reboots, edit the `/etc/sysctl.conf` file or place configuration files in `/etc/sysctl.d/`.

## Network Tuning (TCP/IP)

For high-traffic web servers or load balancers, TCP tuning is critical.

**Increase the max number of connections:**
```ini
net.core.somaxconn = 65535
```
This increases the size of the listen queue for accepting new TCP connections.

**Enable TCP BBR Congestion Control:**
BBR provides significantly better bandwidth utilization and lower latency on networks with packet loss.
```ini
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

**Tune TIME_WAIT sockets:**
On busy servers, sockets in `TIME_WAIT` state can accumulate, exhausting ephemeral ports.
```ini
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535
```

## Virtual Memory (VM) Tuning

**Swappiness:**
The `vm.swappiness` parameter dictates how aggressively the kernel swaps memory pages to disk. A value of 0 means avoid swapping unless absolutely necessary, while 100 means swap aggressively. For databases, a low value is preferred.
```ini
vm.swappiness = 10
```

**Dirty Pages:**
Tuning how the kernel writes dirty data to disk can improve I/O performance.
```ini
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
```
These parameters dictate the percentage of system memory that can be filled with "dirty" (modified but unwritten) data before the kernel begins flushing it to disk.

Apply your changes immediately using `sysctl -p`. Always test kernel parameter changes in a staging environment before applying them to production systems.

