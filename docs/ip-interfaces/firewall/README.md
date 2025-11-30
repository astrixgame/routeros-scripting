---
description: This section covers RouterOS firewall configuration, including packet filtering, NAT, mangle rules, and advanced security features.
icon: shield
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

# Firewall

{% hint style="info" %}
RouterOS firewall is a powerful packet filtering and manipulation system that provides stateful packet inspection, NAT, traffic shaping, and advanced security features for network protection and traffic control.
{% endhint %}

In WinBox you can configure firewall in **IP -> Firewall**, or you can use terminal with command `/ip firewall`

The RouterOS firewall operates on multiple chains and provides comprehensive control over network traffic through filtering, NAT, and mangle rules.

***

## Firewall fundamentals

### How RouterOS firewall works

**Core components:**
- **Filter rules** - Accept, drop, or reject packets based on criteria
- **NAT rules** - Network Address Translation for routing and security
- **Mangle rules** - Mark packets for QoS and advanced routing
- **Connection tracking** - Stateful inspection of network connections
- **Address lists** - Dynamic IP address groupings for rules

**Processing order:**
1. **Raw** - Before connection tracking (advanced users)
2. **Mangle** - Packet marking and modification 
3. **NAT** - Address translation
4. **Filter** - Accept/drop decisions
5. **Connection tracking** - State management throughout

***

## Basic firewall concepts

### Chains and processing

RouterOS uses predefined chains for packet processing:

**Filter chains:**
- **input** - Packets destined for the router itself
- **forward** - Packets routed through the router
- **output** - Packets originating from the router

**NAT chains:**
- **srcnat** - Source NAT (typically for internet access)
- **dstnat** - Destination NAT (port forwarding, load balancing)

**Mangle chains:**
- **prerouting** - Before routing decision
- **input** - To router (before input filter)
- **forward** - Through router (before forward filter)
- **output** - From router (before output filter)  
- **postrouting** - After routing decision

### Connection states

{% code overflow="wrap" %}
```bash
# Connection states for stateful filtering
new         # First packet of new connection
established # Packets of established connections
related     # Packets related to established connections (FTP data, ICMP errors)
invalid     # Packets that don't belong to any connection
untracked   # Packets not tracked by connection tracking
```
{% endcode %}

***

## Basic firewall configuration

### Default secure firewall setup

Essential firewall rules for basic security:

{% code overflow="wrap" %}
```bash
# Allow established and related connections
/ip firewall filter add chain=input action=accept connection-state=established,related comment="Accept established/related"

# Allow loopback traffic
/ip firewall filter add chain=input action=accept in-interface=lo comment="Accept loopback"

# Allow ICMP (ping)
/ip firewall filter add chain=input action=accept protocol=icmp comment="Accept ICMP"

# Allow SSH from LAN
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=22 src-address-list=LAN comment="Accept SSH from LAN"

# Allow WinBox from LAN  
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=8291 src-address-list=LAN comment="Accept WinBox from LAN"

# Allow web management from LAN
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=80 src-address-list=LAN comment="Accept HTTP from LAN"

# Drop everything else to router
/ip firewall filter add chain=input action=drop comment="Drop all other input"
```
{% endcode %}

### Create address lists

Define network groups for easier rule management:

{% code overflow="wrap" %}
```bash
# Create LAN address list
/ip firewall address-list add list=LAN address=192.168.1.0/24 comment="Local network"
/ip firewall address-list add list=LAN address=10.0.0.0/8 comment="RFC1918 - Class A"
/ip firewall address-list add list=LAN address=172.16.0.0/12 comment="RFC1918 - Class B"

# Create WAN interface list
/interface list add name=WAN comment="Internet-facing interfaces"
/interface list member add list=WAN interface=ether1 comment="Main internet connection"

# Create LAN interface list
/interface list add name=LAN comment="Internal network interfaces"
/interface list member add list=LAN interface=bridge comment="LAN bridge"
```
{% endcode %}

***

## Advanced filtering rules

### Protect against common attacks

{% code overflow="wrap" %}
```bash
# Drop invalid connections
/ip firewall filter add chain=input action=drop connection-state=invalid comment="Drop invalid connections"

# Prevent brute force attacks
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 \
    connection-state=new src-address-list=ssh_blacklist address-list-timeout=1d comment="SSH brute force protection"

/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 \
    connection-state=new src-address-list=ssh_stage3 address-list-timeout=1m comment="SSH stage 3"

/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 \
    connection-state=new src-address-list=ssh_stage2 address-list-timeout=1m src-address-list=ssh_stage3 comment="SSH stage 2"

/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=22 \
    connection-state=new address-list-timeout=1m src-address-list=ssh_stage2 address-list=ssh_blacklist comment="SSH blacklist"

/ip firewall filter add chain=input action=drop src-address-list=ssh_blacklist comment="Drop SSH blacklisted IPs"
```
{% endcode %}

### Port knocking for enhanced security

{% code overflow="wrap" %}
```bash
# Port knocking sequence: 1001 -> 1002 -> 1003 -> SSH
/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=1001 \
    src-address-list=!knock_stage3 address-list=knock_stage1 address-list-timeout=10s comment="Knock stage 1"

/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=1002 \
    src-address-list=knock_stage1 address-list=knock_stage2 address-list-timeout=10s comment="Knock stage 2"

/ip firewall filter add chain=input action=add-src-to-address-list protocol=tcp dst-port=1003 \
    src-address-list=knock_stage2 address-list=knock_stage3 address-list-timeout=10s comment="Knock stage 3"

# Allow SSH only after successful knock
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=22 src-address-list=knock_stage3 comment="Accept SSH after knock"
```
{% endcode %}

***

## Connection tracking and optimization

### Connection tracking settings

{% code overflow="wrap" %}
```bash
# View connection tracking table
/ip firewall connection print

# Adjust connection tracking settings
/ip settings set tcp-syncookies=yes
/ip settings set rp-filter=strict

# Connection limits per source
/ip firewall filter add chain=forward action=drop connection-limit=100,32 protocol=tcp comment="Limit connections per /32"

# Rate limiting
/ip firewall filter add chain=input action=drop src-address-list=rate_limit comment="Drop rate limited IPs"
/ip firewall filter add chain=input action=add-src-to-address-list protocol=icmp \
    address-list=rate_limit address-list-timeout=1m limit=5,5:packet comment="Rate limit ICMP"
```
{% endcode %}

### FastTrack for performance

Enable FastTrack to bypass firewall for established connections:

{% code overflow="wrap" %}
```bash
# Enable FastTrack for better performance (place early in filter rules)
/ip firewall filter add chain=forward action=fasttrack-connection connection-state=established,related \
    comment="FastTrack established/related"

/ip firewall filter add chain=forward action=accept connection-state=established,related \
    comment="Accept established/related (non-FastTrack)"
```
{% endcode %}

***

## Service protection

### Secure management services

{% code overflow="wrap" %}
```bash
# Disable unnecessary services
/ip service disable telnet,ftp,www,api,api-ssl

# Secure SSH
/ip service set ssh port=2222 address=192.168.1.0/24

# Secure WinBox  
/ip service set winbox port=8291 address=192.168.1.0/24

# Secure web interface
/ip service set www-ssl port=8443 address=192.168.1.0/24 certificate=https-cert

# API security
/ip service set api-ssl port=8729 address=192.168.1.0/24 certificate=api-cert
```
{% endcode %}

### DNS and NTP security

{% code overflow="wrap" %}
```bash
# Allow DNS queries from LAN only
/ip firewall filter add chain=input action=accept protocol=udp dst-port=53 src-address-list=LAN comment="Accept DNS from LAN"
/ip firewall filter add chain=input action=drop protocol=udp dst-port=53 comment="Drop external DNS queries"

# Allow NTP from LAN
/ip firewall filter add chain=input action=accept protocol=udp dst-port=123 src-address-list=LAN comment="Accept NTP from LAN"

# Secure SNMP (if needed)
/ip firewall filter add chain=input action=accept protocol=udp dst-port=161 src-address-list=SNMP-MGMT comment="Accept SNMP from management"
```
{% endcode %}

***

## IPv6 firewall basics

### IPv6 firewall configuration

{% code overflow="wrap" %}
```bash
# Accept established and related
/ipv6 firewall filter add chain=input action=accept connection-state=established,related comment="Accept established/related"

# Accept ICMPv6 (essential for IPv6)
/ipv6 firewall filter add chain=input action=accept protocol=icmpv6 comment="Accept ICMPv6"

# Accept link-local
/ipv6 firewall filter add chain=input action=accept src-address=fe80::/16 comment="Accept link-local"

# Accept loopback
/ipv6 firewall filter add chain=input action=accept src-address=::1 comment="Accept loopback"

# Drop invalid
/ipv6 firewall filter add chain=input action=drop connection-state=invalid comment="Drop invalid"

# Allow management from LAN
/ipv6 firewall filter add chain=input action=accept src-address=2001:db8::/32 protocol=tcp dst-port=22 comment="SSH from IPv6 LAN"

# Drop all other input
/ipv6 firewall filter add chain=input action=drop comment="Drop all other IPv6 input"
```
{% endcode %}

***

## Monitoring and logging

### Traffic monitoring

{% code overflow="wrap" %}
```bash
# Monitor firewall rules
/ip firewall filter print stats

# Monitor connections
/ip firewall connection print count-only
/ip firewall connection print where protocol=tcp and state=established

# Monitor address lists
/ip firewall address-list print where list=ssh_blacklist

# Connection tracking statistics
/ip firewall connection tracking print
```
{% endcode %}

### Logging configuration

{% code overflow="wrap" %}
```bash
# Enable logging for dropped packets
/ip firewall filter add chain=input action=log log-prefix="INPUT-DROP: " comment="Log dropped input"
/ip firewall filter add chain=input action=drop

# Log specific attacks
/ip firewall filter add chain=input action=log log-prefix="SSH-ATTACK: " protocol=tcp dst-port=22 \
    connection-state=new src-address-list=ssh_blacklist

# Configure log settings
/system logging action set memory memory-lines=1000
/system logging add topics=firewall,info action=memory
/system logging add topics=firewall,warning action=disk
```
{% endcode %}

***

## Troubleshooting firewall

### Debugging connectivity issues

{% code overflow="wrap" %}
```bash
# Temporarily log all traffic to debug
/ip firewall filter add chain=forward action=log log-prefix="FWD-DEBUG: " place-before=0

# Check specific rule matches
/ip firewall filter print stats where comment~"problem-rule"

# Disable all rules temporarily (DANGEROUS - do only locally!)
/ip firewall filter disable [find]

# Enable rules one by one to identify issue
/ip firewall filter enable 0,1,2

# Monitor in real time
/log print follow where topics~"firewall"
```
{% endcode %}

### Performance troubleshooting

{% code overflow="wrap" %}
```bash
# Check connection tracking usage
/ip firewall connection tracking print

# Monitor CPU usage
/system resource print

# Check if FastTrack is working
/ip firewall filter print stats where action=fasttrack-connection

# Disable connection tracking for specific traffic (if needed)
/ip firewall raw add chain=prerouting action=notrack dst-port=80 protocol=tcp comment="NoTrack HTTP"
```
{% endcode %}

***

## Firewall topics

### Detailed configurations

This firewall section covers several specialized topics:

- **[NAT](nat.md)** - Network Address Translation
  - Masquerade for internet sharing
  - Destination NAT for port forwarding  
  - Source NAT for network routing
  - Advanced NAT scenarios and load balancing

- **[Mangle](mangle.md)** - Packet marking and manipulation
  - QoS packet marking for traffic shaping
  - Policy-based routing with packet marks
  - Connection marking for bandwidth management
  - Advanced traffic manipulation

- **[Layer-7](layer-7.md)** - Application protocol filtering
  - Deep packet inspection patterns
  - Protocol-specific blocking and shaping
  - Custom protocol detection
  - Application-aware firewall rules

***

<details>

<summary>Show complete basic firewall setup</summary>

{% code overflow="wrap" %}
```bash
# Complete basic firewall configuration for RouterOS v7+
# 1. Create interface lists
/interface list add name=WAN comment="Internet-facing interfaces"
/interface list add name=LAN comment="Internal network interfaces"
/interface list member add list=WAN interface=ether1
/interface list member add list=LAN interface=bridge

# 2. Create address lists
/ip firewall address-list add list=LAN address=192.168.1.0/24 comment="Local network"
/ip firewall address-list add list=LAN address=10.0.0.0/8 comment="RFC1918 Class A"
/ip firewall address-list add list=LAN address=172.16.0.0/12 comment="RFC1918 Class B"

# 3. Input chain rules (traffic to router)
/ip firewall filter add chain=input action=accept connection-state=established,related comment="Accept established/related"
/ip firewall filter add chain=input action=accept connection-state=untracked comment="Accept untracked"
/ip firewall filter add chain=input action=drop connection-state=invalid comment="Drop invalid"
/ip firewall filter add chain=input action=accept in-interface=lo comment="Accept loopback"
/ip firewall filter add chain=input action=accept protocol=icmp comment="Accept ICMP"
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=22 in-interface-list=LAN comment="SSH from LAN"
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=8291 in-interface-list=LAN comment="WinBox from LAN"
/ip firewall filter add chain=input action=accept protocol=udp dst-port=53 in-interface-list=LAN comment="DNS from LAN"
/ip firewall filter add chain=input action=drop comment="Drop all other input"

# 4. Forward chain rules (traffic through router)
/ip firewall filter add chain=forward action=fasttrack-connection connection-state=established,related comment="FastTrack established/related"
/ip firewall filter add chain=forward action=accept connection-state=established,related comment="Accept established/related"
/ip firewall filter add chain=forward action=drop connection-state=invalid comment="Drop invalid"
/ip firewall filter add chain=forward action=drop connection-nat-state=!dstnat connection-state=new in-interface-list=WAN comment="Drop new connections from WAN not DSTNATed"

# 5. Basic NAT for internet access
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN comment="Masquerade to internet"

# 6. IPv6 firewall (if using IPv6)
/ipv6 firewall filter add chain=input action=accept connection-state=established,related,untracked comment="Accept established/related/untracked"
/ipv6 firewall filter add chain=input action=drop connection-state=invalid comment="Drop invalid"
/ipv6 firewall filter add chain=input action=accept protocol=icmpv6 comment="Accept ICMPv6"
/ipv6 firewall filter add chain=input action=accept src-address=fe80::/16 comment="Accept link-local"
/ipv6 firewall filter add chain=input action=accept src-address=::1 comment="Accept loopback"
/ipv6 firewall filter add chain=input action=drop comment="Drop all other IPv6 input"

/ipv6 firewall filter add chain=forward action=accept connection-state=established,related,untracked comment="Accept established/related/untracked"
/ipv6 firewall filter add chain=forward action=drop connection-state=invalid comment="Drop invalid"
```
{% endcode %}

</details>

## Best practices

### Security recommendations

1. **Default deny policy** - Block everything not explicitly allowed
2. **Least privilege** - Grant minimum necessary access
3. **Regular monitoring** - Watch logs and connection tables
4. **Address list management** - Use dynamic lists for threats
5. **Service hardening** - Disable unnecessary services

### Performance optimization

1. **Use FastTrack** - Bypass firewall for established connections  
2. **Optimize rule order** - Most frequent matches first
3. **Connection limits** - Prevent resource exhaustion
4. **Interface lists** - Use instead of individual interfaces
5. **Regular cleanup** - Remove unused rules and lists

### Maintenance tips

1. **Document rules** - Use meaningful comments
2. **Version control** - Export configurations regularly
3. **Test changes** - Verify in lab environment first
4. **Monitor impact** - Check performance after changes
5. **Emergency access** - Always maintain management access