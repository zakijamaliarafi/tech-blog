---
heroImage: '/linux-performance-tuning.svg'
title: 'Practical Linux Performance Tuning for Production Servers'
description: 'Optimize your Linux servers by tuning the CPU frequency governor, tweaking sysctl networking parameters, and managing I/O schedulers.'
pubDate: 'Apr 27 2026'
---

Out of the box, Linux distributions are tuned for a balance of power savings, desktop responsiveness, and general compatibility. If you are running high-traffic web servers, massive databases, or high-throughput network appliances, the default settings leave significant performance on the table.

Here is a guide to tuning critical subsystems in Linux.

## 1. CPU Power Management

Modern processors aggressively scale down their frequency to save power when idle. However, the time it takes to ramp up the CPU frequency when a sudden burst of requests arrives can add measurable latency.

Linux controls this via "Governors".
- **powersave:** Prioritizes low power consumption.
- **performance:** Locks the CPU at its maximum frequency.

For database servers and high-frequency trading applications, lock the CPU to maximum performance:
```bash
# Apply to all cores instantly
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```
To make this persistent, install `cpufrequtils` and configure it in `/etc/default/cpufrequtils`.

## 2. Networking (sysctl Tuning)

The Linux network stack is highly tunable via `/etc/sysctl.conf`. The defaults are generally designed for modest hardware and varied network conditions. 

If you are dealing with thousands of concurrent connections (e.g., Nginx or HAProxy), you need to increase system limits.

### TCP Connection Queue
Increase the number of connections allowed in the backlog (preventing dropped connections during sudden traffic spikes):
```ini
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
```

### Ephemeral Ports
If your server makes many outgoing connections (like a reverse proxy), it might run out of local ports. Expand the range and allow fast recycling of TIME_WAIT sockets:
```ini
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
```

### BBR Congestion Control
Google developed the BBR (Bottleneck Bandwidth and Round-trip propagation time) congestion control algorithm. It drastically improves throughput and reduces latency over long-distance, high-packet-loss links compared to the default `cubic` algorithm.
```ini
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
```
Apply all changes instantly with `sudo sysctl -p`.

## 3. Storage I/O Schedulers

The kernel uses I/O schedulers to order disk read/write requests. 

Historically, for spinning hard drives (HDDs), schedulers like `cfq` grouped requests by physical location to minimize the mechanical arm movement.
For modern NVMe and SSD drives, mechanical movement is irrelevant. Using a legacy scheduler on an SSD wastes CPU cycles.

Check your current scheduler (replace `sda` with your drive):
```bash
cat /sys/block/sda/queue/scheduler
```

For NVMe/SSD drives, ensure you are using **`none`** or **`mq-deadline`** (Multi-Queue).
To change it temporarily:
```bash
echo none | sudo tee /sys/block/sda/queue/scheduler
```
Make it permanent by adding `elevator=none` to your GRUB boot parameters.

## 4. File Descriptors

In Linux, "everything is a file," including network sockets. A busy web server handling 10,000 concurrent users needs at least 10,000 open file descriptors.

Check the system-wide limit:
```bash
cat /proc/sys/fs/file-max
```
Increase it in `/etc/sysctl.conf` if necessary:
```ini
fs.file-max = 2097152
```

More importantly, check the per-user limits defined in `/etc/security/limits.conf`:
```ini
*       soft    nofile  100000
*       hard    nofile  100000
```

## Measurement Before Modification

The golden rule of performance tuning is: **Never change a parameter without measuring its impact.**
Use tools like `perf`, `iostat`, `ss`, and `bcc` to identify your actual bottleneck. Tuning network queues won't help if your application is bottlenecked by CPU cache misses!
