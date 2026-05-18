---
heroImage: '/vps-monitoring-and-logging.svg'
title: 'Monitoring Performance and Managing Logs on Your VPS'
description: 'Tools and techniques for keeping an eye on your server health and troubleshooting issues.'
pubDate: 'Apr 13 2026'
---

Once your VPS is configured and serving traffic, your job shifts from deployment to maintenance. You need to know how the server is performing under load and have the tools necessary to investigate when things go wrong.

This involves two critical practices: **Monitoring** and **Log Management**.

## Real-Time Command Line Monitoring

Before installing complex dashboard software, you should be familiar with the built-in Linux tools for diagnosing immediate issues.

### 1. `top` and `htop`
`top` provides a dynamic, real-time view of running processes, CPU usage, and RAM consumption. 
`htop` is an enhanced, interactive version of `top` (often requires installation: `sudo apt install htop`). It allows you to scroll vertically and horizontally and send signals (like SIGKILL) directly to processes.

### 2. `free -m`
This command instantly displays the total, used, and free memory (RAM) and swap space in megabytes.

### 3. `df -h`
Use `df` (disk free) with the `-h` (human-readable) flag to check how much storage space is available on your server's partitions. Running out of disk space is a very common cause of server crashes.

### 4. `du -sh *`
If your disk is full, `du` (disk usage) helps you find the culprit. Running this inside a directory will show you the size of all files and folders within it.

## Managing and Viewing Logs

Logs are text files where the OS and applications record events, errors, and access requests. They are your primary source of truth when debugging.

The standard directory for logs in Linux is `/var/log/`.

### Viewing Logs with `tail` and `less`
*   `tail -f /var/log/syslog` (or `messages` on RHEL): Continuously prints new lines added to the system log in real-time. Crucial for watching live events.
*   `less /var/log/nginx/error.log`: Opens a log file in a reader that allows you to scroll up and down and search for specific text without loading the entire file into memory at once.

### Key Log Files to Know
*   `/var/log/auth.log` (Ubuntu/Debian) or `/var/log/secure` (RHEL): Logs authentication attempts, including SSH logins and sudo usage. Watch this for brute-force attacks.
*   `/var/log/nginx/access.log` and `error.log`: If you run Nginx, these record every HTTP request and server error.
*   `/var/log/mysql/error.log`: Crucial for diagnosing database crashes or configuration issues.

## Setting Up Comprehensive Monitoring Tools

Command-line tools are great for active debugging, but you also need historical data and automated alerts.

### 1. Netdata
Netdata is an incredibly powerful, low-overhead monitoring tool that provides highly detailed, real-time dashboards right out of the box. It monitors CPU, RAM, disk I/O, network traffic, and can automatically detect applications like Nginx and MySQL.
*   It is generally easy to install via a one-line script and runs on its own built-in web server.

### 2. Prometheus and Grafana
This is the industry standard for enterprise monitoring.
*   **Prometheus** is a time-series database that scrapes metrics from your server.
*   **Grafana** is a visualization tool that connects to Prometheus to build beautiful, customizable dashboards.
While more complex to set up than Netdata, it offers unparalleled flexibility and is the go-to solution for monitoring Docker/Kubernetes environments.

### 3. Uptime Monitoring (External)
Internal monitoring won't help if the entire VPS goes offline. Use external services like **UptimeRobot** or **Pingdom** to ping your website every minute from outside your network. They will send you an email or SMS the moment your site becomes unreachable.
