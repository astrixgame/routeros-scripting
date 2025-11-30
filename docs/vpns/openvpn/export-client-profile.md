---
description: This section covers how to create and export OpenVPN client configuration files manually.
icon: file-export
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

# Export client profile

{% hint style="info" %}
RouterOS doesn't have built-in client profile export. You need to manually create .ovpn files using exported certificates and server configuration.
{% endhint %}

Creating OpenVPN client profiles involves exporting certificates from RouterOS and manually creating the .ovpn configuration file with the proper settings.

***

## Exporting certificates from RouterOS

### Export CA certificate

First, export the Certificate Authority (CA) certificate:

In WinBox go to **System -> Certificates**, right-click on your **CA certificate** (e.g., LMTCA) and select **Export**:

* **File Name** - ca.crt
* **Type** - PEM
* **Export Passphrase** - Leave empty for CA certificate

{% code overflow="wrap" %}
```bash
/certificate export-certificate LMTCA export-passphrase="" type=pem
```
{% endcode %}

### Export client certificate and key

Export the client certificate with its private key:

In WinBox go to **System -> Certificates**, right-click on your **client certificate** and select **Export**:

* **File Name** - client.crt
* **Type** - PEM  
* **Export Passphrase** - Enter a secure passphrase

{% code overflow="wrap" %}
```bash
/certificate export-certificate CLIENT01 export-passphrase="YourSecurePassphrase123" type=pem
```
{% endcode %}

This will create two files:
- `cert_export_CLIENT01.crt` - Client certificate
- `cert_export_CLIENT01.key` - Client private key (encrypted)

### Download certificate files

Go to **Files** in WinBox and download the exported certificate files:
- `cert_export_LMTCA.crt` (CA certificate)
- `cert_export_CLIENT01.crt` (Client certificate)  
- `cert_export_CLIENT01.key` (Client private key)

***

## Creating the .ovpn configuration file

### Basic client configuration template

Create a new text file with `.ovpn` extension and add the following configuration:

{% code overflow="wrap" %}
```bash
# OpenVPN Client Configuration
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3

# Authentication method
auth-user-pass

# Certificate files
ca ca.crt
cert client.crt
key client.key

# Optional: Redirect all traffic through VPN
redirect-gateway def1

# Optional: Custom DNS servers
dhcp-option DNS 8.8.8.8
dhcp-option DNS 8.8.4.4
```
{% endcode %}

### Configuration parameters explanation

**Connection settings:**
- `remote` - Your server's public IP and port
- `proto` - Protocol (udp/tcp, must match server)
- `cipher` - Encryption cipher (must match server)
- `auth` - Authentication method (must match server)

**Security settings:**
- `tls-version-min` - Minimum TLS version
- `remote-cert-tls server` - Verify server certificate
- `auth-user-pass` - Enable username/password authentication

**Network settings:**
- `redirect-gateway def1` - Route all traffic through VPN
- `dhcp-option DNS` - Custom DNS servers

***

## Advanced configuration options

### Using inline certificates

Instead of separate certificate files, you can embed certificates directly in the .ovpn file:

{% code overflow="wrap" %}
```bash
# OpenVPN Client Configuration
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass

<ca>
-----BEGIN CERTIFICATE-----
[Paste CA certificate content here]
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
[Paste client certificate content here]
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
[Paste client private key content here]
-----END PRIVATE KEY-----
</key>
```
{% endcode %}

### Multiple server configurations

For redundancy, you can specify multiple server addresses:

{% code overflow="wrap" %}
```bash
# Primary server (UDP)
remote your-server.example.com 35324 udp

# Backup server (TCP)
remote your-server.example.com 443 tcp

# Backup by IP
remote 203.0.113.10 35324 udp
```
{% endcode %}

### Connection-specific routes

Add specific routes instead of redirecting all traffic:

{% code overflow="wrap" %}
```bash
# Don't redirect all traffic
# redirect-gateway def1

# Add specific routes for internal networks
route 192.168.1.0 255.255.255.0
route 10.0.0.0 255.0.0.0
```
{% endcode %}

***

## Client configuration templates

### Template for different client types

#### Full tunnel (all traffic through VPN)
{% code overflow="wrap" %}
```bash
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass
redirect-gateway def1
dhcp-option DNS 1.1.1.1
dhcp-option DNS 1.0.0.1

ca ca.crt
cert client.crt
key client.key
```
{% endcode %}

#### Split tunnel (specific networks only)
{% code overflow="wrap" %}
```bash
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass

# Only route specific networks through VPN
route 192.168.10.0 255.255.255.0
route 10.10.10.0 255.255.255.0

ca ca.crt
cert client.crt  
key client.key
```
{% endcode %}

#### Mobile client (with compression)
{% code overflow="wrap" %}
```bash
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass
redirect-gateway def1

# Mobile optimizations
compress lz4-v2
fast-io
sndbuf 0
rcvbuf 0

ca ca.crt
cert client.crt
key client.key
```
{% endcode %}

***

## Automated profile generation

### Using variables for template generation

You can create a template file and use variables for automated generation:

Create `client-template.ovpn`:
{% code overflow="wrap" %}
```bash
client
dev tun
proto {{ protocol | default('udp') }}
remote {{ server_address }} {{ server_port | default('35324') }}
resolv-retry infinite
nobind
persist-key
persist-tun
cipher {{ cipher | default('AES-256-CBC') }}
auth {{ auth | default('SHA256') }}
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass

{% if redirect_gateway %}
redirect-gateway def1
{% endif %}

{% if dns_servers %}
{% for dns in dns_servers %}
dhcp-option DNS {{ dns }}
{% endfor %}
{% endif %}

{% if inline_certs %}
<ca>
{{ ca_cert }}
</ca>

<cert>
{{ client_cert }}
</cert>

<key>
{{ client_key }}
</key>
{% else %}
ca ca.crt
cert {{ client_name }}.crt
key {{ client_name }}.key
{% endif %}
```
{% endcode %}

### Generate using template engine

Use a template engine (like Jinja2) to generate client configs:

{% code overflow="wrap" %}
```bash
# Generate client config with variables
jinja2 client-template.ovpn \
  -D server_address="vpn.example.com" \
  -D server_port="35324" \
  -D protocol="udp" \
  -D client_name="user1" \
  -D redirect_gateway=true \
  -D dns_servers='["1.1.1.1", "8.8.8.8"]' \
  > user1.ovpn
```
{% endcode %}

***

<details>

<summary>Show complete profile creation workflow</summary>

{% code overflow="wrap" %}
```bash
# 1. Export certificates from RouterOS
/certificate export-certificate LMTCA export-passphrase="" type=pem
/certificate export-certificate CLIENT01 export-passphrase="SecurePass123" type=pem

# 2. Download files from RouterOS Files section:
# - cert_export_LMTCA.crt
# - cert_export_CLIENT01.crt  
# - cert_export_CLIENT01.key

# 3. Create client.ovpn file with this content:
client
dev tun
proto udp
remote YOUR_SERVER_IP 35324
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-CBC
auth SHA256
tls-version-min 1.2
remote-cert-tls server
verb 3
auth-user-pass
redirect-gateway def1
dhcp-option DNS 1.1.1.1

ca cert_export_LMTCA.crt
cert cert_export_CLIENT01.crt
key cert_export_CLIENT01.key

# 4. Test connection
openvpn --config client.ovpn
```
{% endcode %}

</details>

## Client distribution

### Secure distribution methods

1. **Encrypted archive** - Put .ovpn and certificates in password-protected ZIP
2. **Secure file sharing** - Use encrypted file sharing services
3. **USB delivery** - For high-security environments
4. **Split delivery** - Send .ovpn and certificates separately

### Client setup instructions

**Windows OpenVPN GUI:**
1. Install OpenVPN GUI client
2. Copy .ovpn file to `C:\Program Files\OpenVPN\config\`
3. Right-click OpenVPN GUI tray icon and connect

**Android OpenVPN Connect:**
1. Install OpenVPN Connect app
2. Import .ovpn profile
3. Enter username/password when prompted

**iOS OpenVPN Connect:**
1. Install OpenVPN Connect app
2. Import profile via iTunes or email
3. Configure and connect

**Linux/macOS Terminal:**
```bash
sudo openvpn --config client.ovpn
```

## Troubleshooting profiles

### Common client issues

**Authentication failures:**
- Verify username/password are correct
- Check certificate files are not corrupted
- Ensure certificate hasn't expired

**Connection timeouts:**
- Verify server IP/port are correct
- Check firewall allows OpenVPN traffic
- Try different protocol (TCP vs UDP)

**Certificate errors:**
- Ensure CA certificate matches server CA
- Check client certificate is signed by same CA
- Verify certificate hasn't been revoked

**DNS issues:**
- Add custom DNS servers to config
- Check if DNS is being pushed by server
- Verify DNS resolution after connection

