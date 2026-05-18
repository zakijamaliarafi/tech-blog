---
heroImage: '/ssh-best-practices.svg'
title: 'Secure Shell (SSH) Best Practices'
description: 'Secure your Linux servers with proper SSH configuration and key-based authentication.'
pubDate: 'Apr 20 2026'
---

Secure Shell (SSH) is the standard protocol for securely connecting to remote Linux servers over an unsecured network. While SSH encrypts traffic by default, relying solely on basic password authentication can leave your server vulnerable to brute-force attacks. 

## 1. Use SSH Keys Instead of Passwords

The most significant security improvement you can make is disabling password logins and using SSH key pairs. 

An SSH key pair consists of a public key (placed on the server) and a private key (kept securely on your local machine). When you attempt to connect, the server verifies your private key cryptographically.

**Generating a key pair locally:**
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
**Copying the public key to the server:**
```bash
ssh-copy-id username@remote_host
```

## 2. Disable Password Authentication

Once you have verified that you can log in using your SSH key, you should disable password authentication entirely. Edit the SSH daemon configuration file on the server:

```bash
sudo nano /etc/ssh/sshd_config
```
Find the line containing `PasswordAuthentication` and set it to `no`:
```text
PasswordAuthentication no
```
Restart the SSH service to apply changes:
```bash
sudo systemctl restart sshd
```

## 3. Disable Root Login

Logging in directly as the `root` user is dangerous. If an attacker guesses the root password, they have total control over the system. Instead, log in as a standard user and use `sudo` to perform administrative tasks.

In `/etc/ssh/sshd_config`, set:
```text
PermitRootLogin no
```

## 4. Change the Default Port (Optional but Recommended)

By default, SSH listens on port 22. Automated bots constantly scan the internet for open port 22 to launch attacks. Changing the port won't stop a determined attacker, but it drastically reduces "background noise" in your logs.

In `/etc/ssh/sshd_config`, change the port number:
```text
Port 2222
```
Remember to update your firewall rules to allow traffic on the new port before restarting the SSH service.

Implementing these practices ensures your Linux servers remain secure against unauthorized access.
