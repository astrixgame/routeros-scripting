---
description: >-
  A fast, modern VPN protocol known for its simplicity and high security. It is
  technically Peer-To-Peer, but is usually used in a Client-Server setup.
icon: shield-keyhole
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# WireGuard

{% hint style="info" %}
WireGuard is a modern VPN protocol that offers excellent performance, strong cryptography, and simpler configuration compared to OpenVPN.
{% endhint %}

In WinBox you can configure WireGuard in **WireGuard** menu, or you can use terminal with command `/interface wireguard`

WireGuard uses **public key cryptography** instead of certificates, making it simpler to set up while maintaining high security.

***

## Key concepts

### WireGuard fundamentals

**Key pairs:**
- Each device has a **private key** (kept secret) and **public key** (shared with peers)
- Keys are used for authentication and encryption

**Peers:**
- WireGuard connections are between "peers"
- Each peer has an allowed IP range that defines which traffic it can send/receive
- No traditional "server" or "client" - all devices are peers

**Interface:**
- WireGuard creates a network interface (like `wg0`)
- This interface has its own IP address within the VPN network

***

## Server setup (RouterOS as WireGuard hub)

### Generate server keys

First, create the WireGuard interface and generate key pair:

In WinBox go to **WireGuard** and click <kbd>**+**</kbd> to add new interface:

* **Name** - wg-server (or your preferred name)
* **Listen Port** - 13231 (default WireGuard port, can be customized)
* **Private Key** - Click "Generate" to create new key pair

{% code overflow="wrap" %}
```bash
# Create WireGuard interface and generate keys
/interface wireguard add name=wg-server listen-port=13231

# The private key is automatically generated, get the public key
/interface wireguard print
```
{% endcode %}

### Configure server IP address

Assign an IP address to the WireGuard interface:

{% code overflow="wrap" %}
```bash
# Assign IP to WireGuard interface
/ip address add address=10.13.13.1/24 interface=wg-server
```
{% endcode %}

### Add firewall rules

Allow WireGuard traffic through firewall:

{% code overflow="wrap" %}
```bash
# Allow WireGuard port
/ip firewall filter add chain=input action=accept protocol=udp dst-port=13231 comment="Allow WireGuard"

# Allow traffic between WireGuard clients (optional)
/ip firewall filter add chain=forward action=accept in-interface=wg-server comment="Allow WireGuard forwarding"
/ip firewall filter add chain=forward action=accept out-interface=wg-server comment="Allow WireGuard forwarding"
```
{% endcode %}

### Configure NAT (if needed)

If you want clients to access internet through the VPN:

{% code overflow="wrap" %}
```bash
# NAT for WireGuard clients to access internet
/ip firewall nat add chain=srcnat out-interface=ether1 src-address=10.13.13.0/24 action=masquerade comment="WireGuard NAT"
```
{% endcode %}

***

## Adding clients (peers)

### Generate client keys

For each client, you need to generate a key pair. This is typically done on the client device, but you can generate them on RouterOS:

{% code overflow="wrap" %}
```bash
# Generate key pair for client (optional - usually done on client)
/interface wireguard peers print
```
{% endcode %}

### Add peer to server

In WinBox go to **WireGuard -> Peers** and click <kbd>**+**</kbd> to add peer:

* **Interface** - wg-server
* **Public Key** - Client's public key
* **Allowed Address** - IP range client can use (e.g., 10.13.13.2/32)
* **Endpoint** - Leave empty for dynamic clients
* **Persistent Keepalive** - 25 (for NAT traversal)

{% code overflow="wrap" %}
```bash
# Add client peer (replace CLIENT_PUBLIC_KEY with actual key)
/interface wireguard peers add interface=wg-server public-key="CLIENT_PUBLIC_KEY_HERE" allowed-address=10.13.13.2/32 persistent-keepalive=25s
```
{% endcode %}

### Multiple clients example

{% code overflow="wrap" %}
```bash
# Client 1
/interface wireguard peers add interface=wg-server public-key="CLIENT1_PUBLIC_KEY" allowed-address=10.13.13.2/32 persistent-keepalive=25s comment="Client 1 - John Doe"

# Client 2  
/interface wireguard peers add interface=wg-server public-key="CLIENT2_PUBLIC_KEY" allowed-address=10.13.13.3/32 persistent-keepalive=25s comment="Client 2 - Jane Smith"

# Client 3 with subnet access
/interface wireguard peers add interface=wg-server public-key="CLIENT3_PUBLIC_KEY" allowed-address=10.13.13.4/32,192.168.100.0/24 persistent-keepalive=25s comment="Client 3 - Site office"
```
{% endcode %}

***

## Client configuration

### Mobile client setup

For mobile devices, create a client configuration file:

{% code overflow="wrap" %}
```bash
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY_HERE
Address = 10.13.13.2/32
DNS = 1.1.1.1, 8.8.8.8

[Peer]
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = YOUR_SERVER_IP:13231
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```
{% endcode %}

### Linux client setup

{% code overflow="wrap" %}
```bash
# Install WireGuard
sudo apt update
sudo apt install wireguard

# Create client config file
sudo nano /etc/wireguard/wg0.conf

# Add configuration:
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY_HERE
Address = 10.13.13.2/32
DNS = 1.1.1.1

[Peer] 
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = YOUR_SERVER_IP:13231
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25

# Start WireGuard
sudo wg-quick up wg0

# Enable at boot
sudo systemctl enable wg-quick@wg0
```
{% endcode %}

### Windows client setup

1. **Install WireGuard** - Download from wireguard.com
2. **Create tunnel** - Import config file or create manually
3. **Configure tunnel** - Use the same configuration as above

***

## Advanced configurations

### Site-to-Site VPN

Connect two RouterOS devices with WireGuard:

**Site A (Router 1):**
{% code overflow="wrap" %}
```bash
# Create interface
/interface wireguard add name=wg-site-b listen-port=13231

# Assign IP
/ip address add address=172.16.0.1/30 interface=wg-site-b

# Add Site B as peer
/interface wireguard peers add interface=wg-site-b public-key="SITE_B_PUBLIC_KEY" allowed-address=172.16.0.2/32,192.168.2.0/24 endpoint=SITE_B_PUBLIC_IP:13231 persistent-keepalive=25s

# Add route to Site B network
/ip route add dst-address=192.168.2.0/24 gateway=wg-site-b
```
{% endcode %}

**Site B (Router 2):**
{% code overflow="wrap" %}
```bash
# Create interface
/interface wireguard add name=wg-site-a listen-port=13231

# Assign IP
/ip address add address=172.16.0.2/30 interface=wg-site-a

# Add Site A as peer
/interface wireguard peers add interface=wg-site-a public-key="SITE_A_PUBLIC_KEY" allowed-address=172.16.0.1/32,192.168.1.0/24 endpoint=SITE_A_PUBLIC_IP:13231 persistent-keepalive=25s

# Add route to Site A network
/ip route add dst-address=192.168.1.0/24 gateway=wg-site-a
```
{% endcode %}

### Road warrior + Site-to-Site

Combine mobile clients with site-to-site connections:

{% code overflow="wrap" %}
```bash
# Main hub router
/interface wireguard add name=wg-hub listen-port=13231
/ip address add address=10.99.0.1/16 interface=wg-hub

# Mobile client
/interface wireguard peers add interface=wg-hub public-key="MOBILE_CLIENT_KEY" allowed-address=10.99.1.2/32 persistent-keepalive=25s

# Branch office router
/interface wireguard peers add interface=wg-hub public-key="BRANCH_OFFICE_KEY" allowed-address=10.99.2.1/32,192.168.2.0/24 endpoint=BRANCH_IP:13231 persistent-keepalive=25s

# Another branch office
/interface wireguard peers add interface=wg-hub public-key="BRANCH2_OFFICE_KEY" allowed-address=10.99.3.1/32,192.168.3.0/24 endpoint=BRANCH2_IP:13231 persistent-keepalive=25s
```
{% endcode %}

***

## Security and best practices

### Key management

1. **Generate keys securely** - Use proper random number generation
2. **Rotate keys regularly** - Especially for long-term connections
3. **Secure key storage** - Protect private keys appropriately
4. **Unique keys per peer** - Never share keys between different peers

### Network design

1. **Use appropriate IP ranges** - Avoid conflicts with existing networks
2. **Implement firewall rules** - Control traffic between peers if needed
3. **Monitor connections** - Track active peers and their usage
4. **Plan for scalability** - Consider IP address allocation for growth

### Performance optimization

{% code overflow="wrap" %}
```bash
# Optimize for performance
/interface wireguard set wg-server mtu=1420

# Monitor performance
/interface wireguard monitor wg-server

# Check peer statistics
/interface wireguard peers print stats
```
{% endcode %}

***

<details>

<summary>Show complete WireGuard server setup</summary>

{% code overflow="wrap" %}
```bash
# 1. Create WireGuard interface
/interface wireguard add name=wg-server listen-port=13231

# 2. Assign IP address
/ip address add address=10.13.13.1/24 interface=wg-server

# 3. Add firewall rules
/ip firewall filter add chain=input action=accept protocol=udp dst-port=13231 comment="Allow WireGuard"
/ip firewall filter add chain=forward action=accept in-interface=wg-server comment="WireGuard forward in"
/ip firewall filter add chain=forward action=accept out-interface=wg-server comment="WireGuard forward out"

# 4. NAT for internet access
/ip firewall nat add chain=srcnat out-interface=ether1 src-address=10.13.13.0/24 action=masquerade comment="WireGuard NAT"

# 5. Add clients (replace PUBLIC_KEYs with actual keys)
/interface wireguard peers add interface=wg-server public-key="CLIENT1_PUBLIC_KEY" allowed-address=10.13.13.2/32 persistent-keepalive=25s comment="Mobile Client 1"
/interface wireguard peers add interface=wg-server public-key="CLIENT2_PUBLIC_KEY" allowed-address=10.13.13.3/32 persistent-keepalive=25s comment="Mobile Client 2"

# 6. Get server public key for client configuration
/interface wireguard print
```
{% endcode %}

</details>

## Troubleshooting

### Common issues

**Handshake failures:**
- Verify public keys are correct
- Check firewall allows UDP traffic on WireGuard port
- Ensure endpoint IP/port is reachable

**No internet access:**
- Check NAT rules are configured
- Verify AllowedIPs includes 0.0.0.0/0 on client
- Test DNS resolution

**Connection drops:**
- Enable persistent keepalive
- Check NAT timeout settings
- Verify network stability

### Monitoring and diagnostics

{% code overflow="wrap" %}
```bash
# Check interface status
/interface wireguard print detail

# Monitor peer connections
/interface wireguard peers print stats

# Check logs
/log print where topics~"wireguard"

# Monitor traffic
/interface monitor-traffic interface=wg-server
```
{% endcode %}

### Performance testing

{% code overflow="wrap" %}
```bash
# Test throughput
/tool bandwidth-test address=PEER_IP protocol=tcp

# Check MTU
/ping address=PEER_IP size=1472 do-not-fragment

# Monitor latency  
/ping address=PEER_IP count=10
```
{% endcode %}

