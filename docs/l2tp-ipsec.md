---
description: >-
  A VPN protocol that combines Layer 2 Tunneling Protocol with IPsec for
  encryption. It operates in a Client-Server model and is considered outdated in
  terms of security.
icon: shield-exclamation
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

# L2TP/IPsec

{% hint style="warning" %}
L2TP/IPsec is outdated and lacks modern cryptographic standards. Consider using WireGuard or modern OpenVPN instead. This documentation is provided for legacy support only.
{% endhint %}

In WinBox you can configure L2TP/IPsec in **PPP -> Interface Templates** and **IP -> IPSec**, or you can use terminal with commands `/interface l2tp-server` and `/ip ipsec`

L2TP (Layer 2 Tunneling Protocol) provides tunneling but no encryption, so it's combined with IPsec for security.

***

## Prerequisites

Before configuring L2TP/IPsec, ensure you have:

1. **Static public IP** or working DDNS
2. **Proper firewall rules** for IPsec and L2TP traffic  
3. **Time synchronization** - IPsec is sensitive to time differences
4. **Certificate or PSK** for IPsec authentication

***

## Basic L2TP/IPsec server setup

### Enable L2TP server

In WinBox go to **PPP -> L2TP Server** and configure:

* **Enabled** - Yes
* **Use IPsec** - Yes  
* **IPsec Secret** - Strong pre-shared key
* **Default Profile** - Select appropriate PPP profile
* **Authentication** - mschap2, mschap1, chap, pap (mschap2 recommended)

{% code overflow="wrap" %}
```bash
# Enable L2TP server with IPsec
/interface l2tp-server server set enabled=yes use-ipsec=yes ipsec-secret="YourStrongPSK123!" default-profile=default-encryption authentication=mschap2
```
{% endcode %}

### Create IP pool for clients

Create an IP pool for L2TP clients:

{% code overflow="wrap" %}
```bash
# Create IP pool for L2TP clients
/ip pool add name=l2tp-pool ranges=192.168.100.2-192.168.100.50
```
{% endcode %}

### Configure PPP profile

Create or modify a PPP profile for L2TP clients:

In WinBox go to **PPP -> Profiles** and create/edit profile:

* **Name** - l2tp-profile  
* **Local Address** - Gateway IP (e.g., 192.168.100.1)
* **Remote Address** - l2tp-pool
* **DNS Server** - Your DNS servers
* **Use Encryption** - Yes

{% code overflow="wrap" %}
```bash
# Create L2TP profile
/ppp profile add name=l2tp-profile local-address=192.168.100.1 remote-address=l2tp-pool dns-server=192.168.1.1,8.8.8.8 use-encryption=yes
```
{% endcode %}

### Update L2TP server to use custom profile

{% code overflow="wrap" %}
```bash
# Set L2TP server to use custom profile
/interface l2tp-server server set default-profile=l2tp-profile
```
{% endcode %}

***

## IPsec configuration

### IPsec policy configuration

L2TP/IPsec requires specific IPsec policies. RouterOS should auto-generate these when you enable "use-ipsec", but you can configure manually:

{% code overflow="wrap" %}
```bash
# IPsec policy for L2TP (usually auto-generated)
/ip ipsec policy add src-address=0.0.0.0/0 dst-address=0.0.0.0/0 protocol=udp src-port=1701 dst-port=1701 action=encrypt level=require ipsec-protocols=esp tunnel=no sa-src-address=0.0.0.0 sa-dst-address=0.0.0.0 proposal=default
```
{% endcode %}

### IPsec proposal settings

Configure encryption and authentication methods:

{% code overflow="wrap" %}
```bash
# Create custom IPsec proposal (optional - default usually sufficient)
/ip ipsec proposal add name=l2tp-proposal auth-algorithms=sha1 enc-algorithms=aes-256-cbc,aes-192-cbc,aes-128-cbc pfs-group=modp1024

# Use custom proposal
/ip ipsec policy set [find] proposal=l2tp-proposal
```
{% endcode %}

***

## Firewall configuration

### Required firewall rules

L2TP/IPsec requires multiple firewall rules:

{% code overflow="wrap" %}
```bash
# Allow IPsec ESP
/ip firewall filter add chain=input action=accept protocol=ipsec-esp comment="Allow IPsec ESP"

# Allow IPsec AH  
/ip firewall filter add chain=input action=accept protocol=ipsec-ah comment="Allow IPsec AH"

# Allow IKE (IPsec key exchange)
/ip firewall filter add chain=input action=accept protocol=udp dst-port=500 comment="Allow IKE"

# Allow IPsec NAT-T
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4500 comment="Allow IPsec NAT-T"

# Allow L2TP
/ip firewall filter add chain=input action=accept protocol=udp dst-port=1701 comment="Allow L2TP"

# Allow forwarding for L2TP clients
/ip firewall filter add chain=forward action=accept in-interface=l2tp-in comment="Allow L2TP client traffic"
```
{% endcode %}

### NAT configuration (if needed)

If L2TP clients need internet access:

{% code overflow="wrap" %}
```bash
# NAT for L2TP clients
/ip firewall nat add chain=srcnat src-address=192.168.100.0/24 out-interface=ether1 action=masquerade comment="L2TP client NAT"
```
{% endcode %}

***

## User management

### Create L2TP users

Add users in **PPP -> Secrets**:

* **Name** - Username
* **Password** - Strong password  
* **Service** - l2tp
* **Profile** - l2tp-profile

{% code overflow="wrap" %}
```bash
# Add L2TP users
/ppp secret add name=user1 password=SecurePass123! service=l2tp profile=l2tp-profile comment="L2TP User 1"
/ppp secret add name=user2 password=AnotherPass456! service=l2tp profile=l2tp-profile comment="L2TP User 2"
```
{% endcode %}

### User with specific IP

Assign specific IP to a user:

{% code overflow="wrap" %}
```bash
# User with fixed IP
/ppp secret add name=admin password=AdminPass789! service=l2tp profile=l2tp-profile remote-address=192.168.100.10 comment="L2TP Admin"
```
{% endcode %}

***

## Client configuration

### Windows built-in client

**Windows 10/11:**
1. **Settings** → **Network & Internet** → **VPN**
2. **Add VPN connection**:
   - VPN Provider: Windows (built-in)
   - Connection name: Your VPN name
   - Server name: Your server IP/domain
   - VPN type: L2TP/IPsec with pre-shared key
   - Pre-shared key: Your IPsec secret
   - Username/Password: Your PPP credentials

**Registry fix for Windows (often required):**
```registry
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent]
"AssumeUDPEncapsulationContextOnSendRule"=dword:00000002
```

### Android configuration

**Android built-in VPN:**
1. **Settings** → **Network & Internet** → **VPN**
2. **Add VPN**:
   - Type: L2TP/IPsec PSK  
   - Server address: Your server IP
   - L2TP secret: (leave empty)
   - IPsec pre-shared key: Your IPsec secret
   - Username/Password: Your PPP credentials

### iOS configuration

**iOS built-in VPN:**
1. **Settings** → **General** → **VPN & Device Management** → **VPN**
2. **Add VPN Configuration**:
   - Type: L2TP
   - Server: Your server IP
   - Account: Username
   - Password: Password  
   - Secret: IPsec pre-shared key

***

## Advanced configuration

### Certificate-based authentication

Instead of PSK, you can use certificates (more secure but complex):

{% code overflow="wrap" %}
```bash
# Generate certificate for IPsec (same process as OpenVPN)
/certificate add name=ipsec-ca common-name="IPsec CA" key-size=4096 days-valid=3650 key-usage=crl-sign,key-cert-sign
/certificate sign ipsec-ca

# Server certificate
/certificate add name=ipsec-server common-name="vpn.example.com" key-size=4096 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server
/certificate sign ipsec-server ca=ipsec-ca
/certificate set ipsec-server trusted=yes

# Configure IPsec to use certificates
/ip ipsec peer add address=0.0.0.0/0 certificate=ipsec-server
```
{% endcode %}

### Multiple L2TP servers

You can run multiple L2TP server instances:

{% code overflow="wrap" %}
```bash
# Create additional L2TP server interface
/interface l2tp-server add name=l2tp-server2 user=user3 password=pass3 profile=l2tp-profile disabled=no

# Different authentication methods
/interface l2tp-server server set authentication=mschap2,mschap1
```
{% endcode %}

***

## Troubleshooting

### Common issues

**Phase 1 failures (IKE):**
- Check IPsec secret matches on both ends
- Verify firewall allows UDP 500 and 4500
- Check time synchronization between client and server
- Ensure correct authentication method

**Phase 2 failures (ESP):**
- Verify L2TP server is enabled
- Check firewall allows UDP 1701 and ESP protocol
- Ensure PPP authentication credentials are correct

**Windows-specific issues:**
- Apply registry fix for UDP encapsulation
- Disable "Use default gateway on remote network" if not needed
- Check Windows firewall settings

### Diagnostic commands

{% code overflow="wrap" %}
```bash
# Check L2TP server status
/interface l2tp-server print detail

# Monitor active L2TP sessions
/ppp active print where service=l2tp

# Check IPsec status  
/ip ipsec active-peers print
/ip ipsec installed-sa print

# View logs
/log print where topics~"ipsec"
/log print where topics~"l2tp"
```
{% endcode %}

### Performance optimization

{% code overflow="wrap" %}
```bash
# Optimize for better performance
/ip ipsec proposal set [find] enc-algorithms=aes-128-cbc pfs-group=none

# Monitor bandwidth
/interface monitor-traffic interface=l2tp-in
```
{% endcode %}

***

<details>

<summary>Show complete L2TP/IPsec setup</summary>

{% code overflow="wrap" %}
```bash
# 1. Create IP pool
/ip pool add name=l2tp-pool ranges=192.168.100.2-192.168.100.50

# 2. Create PPP profile
/ppp profile add name=l2tp-profile local-address=192.168.100.1 remote-address=l2tp-pool dns-server=8.8.8.8,1.1.1.1 use-encryption=yes

# 3. Enable L2TP server
/interface l2tp-server server set enabled=yes use-ipsec=yes ipsec-secret="YourStrongPSK123!" default-profile=l2tp-profile authentication=mschap2

# 4. Firewall rules
/ip firewall filter add chain=input action=accept protocol=ipsec-esp comment="IPsec ESP"
/ip firewall filter add chain=input action=accept protocol=ipsec-ah comment="IPsec AH"  
/ip firewall filter add chain=input action=accept protocol=udp dst-port=500 comment="IKE"
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4500 comment="IPsec NAT-T"
/ip firewall filter add chain=input action=accept protocol=udp dst-port=1701 comment="L2TP"
/ip firewall filter add chain=forward action=accept in-interface=l2tp-in comment="L2TP forward"

# 5. NAT for internet access
/ip firewall nat add chain=srcnat src-address=192.168.100.0/24 out-interface=ether1 action=masquerade comment="L2TP NAT"

# 6. Add users
/ppp secret add name=user1 password=SecurePass123! service=l2tp profile=l2tp-profile
/ppp secret add name=user2 password=AnotherPass456! service=l2tp profile=l2tp-profile
```
{% endcode %}

</details>

## Migration recommendations

### Why migrate from L2TP/IPsec

1. **Outdated cryptography** - Uses older encryption methods
2. **Complex NAT traversal** - Requires multiple ports and protocols
3. **Performance issues** - Higher overhead compared to modern VPNs
4. **Limited mobile support** - Inconsistent behavior across devices

### Migration paths

**To WireGuard:**
- Modern cryptography with better performance
- Simpler configuration and better mobile support  
- Native support in RouterOS v7+

**To OpenVPN:**
- Better compatibility across all platforms
- More configuration flexibility
- Established security track record

### Coexistence period

You can run L2TP/IPsec alongside modern VPNs during migration:

{% code overflow="wrap" %}
```bash
# Run L2TP on non-standard port during migration
/interface l2tp-server server set port=1702

# Adjust firewall rules accordingly
/ip firewall filter set [find comment="Allow L2TP"] dst-port=1702
```
{% endcode %}
