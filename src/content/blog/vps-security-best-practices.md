---
title: 'Essential Security Best Practices for Your New VPS'
description: 'Learn how to secure your Virtual Private Server against common threats and vulnerabilities.'
pubDate: 'Apr 10 2026'
---

The moment your new Virtual Private Server (VPS) goes online, it becomes a target for automated scanning bots and malicious actors looking for vulnerabilities. Unlike managed shared hosting, securing a VPS is entirely your responsibility. 

Implementing these essential security best practices immediately after provisioning your server will significantly reduce your attack surface.

## 1. Ditch Passwords, Use SSH Keys

Password authentication for SSH (Secure Shell) is vulnerable to brute-force attacks. SSH keys provide a cryptographic, vastly more secure method of authentication.

*   **Generate an SSH Key Pair:** On your local machine, generate an RSA or Ed25519 key pair.
*   **Add the Public Key to the VPS:** Add your public key to the `~/.ssh/authorized_keys` file on the server.
*   **Disable Password Authentication:** Once key-based login is confirmed working, edit your SSH daemon configuration (`/etc/ssh/sshd_config`) and set `PasswordAuthentication no`. Restart the SSH service.

## 2. Change the Default SSH Port

Bots constantly scan port 22 (the default SSH port). While changing the port doesn't stop targeted attacks, it drastically reduces the "background noise" of automated brute-force attempts.

*   Edit `/etc/ssh/sshd_config`.
*   Find the line `Port 22` and change it to a non-standard port (e.g., `Port 2244`).
*   Ensure your firewall allows traffic on the new port before restarting the SSH service!

## 3. Disable Root Login via SSH

Logging in directly as the `root` user is risky. Instead, you should log in as a standard user and escalate privileges only when necessary using `sudo`.

*   Create a new user: `adduser yourusername`
*   Add the user to the sudo group: `usermod -aG sudo yourusername` (Ubuntu/Debian) or `usermod -aG wheel yourusername` (RHEL/AlmaLinux).
*   Edit `/etc/ssh/sshd_config` and set `PermitRootLogin no`.

## 4. Configure a Firewall (UFW or Firewalld)

A firewall acts as a gatekeeper, controlling which incoming and outgoing network traffic is allowed. By default, a VPS might have all ports open.

**For Ubuntu/Debian (Using UFW - Uncomplicated Firewall):**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2244/tcp # Your custom SSH port
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw enable
```

**For AlmaLinux/Rocky (Using Firewalld):**
```bash
sudo firewall-cmd --permanent --add-port=2244/tcp
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

## 5. Install and Configure Fail2Ban

Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. It monitors log files (like SSH logs) and automatically updates firewall rules to ban IP addresses that exhibit malicious signs, such as too many password failures.

*   Install Fail2Ban (`sudo apt install fail2ban` or `sudo dnf install fail2ban`).
*   Create a local configuration file (`/etc/fail2ban/jail.local`) to define your rules and ban times.

## 6. Keep the System Updated

Outdated software is one of the most common vectors for exploitation. Regularly apply security patches and updates.

*   **Debian/Ubuntu:** `sudo apt update && sudo apt upgrade -y`
*   **Alma/Rocky:** `sudo dnf update -y`

Consider setting up **Unattended Upgrades** to automatically install security patches without manual intervention.

## 7. Audit Listening Services

Minimize the software running on your VPS. Every running service is a potential entry point. Use tools like `netstat` or `ss` to view which ports are open and listening.

```bash
sudo ss -tulpen
```
If you see a service listening on a public IP that shouldn't be (e.g., a database that only needs to be accessed locally), reconfigure it to bind only to `127.0.0.1` (localhost) or stop and disable the service entirely.
