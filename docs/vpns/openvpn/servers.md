---
description: This section covers how to configure OpenVPN servers on RouterOS.
icon: server
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

# Servers

{% hint style="info" %}
OpenVPN server requires certificates to be configured first. Make sure you have completed the certificate setup.
{% endhint %}

In WinBox you can configure OpenVPN servers in **Interfaces -> OpenVPN Server**, there you can click to <kbd>**+**</kbd> to create new server or you can use terminal with command `/interface ovpn-server server`

***

## Basic server configuration

### Creating server interface

Click on **`+`** to add new OpenVPN server

You will need to fill out:

* **Name** - Your server interface name (e.g. "ovpn-server1")
* **Port** - OpenVPN port (default is 1194, consider using custom port like 35324 for security)
* **Protocol** - Choose "tcp" or "udp" (udp is recommended for better performance)
* **Certificate** - Select your server certificate created earlier
* **Require Client Certificate** - Enable this for certificate-based authentication
* **Auth** - Authentication method (sha256, sha512 recommended)
* **Cipher** - Encryption cipher (aes256 recommended)
* **TLS Version** - Set to "only-1.2" or higher for security
* **User Auth Method** - Authentication method for users (mschap2, eap-mschap2)
* **Default Profile** - Select your custom VPN profile
* **Netmask** - Network prefix length (usually 20 or 24)

{% code overflow="wrap" %}
```bash
/interface ovpn-server server add name=ovpn-server1 port=35324 certificate=SERVER auth=sha256 cipher=aes256 require-client-certificate=yes tls-version=only-1.2 user-auth-method=mschap2 default-profile=vpn-users netmask=20 disabled=no
```
{% endcode %}

### Advanced server settings

For enhanced security and performance, you can configure additional parameters:

* **Max MTU** - Maximum transmission unit (usually 1500)
* **Keepalive Timeout** - Connection keepalive in seconds
* **Push Routes** - Routes to push to clients (leave empty for default)
* **Redirect Gateway** - Force all client traffic through VPN
* **MAC Address** - Custom MAC address for the interface

{% code overflow="wrap" %}
```bash
/interface ovpn-server server set ovpn-server1 max-mtu=1500 keepalive-timeout=60 push-routes="" redirect-gateway=def1
```
{% endcode %}

***

## Firewall configuration

### Allow OpenVPN through firewall

You need to create firewall rules to allow OpenVPN traffic.

In WinBox go to **IP -> Firewall -> Filter Rules** and add new rule:

* **Chain** - input
* **Protocol** - udp (or tcp if using TCP)
* **Dst. Port** - 1194 (or your custom port)
* **Action** - accept

{% code overflow="wrap" %}
```bash
/ip firewall filter add chain=input action=accept protocol=udp dst-port=1194 comment="Allow OpenVPN"
```
{% endcode %}

### NAT configuration (if needed)

If your RouterOS is behind NAT, you need to forward the OpenVPN port:

{% code overflow="wrap" %}
```bash
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=192.168.1.1 to-ports=1194 protocol=udp dst-port=1194 comment="OpenVPN port forward"
```
{% endcode %}

***

## Multiple server instances

You can create multiple OpenVPN server instances for different purposes:

### Server for different user groups

Create additional server instances with different profiles:

{% code overflow="wrap" %}
```bash
# Management VPN server
/interface ovpn-server server add name=ovpn-mgmt port=1195 certificate=SERVER auth=sha256 cipher=aes256 tls-version=only-1.2 user-auth-method=mschap2 default-profile=mgmt-users netmask=24 disabled=no

# Guest VPN server  
/interface ovpn-server server add name=ovpn-guest port=1196 certificate=SERVER auth=sha256 cipher=aes128 tls-version=only-1.2 user-auth-method=mschap2 default-profile=guest-users netmask=24 disabled=no
```
{% endcode %}

### Different protocols

You can run both TCP and UDP servers simultaneously:

{% code overflow="wrap" %}
```bash
# UDP server (recommended)
/interface ovpn-server server set ovpn-server1 protocol=udp port=35324

# TCP server (for restrictive networks)
/interface ovpn-server server add name=ovpn-tcp port=443 protocol=tcp certificate=SERVER tls-version=only-1.2 user-auth-method=mschap2 default-profile=vpn-users netmask=20 disabled=no
```
{% endcode %}

***

<details>

<summary>Show complete server configuration</summary>

{% code overflow="wrap" %}
```bash
# Enable OpenVPN server with modern secure settings
/interface ovpn-server server add name=ovpn-server1 port=35324 certificate=SERVER auth=sha256 cipher=aes256 require-client-certificate=yes tls-version=only-1.2 user-auth-method=mschap2 default-profile=vpn-users netmask=20 protocol=udp max-mtu=1500 disabled=no

# Allow OpenVPN through firewall
/ip firewall filter add chain=input action=accept protocol=udp dst-port=35324 comment="Allow OpenVPN UDP"

# Optional: Create additional TCP server for restrictive networks
/interface ovpn-server server add name=ovpn-tcp port=443 protocol=tcp certificate=SERVER auth=sha256 cipher=aes256 tls-version=only-1.2 user-auth-method=mschap2 default-profile=vpn-users netmask=20 disabled=no

# Allow TCP OpenVPN through firewall
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=443 comment="Allow OpenVPN TCP"
```
{% endcode %}

</details>

## Troubleshooting

### Common issues

**Server not starting:**
- Check if certificate is properly signed and trusted
- Verify firewall rules allow the configured port
- Ensure profile is correctly configured with IP pools

**Clients cannot connect:**
- Verify client certificates are signed by the same CA
- Check if server is listening on correct interface
- Test connectivity to the OpenVPN port from external network

**Performance issues:**
- Use UDP protocol instead of TCP when possible
- Adjust MTU size if experiencing packet fragmentation
- Consider using hardware with AES acceleration

### Monitoring connections

Check active OpenVPN connections:

{% code overflow="wrap" %}
```bash
# View server status and configuration
/interface ovpn-server server print detail

# Monitor active connections
/ppp active print

# Check server statistics and connection info
/interface ovpn-server server monitor ovpn-server1

# View connection logs
/log print where topics~"ovpn"
```
{% endcode %}

