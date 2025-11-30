---
description: This section covers all methods to connect to your MikroTik router including WinBox, SSH, web interface, and MAC-Telnet for management and configuration.
icon: plug
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

# Connecting to MT

{% hint style="info" %}
MikroTik RouterOS provides multiple connection methods including the graphical WinBox interface, command-line SSH access, web-based management, and MAC-layer protocols for initial setup and ongoing administration.
{% endhint %}

RouterOS offers several management interfaces suitable for different scenarios, from initial setup to remote administration and scripting automation.

### Default credentials

You can find default credentials, including the MAC address, on the MikroTik device label typically located at the bottom. Here's an example:

<div align="center"><figure><img src="../.gitbook/assets/mikrotik.png" alt="Device Label" width="250"><figcaption><p>Device Label</p></figcaption></figure></div>

***

### Using WinBox

{% stepper %}
{% step %}
### Download & Install WinBox

{% tabs %}
{% tab title="Windows" %}
You can download [latest version](https://mikrotik.com/download) of WinBox from official website.

{% embed url="https://mikrotik.com/download" %}
MikroTik Downloads
{% endembed %}
{% endtab %}

{% tab title="Linux" %}
First things first you will need **snap package manager**, you can install it using:

```bash
sudo apt update
sudo apt install snapd
```

Then you can simply install WinBox using snap:

```bash
sudo snap install winbox
```

And you are ready to go
{% endtab %}

{% tab title="Mac" %}
First things first you will need **Wine** which you can download and install from GitHub:

{% @github-files/github-code-block url="https://github.com/Gcenx/macOS_Wine_builds/releases" %}

Then download [latest version](https://mikrotik.com/download) of WinBox from official website:

{% embed url="https://mikrotik.com/download" %}
MikroTik Downloads
{% endembed %}

And launch W**inBox64.exe** using W**ine64.app**
{% endtab %}

{% tab title="Android" %}
Simply go to **Google Play Store** and search for **Mikrotik Pro**.

{% embed url="https://play.google.com/store/apps/details?id=com.mikrotik.android.tikapp" %}
Google Play Store - MikroTik Pro
{% endembed %}
{% endtab %}
{% endtabs %}


{% endstep %}

{% step %}
### Connect WinBox to MT

You can enter either your **IP** or **MAC address** with user and password manually or you can click on **Neighbors Tab** and find your MT and just enter user and password.

<figure><img src="../.gitbook/assets/winbox.png" alt="" width="563"><figcaption><p>WinBox Login</p></figcaption></figure>
{% endstep %}

{% step %}
### Start configuring

When you connect for the first time you will be prompt to change the password.

<figure><img src="../.gitbook/assets/winbox_config.png" alt=""><figcaption><p>WinBox Config</p></figcaption></figure>
{% endstep %}
{% endstepper %}



***

## Connection methods overview

### Available connection methods

**Graphical interfaces:**
- **WinBox** - Native Windows/Linux/Mac management tool
- **WebFig** - Web-based configuration interface  
- **QuickSet** - Simplified web setup wizard
- **Mobile apps** - Android/iOS MikroTik applications

**Command line interfaces:**
- **SSH** - Secure Shell for encrypted terminal access
- **Telnet** - Unencrypted terminal access (not recommended)
- **Serial console** - Direct hardware connection
- **MAC-Telnet** - Layer-2 protocol for initial access

**API interfaces:**
- **REST API** - HTTP-based programming interface
- **API** - Native MikroTik programming protocol
- **SNMP** - Network monitoring protocol

***

## SSH Connection

### Using SSH from different platforms

{% tabs %}
{% tab title="Windows" %}
**Built-in SSH client (Windows 10+):**
{% code overflow="wrap" %}
```bash
# Connect using default port
ssh admin@192.168.88.1

# Connect using custom port
ssh admin@192.168.88.1 -p 2222

# Connect with specific identity file
ssh -i ~/.ssh/mikrotik_key admin@192.168.88.1
```
{% endcode %}

**Using PuTTY:**
1. Download PuTTY from [putty.org](https://putty.org)
2. Enter router IP address and port 22
3. Set connection type to SSH
4. Click "Open" and login with credentials
{% endtab %}

{% tab title="Linux/Mac" %}
**Terminal SSH connection:**
{% code overflow="wrap" %}
```bash
# Basic connection
ssh admin@192.168.88.1

# Connection with custom port
ssh admin@192.168.88.1 -p 2222

# Connection with key authentication
ssh -i ~/.ssh/mikrotik_private_key admin@192.168.88.1

# Connection with specific cipher (for older RouterOS)
ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 admin@192.168.88.1
```
{% endcode %}

**Generate SSH keys for key-based authentication:**
{% code overflow="wrap" %}
```bash
# Generate new SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/mikrotik_key

# Copy public key content
cat ~/.ssh/mikrotik_key.pub
```
{% endcode %}
{% endtab %}
{% endtabs %}

### SSH key authentication setup

Configure SSH keys for passwordless authentication:

{% code overflow="wrap" %}
```bash
# On RouterOS - import SSH public key
/user ssh-keys import public-key-file=mikrotik_key.pub user=admin

# Or add key directly via command line
/user ssh-keys add key-data="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC..." user=admin comment="Management workstation"

# Verify imported keys
/user ssh-keys print

# Test connection from client
ssh -i ~/.ssh/mikrotik_key admin@192.168.88.1
```
{% endcode %}

### SSH security hardening

Secure SSH access configuration:

{% code overflow="wrap" %}
```bash
# Change SSH port from default
/ip service set ssh port=2222

# Restrict SSH access to specific networks
/ip service set ssh address=192.168.1.0/24,10.0.0.0/8

# Disable password authentication (keys only)
/ip ssh set strong-crypto=yes always-allow-password-login=no

# Set SSH timeout
/ip ssh set forwarding-enabled=no host-key-size=2048

# Check SSH configuration
/ip ssh print
/ip service print where name=ssh
```
{% endcode %}

***

## Web interface access

### WebFig configuration interface

Access the web interface for graphical configuration:

{% code overflow="wrap" %}
```bash
# Default web interface access
http://192.168.88.1

# Secure HTTPS access (if configured)
https://192.168.88.1:8443

# Configure HTTPS certificate
/certificate add name=webfig-cert common-name=router.local days-valid=3650 key-size=2048
/certificate sign webfig-cert
/ip service set www-ssl certificate=webfig-cert disabled=no port=8443

# Restrict web access to LAN only
/ip service set www address=192.168.88.0/24
/ip service set www-ssl address=192.168.88.0/24
```
{% endcode %}

### QuickSet initial configuration

Use QuickSet for rapid initial setup:

1. **Access QuickSet**: Navigate to router IP in web browser
2. **Choose mode**: Home AP, Router, Bridge, etc.
3. **Configure basics**: WiFi, passwords, internet connection
4. **Apply settings**: QuickSet applies comprehensive configuration

***

## MAC-Telnet connection

### Using MAC-Telnet for layer-2 access

MAC-Telnet allows connection without IP configuration:

{% code overflow="wrap" %}
```bash
# From Linux with mactelnet package
sudo apt install mactelnet-client
mactelnet -u admin MAC-ADDRESS

# From Windows using WinBox
# Use "Neighbors" tab in WinBox to discover devices by MAC

# From another RouterOS device
/tool mac-telnet MAC-ADDRESS

# Enable MAC-Telnet server on RouterOS
/ip neighbor discovery-settings set discover-interface-list=LAN
/tool mac-server set allowed-interface-list=LAN
/tool mac-server mac-winbox set allowed-interface-list=LAN
```
{% endcode %}

### MAC-based discovery

Discover MikroTik devices on the network:

{% code overflow="wrap" %}
```bash
# From Linux
mndp-scan

# From RouterOS device
/ip neighbor discovery print

# Check MAC server status
/tool mac-server print
/tool mac-server mac-winbox print
```
{% endcode %}

***

## Serial console connection

### Physical serial connection

Direct hardware connection for recovery:

**Connection parameters:**
- **Baud rate**: 115200
- **Data bits**: 8  
- **Parity**: None
- **Stop bits**: 1
- **Flow control**: None

{% code overflow="wrap" %}
```bash
# Linux serial connection
screen /dev/ttyUSB0 115200

# Or using minicom
minicom -D /dev/ttyUSB0 -b 115200

# Windows using PuTTY
# Set connection type to "Serial"
# Enter COM port and baud rate 115200
```
{% endcode %}

### Console cable requirements

**RB series devices**: DB9 serial cable (null-modem)
**CCR series**: RJ45 console cable  
**USB-to-serial adapters**: Use FTDI or similar reliable chips

***

## API access

### REST API usage

Modern HTTP-based API access:

{% code overflow="wrap" %}
```bash
# Enable REST API
/ip service set api disabled=no port=8728
/ip service set api-ssl disabled=no port=8729

# Example REST API calls
curl -k -u admin:password https://192.168.88.1:8729/rest/interface
curl -k -u admin:password https://192.168.88.1:8729/rest/ip/address

# Python example using requests
import requests
response = requests.get('https://192.168.88.1:8729/rest/interface', 
                       auth=('admin', 'password'), verify=False)
```
{% endcode %}

### Native API protocol

Binary API for custom applications:

{% code overflow="wrap" %}
```python
# Python example using librouteros
from librouteros import connect

api = connect(host='192.168.88.1', username='admin', password='password')
interfaces = api('/interface/print')
for interface in interfaces:
    print(interface['name'], interface['type'])
```
{% endcode %}

***

## Connection troubleshooting

### Common connection issues

{% code overflow="wrap" %}
```bash
# Issue: Cannot connect to router
# Check network connectivity
ping 192.168.88.1

# Check if services are enabled
/ip service print

# Verify firewall rules allow management
/ip firewall filter print where chain=input

# Issue: SSH connection refused
# Check SSH service status
/ip service print where name=ssh

# Verify SSH is not blocked by firewall
/ip firewall filter print where protocol=tcp and dst-port=22

# Issue: Forgotten password
# Reset using reset button or serial console
# Hold reset button for 10+ seconds after power on
```
{% endcode %}

### Reset and recovery procedures

{% code overflow="wrap" %}
```bash
# Software reset via terminal
/system reset-configuration

# Hardware reset button sequence
# 1. Power off device
# 2. Hold reset button
# 3. Power on while holding reset
# 4. Wait for LED sequence (varies by model)
# 5. Release reset button

# Netinstall recovery (for serious issues)
# 1. Download Netinstall from MikroTik website
# 2. Boot device in Netinstall mode
# 3. Use Netinstall to reflash RouterOS
```
{% endcode %}

### Network discovery tools

{% code overflow="wrap" %}
```bash
# Discover MikroTik devices on network
# Using nmap to scan for RouterOS
nmap -sS -O 192.168.1.0/24 | grep -B 5 "MikroTik"

# Using MikroTik discovery protocol
# From Linux with mactelnet
mndp-scan

# From Windows
# WinBox Neighbors tab
# Use "..." -> "Discovery" in WinBox
```
{% endcode %}

***

## Advanced connection scenarios

### Connection through VPN

Access router through VPN tunnels:

{% code overflow="wrap" %}
```bash
# SSH through OpenVPN tunnel
ssh admin@10.8.0.1

# WinBox through IPSec tunnel  
# Connect to tunnel endpoint IP
# Use WinBox with router's tunnel IP

# WebFig through WireGuard
# Access via WireGuard interface IP
https://10.13.13.1:8443
```
{% endcode %}

### Jump host connections

SSH through intermediate servers:

{% code overflow="wrap" %}
```bash
# SSH through jump host
ssh -J jumphost@gateway.example.com admin@192.168.88.1

# SSH with port forwarding
ssh -L 8291:192.168.88.1:8291 user@jumphost
# Then connect WinBox to localhost:8291

# ProxyCommand method
ssh -o ProxyCommand='ssh user@jumphost nc 192.168.88.1 22' admin@192.168.88.1
```
{% endcode %}

### Automated connections

Script connections for automation:

{% code overflow="wrap" %}
```bash
# SSH with expect script
#!/usr/bin/expect
spawn ssh admin@192.168.88.1
expect "Password:"
send "mypassword\r"
expect ">"
send "/system identity print\r"
expect ">"
send "quit\r"

# SSH with key authentication for scripts
ssh -i ~/.ssh/mikrotik_key -o StrictHostKeyChecking=no admin@192.168.88.1 '/system resource print'

# Batch commands via SSH
echo "/system resource print; /interface print" | ssh admin@192.168.88.1
```
{% endcode %}

***

<details>

<summary>Show complete connection troubleshooting guide</summary>

{% code overflow="wrap" %}
```bash
# Complete connection troubleshooting checklist

# 1. Physical connectivity
ping 192.168.88.1                    # Test basic IP connectivity
arp -a | grep 192.168.88.1           # Check ARP table entry
netstat -an | grep :8291              # Check if WinBox port is reachable

# 2. Service status verification
/ip service print                     # Check enabled services
/interface print brief                # Verify interface status
/ip address print                     # Check IP configuration

# 3. Firewall verification
/ip firewall filter print where chain=input    # Check input rules
/log print where topics~"firewall"             # Check firewall logs

# 4. User account verification
/user print                           # Check user accounts
/user active print                    # Check active sessions

# 5. Certificate issues (HTTPS/API-SSL)
/certificate print                    # Check certificate status
/certificate sign-verify cert-name    # Verify certificate

# 6. Network discovery
mndp-scan                            # Scan for MikroTik devices (Linux)
nmap -p 8291,22,80 192.168.88.1     # Scan management ports

# 7. Recovery procedures
# Hardware reset: Hold reset 10+ seconds after power on
# Software reset: /system reset-configuration
# Netinstall: For complete firmware recovery

# 8. Advanced diagnostics
/tool torch interface=ether1         # Monitor traffic
/system logging print               # Check system logs
/system resource monitor            # Check system resources
/ip neighbor print                  # Check ARP entries
```
{% endcode %}

</details>

## Connection best practices

### Security recommendations

1. **Use strong passwords** - Never use default credentials in production
2. **Enable SSH keys** - Disable password authentication when possible  
3. **Restrict access** - Limit management to trusted networks
4. **Use HTTPS** - Enable SSL certificates for web access
5. **Monitor access** - Log and review connection attempts

### Performance optimization

1. **Choose appropriate method** - WinBox for GUI, SSH for scripting
2. **Use local connections** - Direct connection when possible
3. **Optimize SSH settings** - Configure proper ciphers and timeouts
4. **Limit concurrent sessions** - Avoid too many simultaneous connections
5. **Use compression** - Enable SSH compression for slow links

### Operational procedures

1. **Document access methods** - Maintain inventory of connection details
2. **Test backup access** - Ensure alternative connection methods work
3. **Plan for recovery** - Know reset procedures and recovery options
4. **Automate routine tasks** - Use scripts for repetitive operations
5. **Monitor connections** - Watch for unauthorized access attempts
