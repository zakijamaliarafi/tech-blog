---
heroImage: '/process-management.svg'
title: 'Linux Process Management and Monitoring'
description: 'Use top, ps, and kill to monitor and manage Linux processes.'
pubDate: 'Apr 22 2026'
---

When managing a Linux system, it is crucial to know how to monitor resource usage and control running programs. In Linux, a running instance of a program is called a process.

## Viewing Active Processes: `ps`

The `ps` (process status) command takes a snapshot of currently running processes.

*   To see processes running in your current terminal session:
    ```bash
    ps
    ```
*   To see a detailed list of all processes running on the system by all users:
    ```bash
    ps aux
    ```
This command outputs the Process ID (PID), user, CPU usage, memory usage, and the command that started the process.

## Real-Time Monitoring: `top` and `htop`

While `ps` provides a static snapshot, `top` provides a dynamic, real-time view of system processes.

```bash
top
```
By default, `top` sorts processes by CPU usage, allowing you to quickly identify resource hogs. Press `q` to exit `top`.

An even better alternative is `htop`, which provides a colorful, interactive interface where you can scroll horizontally and vertically. It usually needs to be installed via your package manager (e.g., `sudo apt install htop`).

## Managing Processes: `kill`

If a process is unresponsive or consuming too many resources, you may need to terminate it. You do this by sending a signal to the process using the `kill` command along with the Process ID (PID).

1.  First, find the PID using `ps` or `top`.
2.  Then, use the `kill` command:
    ```bash
    kill 1234
    ```
    (Replace 1234 with the actual PID). This sends a standard termination signal (SIGTERM), asking the process to shut down gracefully.

If the process refuses to terminate, you can force it to stop by sending a SIGKILL signal using the `-9` flag:
```bash
kill -9 1234
```
Use `kill -9` as a last resort, as it doesn't allow the program to clean up temporary files or save data before closing.

Understanding these tools gives you complete control over the applications running on your system.
