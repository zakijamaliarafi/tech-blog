---
heroImage: '/setting-up-wireguard-vpn-linux.svg'
title: 'Setting up a Secure WireGuard VPN on Linux'
description: 'A step-by-step tutorial on deploying WireGuard, the modern, high-performance VPN protocol, on a Linux VPS.'
pubDate: 'May 16 2026'
---

WireGuard is a modern, fast, and secure VPN protocol that aims to be leaner and more performant than OpenVPN or IPsec. It utilizes state-of-the-art cryptography and is remarkably easy to configure.

## Installing WireGuard

WireGuard is available in the repositories of most major Linux distributions.

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install wireguard
```

## Generating Keys

WireGuard uses public-key cryptography (similar to SSH). You need a private and public key for the server, and a pair for every client.

On the server:
```bash
wg genkey | tee server_private_key | wg pubkey > server_public_key
```

## Configuring the Server

Create a configuration file at `/etc/wireguard/wg0.conf`.

```ini
[Interface]
Address = 10.8.0.1/24
SaveConfig = true
PrivateKey = <YOUR_SERVER_PRIVATE_KEY>
ListenPort = 51820

# Enable IP forwarding and NAT
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client 1
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32
```
*Note: Replace `eth0` with your server's actual public network interface name.*

## Enabling IP Forwarding

To allow traffic to route through the VPN to the internet, enable IPv4 forwarding:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.d/99-wireguard.conf
sudo sysctl -p /etc/sysctl.d/99-wireguard.conf
```

## Starting the VPN

Bring up the WireGuard interface and enable it on boot:
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

## Client Configuration

On the client machine, create a similar configuration file:

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <YOUR_SERVER_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0, ::/0
```
`AllowedIPs = 0.0.0.0/0` ensures all client traffic is routed through the VPN tunnel.

With minimal configuration, you now have an extremely secure, high-speed VPN tunnel.

