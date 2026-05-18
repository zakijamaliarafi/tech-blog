---
heroImage: '/ebpf-tracing-guide.svg'
title: 'Superpowers for Linux: An Introduction to eBPF'
description: 'Discover how eBPF is revolutionizing Linux observability, networking, and security by safely running sandboxed programs within the kernel.'
pubDate: 'May 03 2026'
---

Historically, extending the Linux kernel's capabilities meant writing kernel modules. This was risky—a single bug could cause a kernel panic, crashing the entire system. 

Enter **eBPF (Extended Berkeley Packet Filter)**. eBPF is arguably the most significant addition to the Linux kernel in the last decade. It allows you to run sandboxed, verified code directly in the kernel space without changing kernel source code or loading kernel modules.

## From Packet Filtering to Everything

The "BPF" in eBPF stands for Berkeley Packet Filter, which was originally used in tools like `tcpdump` to efficiently filter network packets. eBPF is the "extended" version, which expanded the architecture from just network filtering to almost any kernel event.

Think of eBPF as JavaScript for the Linux kernel. Just as JavaScript allows you to run code on web pages in response to events (clicks, loads), eBPF allows you to run code in the kernel in response to system events (disk I/O, network packets, system calls, function entries/exits).

## How eBPF Works

1. **Write:** You write an eBPF program, typically in a restricted subset of C.
2. **Compile:** You compile the C code into eBPF bytecode using LLVM/Clang.
3. **Verify:** You load the bytecode into the kernel using the `bpf()` system call. Before the program is allowed to run, the **eBPF Verifier** analyzes it to ensure it is safe:
    * It must not crash.
    * It must not loop infinitely.
    * It must not access out-of-bounds memory.
4. **JIT Compile:** Once verified, the Just-In-Time (JIT) compiler translates the bytecode into native machine code for maximum performance.
5. **Attach:** The program is attached to a specific hook point in the kernel (e.g., a network interface, a tracepoint, or a kprobe).

## Key Use Cases

### 1. High-Performance Networking

Traditional Linux networking pushes packets through a complex, multi-layered networking stack. eBPF can intercept packets the moment they hit the network interface card (NIC), bypassing the kernel stack entirely. Projects like **Cilium** use eBPF to provide blazing-fast load balancing, routing, and network policies for Kubernetes.

### 2. Deep Observability

Because eBPF can hook into almost any kernel function, it provides unprecedented visibility into system behavior. You can measure exactly how long a database query spends waiting for disk I/O at the block device layer, or trace every execution of the `execve` system call to see what processes are starting.

Tools like **bcc (BPF Compiler Collection)** and **bpftrace** make writing observability scripts incredibly simple.

A simple `bpftrace` one-liner to trace all files being opened:
```bash
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'
```

### 3. Advanced Security

Security tools can use eBPF to enforce policies at the kernel level. For instance, an eBPF program can inspect the arguments of a system call before allowing it to proceed. If a web server process suddenly attempts to execute `/bin/bash`, an eBPF program can instantly block the action and trigger an alert. Projects like **Tetragon** leverage this for runtime security enforcement.

## Getting Started with bpftrace

If you want to start exploring eBPF, the easiest entry point is `bpftrace`. It uses a high-level scripting language inspired by awk and DTrace.

Install it on Ubuntu/Debian:
```bash
sudo apt install bpftrace
```

Here's a small script (`vfsstat.bt`) to count Virtual File System (VFS) calls:
```c
kprobe:vfs_read, kprobe:vfs_write {
    @[func] = count();
}

interval:s:1 {
    print(@);
    clear(@);
}
```
Run it: `sudo bpftrace vfsstat.bt`. It will output the number of reads and writes every second, with negligible overhead to the system.

## The Future of Linux

eBPF is fundamentally changing how infrastructure software is built. By providing a safe, programmable interface to the kernel, it is replacing legacy tools with highly performant, dynamic alternatives. Whether you are debugging a latency spike or securing a massive container cluster, eBPF is the modern superpower you need.
