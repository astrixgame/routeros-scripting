---
description: This section covers the complete RouterOS default configuration setup for new installations and factory resets.
icon: sheet-plastic
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

# Default configuration

{% hint style="info" %}
RouterOS default configuration provides a secure, production-ready setup for most networking scenarios, including firewall protection, DHCP services, and basic connectivity suitable for home offices and small businesses.
{% endhint %}

In WinBox you can apply default configuration in **System -> Reset Configuration**, or you can use terminal with command `/system reset-configuration`

The RouterOS default configuration includes essential services, security features, and network connectivity that works out-of-the-box for most common deployment scenarios.

***

## Understanding default configuration

### What gets configured automatically

**Network setup:**
- **Bridge interface** - All ethernet ports bridged together
- **DHCP server** - Automatic IP assignment for LAN clients  
- **DNS resolver** - Local DNS forwarding and caching
- **Default routes** - Automatic gateway configuration via DHCP client
- **Network interfaces** - All ports enabled and configured

**Security features:**
- **Firewall rules** - Basic input and forward filtering
- **NAT configuration** - Internet sharing via masquerade
- **Service security** - Management services restricted to LAN
- **User accounts** - Admin user with no password (initially)

**Management services:**
- **WinBox access** - Graphical management interface  
- **Web interface** - HTTP-based configuration
- **SSH access** - Secure command line access
- **MAC-Telnet** - Discovery and access protocol

***

## Fresh RouterOS installation

### Initial setup procedure

After flashing RouterOS or factory reset:

{% code overflow="wrap" %}
```bash
# 1. Connect to the device
# Physical: Connect ethernet cable to ether1 (WAN) and ether2+ (LAN)
# Access: Use WinBox, WebFig, or SSH to 192.168.88.1

# 2. Secure the installation immediately
/user set admin password=YourSecurePassword123!

# 3. Configure basic identification
/system identity set name=MyRouter-Office
/system clock set time-zone-name=America/New_York

# 4. Update to latest version
/system package update check-for-updates
/system package update download
# Reboot required after download
/system reboot
```
{% endcode %}

### Default network addressing

The default configuration uses these IP ranges:

{% code overflow="wrap" %}
```bash
# Default LAN configuration
LAN Network: 192.168.88.0/24
Router IP: 192.168.88.1/24
DHCP Pool: 192.168.88.10-192.168.88.254
DNS Server: 192.168.88.1

# WAN configuration (ether1)
DHCP Client: Enabled on ether1
Default Route: Via DHCP gateway

# Check current configuration
/ip address print
/ip dhcp-server print
/ip route print
```
{% endcode %}

***

## Complete default configuration script

### RouterOS v7 default configuration

This is what RouterOS applies during reset-configuration:

{% code overflow="wrap" %}
```bash
# RouterOS Default Configuration Script (v7.x)

# 1. System identification and time
/system clock set time-zone-name=auto
/system identity set name=MikroTik

# 2. Create bridge for LAN ports
/interface bridge add name=bridge disabled=no auto-mac=yes protocol-mode=rstp comment="defconf"

# 3. Add LAN ports to bridge (exclude ether1 for WAN)
/interface bridge port
add bridge=bridge interface=ether2 comment="defconf"
add bridge=bridge interface=ether3 comment="defconf"  
add bridge=bridge interface=ether4 comment="defconf"
add bridge=bridge interface=ether5 comment="defconf"
# Note: ether1 is typically WAN and not bridged

# 4. Create interface lists for firewall
/interface list add name=WAN comment="defconf"
/interface list add name=LAN comment="defconf"
/interface list member add list=WAN interface=ether1 comment="defconf"
/interface list member add list=LAN interface=bridge comment="defconf"

# 5. Configure IP addressing
/ip address add address=192.168.88.1/24 interface=bridge comment="defconf"

# 6. Configure DHCP client on WAN
/ip dhcp-client add interface=ether1 disabled=no add-default-route=yes use-peer-dns=yes comment="defconf"

# 7. Configure DHCP server for LAN
/ip pool add name=default-dhcp ranges=192.168.88.10-192.168.88.254 comment="defconf"
/ip dhcp-server add name=defconf address-pool=default-dhcp interface=bridge disabled=no
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1 comment="defconf"

# 8. Configure DNS
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes

# 9. Basic firewall configuration
# Input chain (to router)
/ip firewall filter
add action=accept chain=input connection-state=established,related,untracked comment="defconf: accept established,related,untracked"
add action=drop chain=input connection-state=invalid comment="defconf: drop invalid"
add action=accept chain=input protocol=icmp comment="defconf: accept ICMP"
add action=accept chain=input dst-address=127.0.0.1 comment="defconf: accept to local loopback (for CAPsMAN)"
add action=drop chain=input in-interface-list=!LAN comment="defconf: drop all not coming from LAN"

# Forward chain (through router)
add action=accept chain=forward connection-state=established,related,untracked comment="defconf: accept in ipsec policy"
add action=drop chain=forward connection-state=invalid comment="defconf: drop invalid"
add action=drop chain=forward connection-nat-state=!dstnat connection-state=new in-interface-list=WAN comment="defconf: drop all from WAN not DSTNATed"

# 10. NAT configuration
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN comment="defconf: masquerade"

# 11. Service configuration
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=no address=192.168.88.0/24
set ssh disabled=no address=192.168.88.0/24
set api disabled=yes
set winbox disabled=no address=192.168.88.0/24
set api-ssl disabled=yes

# 12. SNTP time synchronization
/system ntp client set enabled=yes
/system ntp client servers add address=time.cloudflare.com
```
{% endcode %}

***

## Customizing default configuration

### Secure the default setup

Essential security modifications:

{% code overflow="wrap" %}
```bash
# 1. Set strong admin password
/user set admin password=ComplexPassword123!

# 2. Create additional admin user
/user add name=netadmin group=full password=AnotherSecurePassword456! comment="Secondary admin account"

# 3. Disable unnecessary services
/ip service disable telnet,ftp,api,api-ssl

# 4. Change default management ports (optional)
/ip service set www port=8080 address=192.168.88.0/24
/ip service set ssh port=2222 address=192.168.88.0/24
/ip service set winbox port=8291 address=192.168.88.0/24

# 5. Enable HTTPS for web management
/certificate add name=https-cert common-name=router.local key-size=2048 days-valid=365 key-usage=digital-signature,key-encipherment
/certificate sign https-cert
/ip service set www-ssl disabled=no port=8443 address=192.168.88.0/24 certificate=https-cert

# 6. Strengthen firewall (add after existing rules)
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh_blacklist address-list-timeout=1d comment="SSH brute force protection"
```
{% endcode %}

### Customize network addressing

Change default IP scheme to match your requirements:

{% code overflow="wrap" %}
```bash
# Change to 192.168.1.0/24 network
# 1. Remove default configuration
/ip dhcp-server remove [find name=defconf]
/ip dhcp-server network remove [find address=192.168.88.0/24]
/ip pool remove [find name=default-dhcp]
/ip address remove [find address=192.168.88.1/24]

# 2. Configure new addressing
/ip address add address=192.168.1.1/24 interface=bridge comment="LAN network"
/ip pool add name=lan-dhcp ranges=192.168.1.100-192.168.1.200 comment="LAN DHCP pool"
/ip dhcp-server add name=lan-dhcp address-pool=lan-dhcp interface=bridge disabled=no
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=192.168.1.1 comment="LAN DHCP network"

# 3. Update service addresses
/ip service set www address=192.168.1.0/24
/ip service set ssh address=192.168.1.0/24  
/ip service set winbox address=192.168.1.0/24
/ip service set www-ssl address=192.168.1.0/24

# 4. Update firewall rules if needed
/ip firewall filter set [find comment~"defconf"] in-interface-list=LAN
```
{% endcode %}

***

## WiFi configuration (if supported)

### Basic WiFi setup on devices with wireless

{% code overflow="wrap" %}
```bash
# 1. Configure WiFi security profile
/interface wireless security-profiles add name=wifi-security mode=wpa2 authentication-types=wpa2-psk \
    wpa2-pre-shared-key=YourWiFiPassword123! comment="Main WiFi security"

# 2. Configure wireless interface
/interface wireless set wlan1 disabled=no mode=ap-bridge ssid=MyRouter-WiFi \
    security-profile=wifi-security channel=auto wireless-protocol=802.11 country=united-states

# 3. Add WiFi to LAN bridge
/interface bridge port add bridge=bridge interface=wlan1 comment="WiFi to LAN bridge"

# 4. Add WiFi to LAN interface list
/interface list member add list=LAN interface=wlan1 comment="WiFi in LAN list"
```
{% endcode %}

### Guest WiFi network (optional)

{% code overflow="wrap" %}
```bash
# 1. Create guest WiFi security profile
/interface wireless security-profiles add name=guest-wifi mode=wpa2 authentication-types=wpa2-psk \
    wpa2-pre-shared-key=GuestPassword123! comment="Guest WiFi security"

# 2. Create virtual access point for guests
/interface wireless set wlan1 wds-mode=dynamic-mesh wds-default-bridge=none
/interface wireless add name=wlan1-guest master-interface=wlan1 disabled=no ssid=MyRouter-Guest \
    security-profile=guest-wifi comment="Guest WiFi interface"

# 3. Create separate bridge for guests
/interface bridge add name=bridge-guest disabled=no comment="Guest network bridge"
/interface bridge port add bridge=bridge-guest interface=wlan1-guest

# 4. Configure guest network addressing  
/ip address add address=192.168.99.1/24 interface=bridge-guest comment="Guest network"
/ip pool add name=guest-dhcp ranges=192.168.99.10-192.168.99.100
/ip dhcp-server add name=guest-dhcp address-pool=guest-dhcp interface=bridge-guest disabled=no
/ip dhcp-server network add address=192.168.99.0/24 gateway=192.168.99.1 dns-server=192.168.99.1

# 5. Add guest interface to WAN list for internet-only access
/interface list member add list=WAN interface=bridge-guest comment="Guest to internet only"
```
{% endcode %}

***

## Default configuration variants

### Home office configuration

Optimized for home/small office use:

{% code overflow="wrap" %}
```bash
# Apply default first, then customize
/system reset-configuration default-no-dhcp-client=yes

# Configure static WAN IP (if available)
/ip address add address=203.0.113.10/24 interface=ether1 comment="Static WAN IP"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 comment="Static default route"
/ip dns set servers=1.1.1.1,8.8.8.8

# Enable UPnP for gaming/media devices
/ip upnp set enabled=yes allow-disable-external-interface=no
/ip upnp interfaces add interface=bridge type=internal
/ip upnp interfaces add interface=ether1 type=external

# Add port forwarding for common services
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    to-addresses=192.168.88.10 to-ports=80 comment="HTTP server"
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=22 \
    to-addresses=192.168.88.5 to-ports=22 comment="SSH server"
```
{% endcode %}

### Branch office configuration

For connecting remote office to headquarters:

{% code overflow="wrap" %}
```bash
# Apply default configuration
/system reset-configuration

# Configure site identification
/system identity set name=BranchOffice-Location
/system clock set time-zone-name=America/New_York

# Change to corporate IP scheme
/ip address set [find interface=bridge] address=192.168.10.1/24
/ip dhcp-server network set [find] address=192.168.10.0/24 gateway=192.168.10.1 dns-server=192.168.10.1
/ip pool set [find] ranges=192.168.10.100-192.168.10.200

# Configure VPN to headquarters (example with IPSec)
/ip ipsec profile add name=hq-profile enc-algorithm=aes-256 auth-algorithm=sha256
/ip ipsec peer add name=hq-peer address=203.0.113.100 profile=hq-profile
/ip ipsec identity add peer=hq-peer secret=SharedSecretKey my-id=address:203.0.113.10 remote-id=address:203.0.113.100
/ip ipsec policy add src-address=192.168.10.0/24 dst-address=192.168.1.0/24 action=encrypt peer=hq-peer

# Add route to headquarters network
/ip route add dst-address=192.168.1.0/24 gateway=hq-peer comment="Route to HQ"

# Update DNS to use HQ DNS servers
/ip dns set servers=192.168.1.10,192.168.1.11
```
{% endcode %}

***

## Monitoring default configuration

### Verify configuration status

{% code overflow="wrap" %}
```bash
# Check network connectivity
/ping 8.8.8.8
/ping google.com

# Verify DHCP server operation  
/ip dhcp-server lease print
/ip dhcp-server binding print

# Check firewall rules
/ip firewall filter print where comment~"defconf"
/ip firewall nat print where comment~"defconf"

# Verify services
/ip service print
/interface print brief

# Check system status
/system health print
/system resource print
/system routerboard print
```
{% endcode %}

### Monitor system logs

{% code overflow="wrap" %}
```bash
# View recent system messages
/log print

# Monitor DHCP activity
/log print where topics~"dhcp"

# Monitor firewall activity
/log print where topics~"firewall"

# Check system startup logs
/log print where topics~"system,info"

# Monitor interface changes
/log print where topics~"interface,info"
```
{% endcode %}

***

## Backup and restore procedures

### Export default configuration

{% code overflow="wrap" %}
```bash
# Export complete configuration
/export file=default-config-backup

# Export specific sections
/ip firewall export file=firewall-backup
/interface export file=interfaces-backup
/ip dhcp-server export file=dhcp-backup

# Create system backup (binary format)
/system backup save name=default-system-backup

# View exported files
/file print where type=file
```
{% endcode %}

### Restore configuration

{% code overflow="wrap" %}
```bash
# Import configuration file
/import default-config-backup.rsc

# Restore from system backup
/system backup load name=default-system-backup.backup

# Reset to factory defaults
/system reset-configuration keep-users=no no-defaults=no skip-backup=yes

# Reset with custom defaults
/system reset-configuration default-no-dhcp-client=yes keep-users=yes
```
{% endcode %}

***

## Troubleshooting default configuration

### Common issues and solutions

{% code overflow="wrap" %}
```bash
# Issue: Can't access internet
# Check WAN connectivity
/ping 8.8.8.8 interface=ether1
/ip route print where dst-address=0.0.0.0/0
/ip dhcp-client print

# Issue: Can't access router from LAN
/interface print where disabled=no
/ip address print where interface=bridge
/ip service print where disabled=no

# Issue: DHCP not working
/ip dhcp-server print detail
/ip dhcp-server lease print
/log print where topics~"dhcp"

# Issue: Firewall blocking traffic
/ip firewall filter print stats
/log print where topics~"firewall"
```
{% endcode %}

### Performance verification

{% code overflow="wrap" %}
```bash
# Test internet speed
/tool speed-test address=speedtest.net duration=30

# Check interface utilization
/interface monitor-traffic interface=ether1 duration=10
/interface monitor-traffic interface=bridge duration=10

# Monitor system resources
/system resource monitor numbers=0 duration=30

# Check memory usage
/system resource print
```
{% endcode %}

***

<details>

<summary>Show complete secure default configuration</summary>

{% code overflow="wrap" %}
```bash
# Complete secure default configuration for RouterOS v7+
# Apply this after factory reset for production-ready setup

# 1. System identification and security
/system identity set name=SecureRouter-Office
/system clock set time-zone-name=America/New_York
/user set admin password=SecureAdminPassword123!
/user add name=backup-admin group=full password=BackupAdminPassword456! comment="Backup admin account"

# 2. Network configuration (using 192.168.1.0/24)
/interface bridge add name=bridge disabled=no auto-mac=yes protocol-mode=rstp comment="LAN Bridge"
/interface bridge port add bridge=bridge interface=ether2,ether3,ether4,ether5 comment="LAN Ports"

/interface list add name=WAN comment="Internet interfaces"
/interface list add name=LAN comment="Local network interfaces"
/interface list member add list=WAN interface=ether1
/interface list member add list=LAN interface=bridge

/ip address add address=192.168.1.1/24 interface=bridge comment="LAN IP address"

# 3. DHCP configuration
/ip pool add name=lan-dhcp ranges=192.168.1.100-192.168.1.200 comment="LAN DHCP Pool"
/ip dhcp-server add name=lan-dhcp address-pool=lan-dhcp interface=bridge lease-time=24h disabled=no
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 dns-server=192.168.1.1,1.1.1.1 comment="LAN DHCP Network"

# 4. WAN configuration
/ip dhcp-client add interface=ether1 disabled=no add-default-route=yes use-peer-dns=yes comment="WAN DHCP Client"

# 5. DNS configuration
/ip dns set servers=1.1.1.1,8.8.8.8 allow-remote-requests=yes cache-size=2048KiB

# 6. Comprehensive firewall configuration
# Input chain (traffic to router)
/ip firewall filter add chain=input action=accept connection-state=established,related,untracked comment="Accept established/related"
/ip firewall filter add chain=input action=drop connection-state=invalid comment="Drop invalid connections"
/ip firewall filter add chain=input action=accept protocol=icmp limit=5,5:packet comment="Accept ICMP (limited)"
/ip firewall filter add chain=input action=accept dst-address=127.0.0.1 comment="Accept loopback"
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=22 in-interface-list=LAN comment="SSH from LAN"
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=8080 in-interface-list=LAN comment="HTTP from LAN"
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=8291 in-interface-list=LAN comment="WinBox from LAN"
/ip firewall filter add chain=input action=accept protocol=udp dst-port=53 in-interface-list=LAN comment="DNS from LAN"

# SSH brute force protection
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh_blacklist address-list-timeout=1d comment="SSH brute force - final block"
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh_stage3 address-list-timeout=1m comment="SSH brute force - stage 3"
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh_stage2 address-list-timeout=1m src-address-list=ssh_stage3 address-list=ssh_stage2 comment="SSH brute force - stage 2"
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 connection-state=new \
    address-list-timeout=1m src-address-list=ssh_stage2 address-list=ssh_blacklist comment="SSH brute force - add to blacklist"

/ip firewall filter add chain=input action=drop comment="Drop all other input"

# Forward chain (traffic through router)
/ip firewall filter add chain=forward action=fasttrack-connection connection-state=established,related comment="FastTrack established connections"
/ip firewall filter add chain=forward action=accept connection-state=established,related,untracked comment="Accept established/related/untracked"
/ip firewall filter add chain=forward action=drop connection-state=invalid comment="Drop invalid connections"
/ip firewall filter add chain=forward action=drop connection-nat-state=!dstnat connection-state=new \
    in-interface-list=WAN comment="Drop new connections from WAN not DSTNATed"

# 7. NAT configuration
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN comment="Masquerade to internet"

# 8. Service security
/ip service disable telnet,ftp,api,api-ssl
/ip service set www disabled=no port=8080 address=192.168.1.0/24
/ip service set ssh disabled=no port=2222 address=192.168.1.0/24
/ip service set winbox disabled=no port=8291 address=192.168.1.0/24

# Create HTTPS certificate and enable secure web
/certificate add name=router-cert common-name=router.local days-valid=3650 key-size=2048 \
    key-usage=digital-signature,key-encipherment
/certificate sign router-cert ca-crl-host=192.168.1.1
/ip service set www-ssl disabled=no port=8443 address=192.168.1.0/24 certificate=router-cert

# 9. Time synchronization  
/system ntp client set enabled=yes
/system ntp client servers add address=pool.ntp.org
/system ntp client servers add address=time.cloudflare.com

# 10. Logging configuration
/system logging add topics=firewall,warning action=memory
/system logging add topics=system,error,critical action=memory
/system logging action set memory memory-lines=2000

# 11. Monitor critical connectivity
/tool netwatch add host=1.1.1.1 timeout=2s interval=30s comment="Monitor internet connectivity"

# 12. SNMP security (disable by default)
/snmp set enabled=no

# This configuration provides a secure, monitored, production-ready RouterOS setup
```
{% endcode %}

</details>

## Default configuration best practices

### Security recommendations

1. **Change default password immediately** - Never leave admin account without password
2. **Disable unnecessary services** - Only enable services you actually need
3. **Use strong firewall rules** - Implement comprehensive filtering
4. **Enable logging** - Monitor system activity and security events
5. **Regular updates** - Keep RouterOS version current

### Network design principles

1. **Plan IP addressing** - Use consistent, documented IP schemes
2. **Implement proper segmentation** - Separate guest, management, and production networks
3. **Configure redundancy** - Plan for backup connectivity when possible
4. **Monitor performance** - Establish baseline metrics
5. **Document configuration** - Keep records of all customizations

### Maintenance procedures

1. **Regular backups** - Export configurations and create system backups
2. **Change monitoring** - Track all configuration modifications
3. **Performance testing** - Verify throughput and latency regularly
4. **Security audits** - Review firewall rules and access controls
5. **Capacity planning** - Monitor resource usage and plan for growth

