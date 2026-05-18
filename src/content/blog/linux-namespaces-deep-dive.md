---
heroImage: '/linux-namespaces-deep-dive.svg'
title: 'A Deep Dive into Linux Namespaces'
description: 'Understand the core isolation technology behind containers. Explore PID, Mount, Network, and User namespaces with practical examples.'
pubDate: 'Apr 28 2026'
---

If Control Groups (cgroups) limit *how much* a process can use, Linux Namespaces limit *what* a process can see. Together, they form the foundation of container engines like Docker, Podman, and LXC.

Namespaces wrap a global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.

## The 7 Types of Namespaces

Linux currently supports several types of namespaces, each isolating a specific system resource:

1. **Mount (mnt):** Isolates filesystem mount points. A process in a mnt namespace can have its own root filesystem.
2. **Process ID (pid):** Isolates the PID number space. Processes in different PID namespaces can have the same PID. The first process in a new PID namespace gets PID 1.
3. **Network (net):** Isolates network interfaces, routing tables, and firewall rules.
4. **Interprocess Communication (ipc):** Isolates System V IPC objects and POSIX message queues.
5. **UNIX Time-sharing System (uts):** Isolates hostname and NIS domain name.
6. **User (user):** Isolates user and group IDs. A process's user and group IDs can be different inside and outside the namespace (e.g., you can be `root` inside, but an unprivileged user outside).
7. **Control Group (cgroup):** Isolates the cgroup root directory, making a process see its own cgroup as the root of the hierarchy.

## Playing with Namespaces using `unshare`

You don't need a container engine to experiment with namespaces. The `unshare` utility, part of the standard `util-linux` package, allows you to run a program with some namespaces unshared from its parent.

### Isolating the Hostname (UTS Namespace)

Let's create a new shell that lives in its own UTS namespace:
```bash
sudo unshare --uts /bin/bash
```
Now, inside this new shell, change the hostname:
```bash
hostname isolated-box
```
If you open a second terminal on your host and run `hostname`, you will see your original hostname. The change is completely isolated to the processes inside the new UTS namespace!

### Isolating Processes (PID Namespace)

Creating a new PID namespace is slightly more complex because tools like `ps` read from the `/proc` filesystem. To make `ps` work correctly, we must also unshare the mount namespace and mount a fresh `/proc`.

```bash
sudo unshare --pid --fork --mount-proc /bin/bash
```
Now run:
```bash
ps aux
```
You will notice that `bash` is running as PID 1! It has no visibility into the thousands of processes running on the host system.

## The Power of User Namespaces

User namespaces are arguably the most critical for security. They allow for "rootless" containers.

When you create a user namespace, you map a range of user IDs on the host to a range of user IDs inside the namespace. For example, you can map the host user `arafi` (UID 1000) to `root` (UID 0) inside the namespace.

Inside the namespace, the process believes it is root. It can perform operations that require root (like creating network namespaces or mounting filesystems), but the kernel knows that *outside* the namespace, it is still just the unprivileged user `arafi`. If the process breaks out, it has no special privileges on the host system.

## Network Namespaces in Action

Network namespaces are heavily used by SDN (Software Defined Networking) and tools like Kubernetes.

Create a new network namespace:
```bash
sudo ip netns add my_net
```
List your network namespaces:
```bash
ip netns list
```
Execute a command inside the namespace:
```bash
sudo ip netns exec my_net ip link
```
You'll see only a `lo` (loopback) interface, and it's completely disconnected from the host's network. To connect it, you would typically create a Virtual Ethernet (`veth`) pair, placing one end in the host and the other in the namespace.

## Conclusion

Linux Namespaces are powerful kernel features that provide process isolation without the overhead of hardware virtualization. By combining UTS, PID, Mount, and Network namespaces, you effectively create a lightweight, isolated execution environment—what we commonly call a container.
