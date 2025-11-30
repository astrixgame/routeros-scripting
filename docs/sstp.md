---
description: >-
  A Microsoft-owned VPN protocol that uses SSL encryption. It works in a
  Client-Server model, but is closed-source and lacks transparency.
icon: windows
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

# SSTP

{% hint style="warning" %}
SSTP is proprietary Microsoft technology with limited transparency. While more secure than PPTP, consider open-source alternatives like WireGuard or OpenVPN for better auditability.
{% endhint %}

In WinBox you can configure SSTP in **PPP -> SSTP Server**, or you can use terminal with command `/interface sstp-server`

SSTP (Secure Socket Tunneling Protocol) was developed by Microsoft as a replacement for PPTP, using SSL/TLS for encryption and better firewall traversal.

***

## SSTP overview

### Protocol characteristics

**Advantages:**
- Uses SSL/TLS encryption (port 443)
- Good firewall traversal (appears as HTTPS traffic)
- Better security than PPTP and L2TP/IPsec
- Native Windows support
- Can work through restrictive firewalls

**Disadvantages:**
- Proprietary Microsoft protocol
- Limited non-Windows client support
- Requires SSL certificates
- Performance overhead from SSL encapsulation
- Not independently auditable (closed source)

***

## Prerequisites

Before configuring SSTP, you need:

1. **SSL certificate** - Valid SSL certificate for your domain
2. **DNS resolution** - Server must be accessible by domain name
3. **Port 443 access** - Firewall must allow HTTPS traffic
4. **Time synchronization** - SSL certificates are time-sensitive

***

## Certificate preparation

### Using existing SSL certificate

If you have a valid SSL certificate (e.g., from Let's Encrypt or commercial CA):

{% code overflow="wrap" %}
```bash
# Import existing certificate (PFX format)
/certificate import file-name=your-cert.pfx passphrase="certificate_password"

# Or import PEM format
/certificate import file-name=ca-cert.pem
/certificate import file-name=server-cert.pem  
/certificate import file-name=server-key.pem
```
{% endcode %}

### Creating self-signed certificate

For testing or internal use only:

{% code overflow="wrap" %}
```bash
# Create CA certificate
/certificate add name=SSTP-CA common-name="SSTP CA" key-size=4096 days-valid=3650 key-usage=crl-sign,key-cert-sign
/certificate sign SSTP-CA

# Create server certificate
/certificate add name=SSTP-Server common-name="vpn.yourdomain.com" key-size=4096 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server
/certificate sign SSTP-Server ca=SSTP-CA
/certificate set SSTP-Server trusted=yes
```
{% endcode %}

***

## SSTP server configuration

### Basic server setup

In WinBox go to **PPP -> SSTP Server**:

* **Enabled** - Yes
* **Port** - 443 (standard HTTPS port)
* **Certificate** - Select your SSL certificate
* **Default Profile** - Select appropriate PPP profile
* **Authentication** - mschap2, mschap1, chap, pap
* **TLS Version** - 1.2 or higher

{% code overflow="wrap" %}
```bash
# Enable SSTP server
/interface sstp-server server set enabled=yes port=443 certificate=SSTP-Server default-profile=default-encryption authentication=mschap2 tls-version=only-1.2
```
{% endcode %}

### Create IP pool and profile

{% code overflow="wrap" %}
```bash
# Create IP pool for SSTP clients
/ip pool add name=sstp-pool ranges=192.168.150.2-192.168.150.50

# Create SSTP profile
/ppp profile add name=sstp-profile local-address=192.168.150.1 remote-address=sstp-pool dns-server=8.8.8.8,1.1.1.1 use-encryption=yes
```
{% endcode %}

### Update server to use custom profile

{% code overflow="wrap" %}
```bash
# Set SSTP server to use custom profile
/interface sstp-server server set default-profile=sstp-profile
```
{% endcode %}

***

## Firewall configuration

### Basic firewall rules

SSTP uses port 443 (HTTPS), which is commonly allowed:

{% code overflow="wrap" %}
```bash
# Allow SSTP (HTTPS) traffic
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=443 comment="Allow SSTP (HTTPS)"

# Allow forwarding for SSTP clients
/ip firewall filter add chain=forward action=accept in-interface=sstp-in comment="Allow SSTP client traffic"
```
{% endcode %}

### NAT configuration

If SSTP clients need internet access:

{% code overflow="wrap" %}
```bash
# NAT for SSTP clients
/ip firewall nat add chain=srcnat src-address=192.168.150.0/24 out-interface=ether1 action=masquerade comment="SSTP client NAT"
```
{% endcode %}

***

## User management

### Add SSTP users

Create users in **PPP -> Secrets**:

{% code overflow="wrap" %}
```bash
# Add SSTP users
/ppp secret add name=user1 password=SecurePass123! service=sstp profile=sstp-profile comment="SSTP User 1"
/ppp secret add name=user2 password=AnotherPass456! service=sstp profile=sstp-profile comment="SSTP User 2"

# User with specific IP
/ppp secret add name=admin password=AdminPass789! service=sstp profile=sstp-profile remote-address=192.168.150.10 comment="SSTP Admin"
```
{% endcode %}

### Advanced user settings

{% code overflow="wrap" %}
```bash
# User with rate limiting
/ppp secret add name=limited password=LimitPass123! service=sstp profile=sstp-profile rate-limit=10M/5M comment="Rate limited user"

# User with specific routes
/ppp secret add name=branch password=BranchPass456! service=sstp profile=sstp-profile routes="192.168.100.0/24 192.168.150.1" comment="Branch office"
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
   - Server name: vpn.yourdomain.com (must match certificate CN)
   - VPN type: SSTP
   - Username/Password: Your PPP credentials

### PowerShell configuration

{% code overflow="wrap" %}
```powershell
# Add SSTP VPN connection via PowerShell
Add-VpnConnection -Name "SSTP-VPN" -ServerAddress "vpn.yourdomain.com" -TunnelType "Sstp" -AuthenticationMethod MSChapv2 -SplitTunneling $false

# Connect to VPN
rasdial "SSTP-VPN" username password
```
{% endcode %}

### Linux SSTP client

Install and configure SSTP client on Linux:

{% code overflow="wrap" %}
```bash
# Install SSTP client (Ubuntu/Debian)
sudo apt update
sudo apt install sstp-client

# Create connection
sudo pppd call sstp-vpn

# Configuration file: /etc/ppp/peers/sstp-vpn
plugin sstp.so
sstp-sock /var/run/sstpc/sstpc-sstp-vpn
user "username"
password "password"
remotename "vpn.yourdomain.com"
refuse-eap
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
noauth
noipdefault
defaultroute
replacedefaultroute
usepeerdns
```
{% endcode %}

***

## Advanced configuration

### Certificate-based authentication

For enhanced security, use certificate-based client authentication:

{% code overflow="wrap" %}
```bash
# Generate client certificates
/certificate add name=SSTP-Client1 common-name="client1" key-size=4096 days-valid=365 key-usage=digital-signature,key-encipherment,tls-client
/certificate sign SSTP-Client1 ca=SSTP-CA
/certificate set SSTP-Client1 trusted=yes

# Export client certificate for installation on client device
/certificate export-certificate SSTP-Client1 export-passphrase="client_cert_password" type=pkcs12
```
{% endcode %}

### Multiple SSTP servers

Run multiple SSTP servers on different ports:

{% code overflow="wrap" %}
```bash
# Primary SSTP server (port 443)
/interface sstp-server server set enabled=yes port=443 certificate=SSTP-Server

# Secondary SSTP server (port 8443)
/interface sstp-server add name=sstp-backup port=8443 certificate=SSTP-Server profile=sstp-profile enabled=yes
```
{% endcode %}

### Custom SSL/TLS settings

{% code overflow="wrap" %}
```bash
# Configure advanced TLS settings
/interface sstp-server server set tls-version=only-1.2 verify-client-certificate=yes
```
{% endcode %}

***

## Troubleshooting

### Common issues

**Certificate errors:**
- Ensure certificate CN matches server hostname in client config
- Check certificate validity dates
- Verify certificate chain is complete
- Confirm certificate is trusted on RouterOS

**Connection failures:**
- Test basic connectivity to port 443
- Check firewall rules allow HTTPS traffic
- Verify DNS resolution of server hostname
- Test with self-signed certificate for debugging

**Authentication problems:**
- Verify username/password are correct
- Check PPP authentication methods match
- Ensure user account is enabled
- Review server-side authentication logs

### Diagnostic commands

{% code overflow="wrap" %}
```bash
# Check SSTP server status
/interface sstp-server print detail

# Monitor active SSTP sessions
/ppp active print where service=sstp

# View certificates
/certificate print detail where name~"SSTP"

# Check logs
/log print where topics~"sstp"
/log print where topics~"ppp"
```
{% endcode %}

### SSL/TLS debugging

{% code overflow="wrap" %}
```bash
# Test SSL certificate from client
openssl s_client -connect vpn.yourdomain.com:443 -servername vpn.yourdomain.com

# Check certificate details
/certificate print detail where name=SSTP-Server
```
{% endcode %}

***

## Security considerations

### Certificate security

1. **Use proper CA certificates** - Avoid self-signed for production
2. **Regular certificate rotation** - Replace certificates before expiration
3. **Strong private keys** - Use 4096-bit RSA or ECDSA keys
4. **Certificate revocation** - Implement proper revocation procedures

### Authentication security

{% code overflow="wrap" %}
```bash
# Use strong authentication
/interface sstp-server server set authentication=mschap2

# Require encryption
/ppp profile set sstp-profile use-encryption=yes

# Implement account lockout
/ppp secret set [find] disabled=yes comment="Locked due to failed attempts"
```
{% endcode %}

### Monitoring and logging

{% code overflow="wrap" %}
```bash
# Enable detailed logging
/system logging add topics=sstp,debug action=memory

# Monitor connection attempts
/log print where topics~"sstp" and message~"connect"

# Check for authentication failures
/log print where topics~"ppp" and message~"authentication failed"
```
{% endcode %}

***

<details>

<summary>Show complete SSTP setup</summary>

{% code overflow="wrap" %}
```bash
# 1. Create certificates
/certificate add name=SSTP-CA common-name="SSTP CA" key-size=4096 days-valid=3650 key-usage=crl-sign,key-cert-sign
/certificate sign SSTP-CA

/certificate add name=SSTP-Server common-name="vpn.yourdomain.com" key-size=4096 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server
/certificate sign SSTP-Server ca=SSTP-CA
/certificate set SSTP-Server trusted=yes

# 2. Create IP pool and profile
/ip pool add name=sstp-pool ranges=192.168.150.2-192.168.150.50
/ppp profile add name=sstp-profile local-address=192.168.150.1 remote-address=sstp-pool dns-server=8.8.8.8,1.1.1.1 use-encryption=yes

# 3. Enable SSTP server
/interface sstp-server server set enabled=yes port=443 certificate=SSTP-Server default-profile=sstp-profile authentication=mschap2 tls-version=only-1.2

# 4. Firewall rules
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=443 comment="Allow SSTP"
/ip firewall filter add chain=forward action=accept in-interface=sstp-in comment="SSTP forward"

# 5. NAT for internet access
/ip firewall nat add chain=srcnat src-address=192.168.150.0/24 out-interface=ether1 action=masquerade comment="SSTP NAT"

# 6. Add users
/ppp secret add name=user1 password=SecurePass123! service=sstp profile=sstp-profile
/ppp secret add name=admin password=AdminPass456! service=sstp profile=sstp-profile remote-address=192.168.150.10
```
{% endcode %}

</details>

## Performance optimization

### SSL optimization

{% code overflow="wrap" %}
```bash
# Optimize for performance
/interface sstp-server server set max-mtu=1500 max-mru=1500

# Monitor performance
/interface sstp-server monitor sstp-server
```
{% endcode %}

### Connection limits

{% code overflow="wrap" %}
```bash
# Set maximum concurrent connections
/interface sstp-server server set max-sessions=50
```
{% endcode %}

***

## Comparison with other VPN protocols

### SSTP vs OpenVPN

| Feature | SSTP | OpenVPN |
|---------|------|---------|
| **Encryption** | SSL/TLS | SSL/TLS or custom |
| **Port** | 443 only | Configurable |
| **Open source** | ❌ Proprietary | ✅ Open source |
| **Windows support** | ✅ Native | ✅ Client required |
| **Linux support** | ⚠️ Third-party | ✅ Native |
| **Firewall traversal** | ✅ Excellent | ✅ Good |
| **Performance** | ⚠️ Overhead | ✅ Optimized |

### SSTP vs WireGuard

| Feature | SSTP | WireGuard |
|---------|------|-----------|
| **Security audit** | ❌ Proprietary | ✅ Audited |
| **Performance** | ⚠️ SSL overhead | ✅ Minimal overhead |
| **Setup complexity** | ⚠️ Certificates required | ✅ Simple keys |
| **Compatibility** | ⚠️ Limited | ✅ Wide support |
| **Maintenance** | ⚠️ Certificate mgmt | ✅ Minimal |

***

## Migration recommendations

### When to use SSTP

**Consider SSTP for:**
- Windows-centric environments
- Situations where only port 443 is allowed outbound
- Legacy systems requiring Windows built-in VPN support
- Temporary solutions during migration to better protocols

### Migration paths

**From SSTP to WireGuard:**
{% code overflow="wrap" %}
```bash
# Deploy WireGuard alongside SSTP
/interface wireguard add name=wg-server listen-port=13231
/ip address add address=10.13.13.1/24 interface=wg-server

# Gradually migrate users
# Eventually disable SSTP
/interface sstp-server server set enabled=no
```
{% endcode %}

**From SSTP to OpenVPN:**
{% code overflow="wrap" %}
```bash
# Deploy OpenVPN on alternate port
/interface ovpn-server server add name=ovpn-server port=1194 certificate=SSTP-Server tls-version=only-1.2

# Migrate users with new client configurations
# Disable SSTP when migration complete
/interface sstp-server server set enabled=no
```
{% endcode %}
