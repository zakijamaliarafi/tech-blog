---
heroImage: '/networking-commands.svg'
title: 'Introduction to Linux Networking Commands'
description: 'Master ip, ping, netstat, and other essential networking tools.'
pubDate: 'Apr 24 2026'
---

When configuring a server or troubleshooting connectivity issues, understanding Linux networking commands is essential. The older `net-tools` suite (like `ifconfig`) is largely deprecated in favor of the newer `iproute2` suite.

## The Versatile `ip` Command

The `ip` command is the modern standard for viewing and configuring network interfaces, routing, and tunnels.

*   **View IP Addresses:** 
    ```bash
    ip addr show
    ```
    This lists all network interfaces and their assigned IP addresses. (You can also use the shorthand `ip a`).
*   **View Routing Table:**
    ```bash
    ip route show
    ```
    This shows how the system decides where to send network traffic.

## Checking Connectivity: `ping`

The `ping` command is the simplest way to check if a remote host is reachable. It sends ICMP Echo Request packets and waits for a reply.
```bash
ping google.com
```
*Note: On Linux, `ping` runs indefinitely by default. Press `Ctrl+C` to stop it, or use the `-c` flag to specify a count (e.g., `ping -c 4 google.com`).*

## DNS Lookups: `dig` and `host`

When you need to resolve domain names to IP addresses or investigate DNS records, use `dig` or `host`.
```bash
host example.com
dig example.com
```
`dig` provides much more detailed output, making it better for advanced DNS troubleshooting.

## Checking Open Ports and Connections

To see what services are listening on your machine and what connections are currently active, use `ss` (socket statistics), which replaces the older `netstat`.
```bash
ss -tuln
```
*   `-t`: Show TCP sockets.
*   `-u`: Show UDP sockets.
*   `-l`: Show only listening sockets.
*   `-n`: Show numerical addresses (don't resolve IPs to hostnames).

Mastering these commands allows you to diagnose network issues quickly and efficiently from the terminal.
