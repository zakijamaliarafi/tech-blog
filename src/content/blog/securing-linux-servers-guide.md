---
title: 'Hardening Linux Servers: A Practical Security Checklist'
description: 'Essential security practices to protect your Linux servers from brute-force attacks, privilege escalation, and unauthorized access.'
pubDate: 'Apr 21 2026'
---

Deploying a Linux server to the public internet means it will be scanned and attacked by automated bots within minutes. Securing your infrastructure is not a one-time task, but following a strict baseline hardening checklist mitigates the vast majority of automated attacks.

Here is a practical guide to securing a production Linux server.

## 1. Secure SSH Access

SSH is the front door to your server. Securing it is priority number one.
Edit `/etc/ssh/sshd_config`:

**Disable Password Authentication:** Force the use of cryptographic SSH keys.
```text
PasswordAuthentication no
```

**Disable Root Login:** Never log in directly as root. Log in as a standard user and use `sudo` or `su`.
```text
PermitRootLogin no
```

**Change the Default Port (Optional):** While security by obscurity isn't true security, changing the port from 22 to something like 2222 drastically reduces the noise from automated script kiddies in your auth logs.
```text
Port 2222
```
*Restart the SSH service after making changes: `sudo systemctl restart sshd`.*

## 2. Implement Fail2Ban

Even with keys only, bots will hammer your SSH port. `fail2ban` monitors log files for repeated failed authentication attempts and dynamically adds `iptables` or `nftables` rules to block the attacker's IP address.

Install it:
```bash
sudo apt install fail2ban
```
It requires almost no configuration out of the box to protect SSH, but it can be easily extended to protect Nginx, Apache, and Postfix.

## 3. Configure a Strict Firewall

Adopt a "default deny" posture. Block everything, and only explicitly allow the ports you need.

Using `ufw` (Uncomplicated Firewall) on Ubuntu:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (use your custom port if changed)
sudo ufw allow 22/tcp
# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable
```

## 4. Principle of Least Privilege

Ensure users and applications only have the permissions they absolutely need.

- **Use sudo diligently:** Do not use `root` for daily tasks. Give users specific `sudo` permissions via `/etc/sudoers.d/` rather than blanket access.
- **Service Accounts:** Do not run web servers or databases as root. Nginx should run as `www-data`, PostgreSQL as `postgres`. If a service is compromised, the attacker only gains the privileges of that specific user.

## 5. Automated Security Updates

Unpatched software is the primary vector for system compromises. Configure your system to install security updates automatically.

On Debian/Ubuntu, configure `unattended-upgrades`:
```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```
Ensure that it is configured to only install *security* updates automatically, to prevent unexpected feature changes from breaking your applications.

## 6. Kernel Hardening and SELinux/AppArmor

### AppArmor / SELinux
Do not disable SELinux or AppArmor. These Mandatory Access Control (MAC) systems provide a massive layer of security. They confine applications; even if an attacker finds an exploit in Nginx, SELinux prevents Nginx from reading files in `/etc` or executing a shell.

### Kernel Parameters (sysctl)
Harden the network stack against common attacks like SYN floods and spoofing by adding these to `/etc/sysctl.d/99-security.conf`:

```ini
# Ignore ICMP broadcast requests (prevent smurf attacks)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Enable TCP SYN Cookie Protection
net.ipv4.tcp_syncookies = 1

# Disable IP source routing
net.ipv4.conf.all.accept_source_route = 0

# Protect against IP spoofing
net.ipv4.conf.all.rp_filter = 1
```
Apply with `sudo sysctl -p`.

## Conclusion

Security is an ongoing process of risk management. By enforcing key-based authentication, running a strict firewall, and isolating services, you eliminate the low-hanging fruit and make your server a significantly harder target for adversaries.
