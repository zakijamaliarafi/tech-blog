---
heroImage: '/ssh-best-practices.svg'
title: 'Secure Shell (SSH) Best Practices: A Definitive Guide'
description: 'Secure your Linux servers against brute-force and targeted attacks with proper SSH configuration, advanced key-based authentication, two-factor authentication, and robust audit logging.'
pubDate: 'Apr 20 2026'
---

Since its creation in 1995 to replace insecure plaintext protocols like Telnet and rlogin, Secure Shell (SSH) has become the undisputed standard for remote server administration. It provides a secure, encrypted channel over an unsecured network, allowing administrators to execute commands, transfer files, and tunnel traffic with confidence.

However, the fact that the protocol itself is cryptographically secure does not mean your server is automatically safe. SSH is the front door to your infrastructure. Because it is so ubiquitous, it is the primary target for automated botnets and sophisticated adversaries. If your SSH configuration relies on default settings and weak passwords, your server will inevitably be compromised. 

Securing SSH requires a defense-in-depth approach. This guide covers everything from the absolute basics of key-based authentication to advanced techniques like Certificate Authorities, Two-Factor Authentication (2FA), and restrictive network policies.

## Phase 1: Eliminating Passwords

The most critical vulnerability in any SSH setup is human memory. Passwords can be guessed, brute-forced, stolen via phishing, or reused from compromised websites. 

### 1. Generating Cryptographic Keys

The foundation of SSH security is Public Key Cryptography. Instead of a password, you generate a mathematical pair of keys:
*   **Public Key:** You place this on the server. It acts as a lock. It is public; anyone can see it, and it cannot be used to deduce the private key.
*   **Private Key:** You keep this on your local machine. It acts as the key to the lock. It must never be shared or transmitted over the internet.

Historically, RSA keys were the standard. However, modern security standards strongly recommend **Ed25519**. Ed25519 is an elliptical curve cryptography algorithm that is significantly faster, generates much shorter keys, and is highly resistant to side-channel attacks.

Generate an Ed25519 key pair on your local machine:
```bash
ssh-keygen -t ed25519 -C "admin_laptop_2026"
```
*Crucially, when prompted, enter a strong passphrase. If someone steals your laptop and extracts your private key file, the passphrase encrypts the key at rest, rendering it useless to the thief.*

### 2. Distributing the Public Key

Next, you must copy the public key to the remote server. The `ssh-copy-id` utility automates this safely:
```bash
ssh-copy-id username@remote_server_ip
```
This appends your public key to the `~/.ssh/authorized_keys` file on the server.

### 3. Disabling Password Authentication

Once you have verified that you can successfully log in using your SSH key, you must slam the door shut on password logins. 

Connect to your server and open the SSH daemon configuration file:
```bash
sudo nano /etc/ssh/sshd_config
```

Find the following directives and change them to `no`:
```text
PasswordAuthentication no
ChallengeResponseAuthentication no # Sometimes named KbdInteractiveAuthentication
UsePAM yes # Keep this yes, but the above settings prevent password prompting
```

Restart the SSH service:
```bash
sudo systemctl restart sshd
```
Your server is now mathematically immune to password brute-force attacks.

## Phase 2: Hardening the `sshd_config`

Disabling passwords is just the beginning. The `sshd_config` file contains numerous settings that should be tightened to restrict access and limit the damage if a compromise occurs.

### 1. Disable Root Login

The `root` user has absolute power over the system. Attackers know the `root` account exists, so 99% of brute-force attempts target it. 

You should never log in directly as `root`. Log in as a standard, unprivileged user, and then use `sudo` to temporarily escalate privileges.
```text
PermitRootLogin no
```

### 2. Restrict Allowed Users

By default, any user with an account on the Linux system can attempt to log in via SSH. You can explicitly whitelist only the users who require remote access. If an attacker compromises a service account (like the `www-data` web server user), they cannot use it to spawn an SSH shell.

```text
AllowUsers alice bob admin
# Alternatively, you can allow a specific group:
# AllowGroups wheel admins
```

### 3. Disable X11 Forwarding and Port Forwarding

SSH is a powerful tunneling protocol. It can be used to forward local ports to remote ports, or even forward graphical X11 applications over the network. If an attacker breaches a restricted user account, they can use SSH port forwarding to bypass your external firewalls and access internal, protected databases.

If you only need SSH for standard command-line administration, disable these features:
```text
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
```

### 4. Configure Idle Timeouts

If an administrator connects to a server, goes to lunch, and forgets to lock their laptop, a malicious actor could sit down and have full access to the production environment. You can configure SSH to automatically drop idle connections.

```text
# Check every 300 seconds (5 minutes) if the client is still responding
ClientAliveInterval 300
# Disconnect after 2 failed checks (10 minutes of total inactivity)
ClientAliveCountMax 2
```

## Phase 3: Advanced Network Security

SSH should not be exposed to the entire internet if it can be avoided.

### 1. Change the Default Port

Changing the default port from 22 to a random high port (like 49152) is controversial. It is "Security by Obscurity." A determined attacker performing a full port scan will find your SSH daemon regardless of what port it listens on.

However, changing the port completely eliminates the massive volume of "background noise"—the automated script kiddies and botnets that mindlessly scan the internet for port 22. It keeps your auth logs clean and saves CPU cycles.
```text
Port 49152
```

### 2. Implement IP Whitelisting

If your administrators connect from a static corporate IP address, or a static VPN IP address, you should use your firewall to completely block SSH access from the rest of the world.

Using `ufw` (Uncomplicated Firewall):
```bash
# Allow SSH only from the corporate VPN IP
sudo ufw allow from 198.51.100.45 to any port 22
```

### 3. Use a Bastion Host (Jump Server)

In an enterprise environment with dozens of servers, none of them should have their SSH ports exposed to the public internet. Instead, you deploy a single, highly secured "Bastion Host" (or Jump Server) in a public subnet. 

Administrators SSH into the Bastion Host first, and from there, they SSH into the internal servers located in private subnets. This reduces your attack surface to a single, easily monitored entry point.

## Phase 4: Ultimate Protection: 2FA and Certificates

For highly sensitive environments, SSH keys alone are not enough. If an attacker steals an administrator's laptop and key passphrase, they gain access.

### 1. Two-Factor Authentication (2FA)

You can configure SSH to require both a valid SSH Key AND a Time-based One-Time Password (TOTP) from an authenticator app (like Google Authenticator or Authy) on a smartphone.

This requires installing the Google Authenticator PAM module:
```bash
sudo apt install libpam-google-authenticator
```
You then configure the PAM stack (`/etc/pam.d/sshd`) and `sshd_config` to require both the `publickey` and `keyboard-interactive` authentication methods. If an attacker steals your private key, they still cannot log in without physically possessing your unlocked smartphone.

### 2. SSH Certificate Authorities

Managing SSH keys at scale is a nightmare. If you have 50 engineers and 500 servers, you must distribute 50 public keys to 500 `authorized_keys` files. If an engineer leaves the company, you must manually delete their key from 500 servers.

The enterprise solution is an **SSH Certificate Authority (CA)**. 

Instead of distributing keys to every server, you create a central CA. You configure all 500 servers to trust this single CA. When an engineer needs to log in, they request a "Certificate" from the central CA. The CA verifies their identity (usually via Single Sign-On, like Okta or Google Workspace) and signs their SSH key with a certificate that is only valid for a very short time—usually just 4 to 8 hours.

The engineer uses this short-lived certificate to log into the servers. Because the certificate expires at the end of the workday, there are no permanent keys to steal, and revoking access for a terminated employee happens instantly by disabling their SSO account. Tools like HashiCorp Vault or Teleport are industry standards for implementing SSH CA workflows.

## Conclusion

A misconfigured SSH daemon is a ticking time bomb. Securing it requires moving beyond the default installation. By enforcing cryptographic keys, hardening the configuration file to enforce least privilege, implementing strict network boundaries, and utilizing multi-factor authentication, you transform SSH from your greatest vulnerability into a fortress, ensuring that only trusted personnel can command your infrastructure.
