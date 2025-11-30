---
description: This section covers Network Address Translation (NAT) configuration for internet sharing, port forwarding, and advanced routing scenarios.
icon: router
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

# NAT

{% hint style="info" %}
Network Address Translation (NAT) allows RouterOS to modify network address information in packet headers while in transit, enabling internet sharing, port forwarding, load balancing, and advanced routing scenarios.
{% endhint %}

In WinBox you can configure NAT in **IP -> Firewall -> NAT**, or you can use terminal with command `/ip firewall nat`

NAT is essential for sharing internet connections and providing services through port forwarding while maintaining network security.

***

## NAT fundamentals

### How NAT works

**NAT types:**
- **Source NAT (SRCNAT)** - Modifies source addresses (typically for internet access)
- **Destination NAT (DSTNAT)** - Modifies destination addresses (port forwarding, load balancing)
- **Masquerade** - Special SRCNAT for dynamic IP addresses
- **Redirect** - Redirects packets to local services

**NAT processing:**
1. **DSTNAT chain** - Processed before routing decision
2. **Routing** - Determine packet destination
3. **SRCNAT chain** - Processed after routing decision
4. **Connection tracking** - Maintains connection state for return traffic

### Connection tracking states

{% code overflow="wrap" %}
```bash
# Connection NAT states
srcnat    # Connection uses source NAT
dstnat    # Connection uses destination NAT
```
{% endcode %}

***

## Basic internet sharing (Masquerade)

### Simple masquerade setup

Most common NAT configuration for sharing internet:

{% code overflow="wrap" %}
```bash
# Basic masquerade for internet sharing
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1 comment="Masquerade to internet"

# Using interface list (recommended)
/interface list add name=WAN comment="Internet-facing interfaces"
/interface list member add list=WAN interface=ether1
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN comment="Masquerade to WAN"
```
{% endcode %}

### Masquerade with specific networks

Control which networks can access internet:

{% code overflow="wrap" %}
```bash
# Allow only LAN networks to access internet
/ip firewall address-list add list=LAN address=192.168.1.0/24
/ip firewall address-list add list=LAN address=10.0.0.0/8

/ip firewall nat add chain=srcnat action=masquerade src-address-list=LAN out-interface-list=WAN comment="Masquerade LAN to internet"
```
{% endcode %}

***

## Port forwarding (DSTNAT)

### Basic port forwarding

Forward external ports to internal services:

{% code overflow="wrap" %}
```bash
# Forward HTTP (port 80) to internal web server
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    in-interface-list=WAN to-addresses=192.168.1.100 to-ports=80 comment="HTTP to web server"

# Forward HTTPS (port 443) to internal web server  
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=443 \
    in-interface-list=WAN to-addresses=192.168.1.100 to-ports=443 comment="HTTPS to web server"

# Forward SSH to internal server (different port)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=2222 \
    in-interface-list=WAN to-addresses=192.168.1.10 to-ports=22 comment="SSH to server"
```
{% endcode %}

### Port forwarding with specific source

Limit port forwarding to specific source addresses:

{% code overflow="wrap" %}
```bash
# Allow RDP only from specific management network
/ip firewall address-list add list=MGMT address=203.0.113.0/24 comment="Management network"

/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=3389 \
    src-address-list=MGMT in-interface-list=WAN \
    to-addresses=192.168.1.50 to-ports=3389 comment="RDP from management only"
```
{% endcode %}

### Range port forwarding

Forward port ranges for applications:

{% code overflow="wrap" %}
```bash
# Forward FTP data ports (passive mode)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=21000-21010 \
    in-interface-list=WAN to-addresses=192.168.1.200 to-ports=21000-21010 comment="FTP passive data ports"

# Forward game server port range
/ip firewall nat add chain=dstnat action=dst-nat protocol=udp dst-port=7777-7784 \
    in-interface-list=WAN to-addresses=192.168.1.150 to-ports=7777-7784 comment="Game server ports"
```
{% endcode %}

***

## Advanced NAT scenarios

### Load balancing with NAT

Distribute connections across multiple servers:

{% code overflow="wrap" %}
```bash
# Create address list for web servers
/ip firewall address-list add list=WEB-SERVERS address=192.168.1.100
/ip firewall address-list add list=WEB-SERVERS address=192.168.1.101
/ip firewall address-list add list=WEB-SERVERS address=192.168.1.102

# Load balance HTTP requests using nth packet
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    nth=3,1 in-interface-list=WAN to-addresses=192.168.1.100 comment="HTTP LB server 1"

/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    nth=3,2 in-interface-list=WAN to-addresses=192.168.1.101 comment="HTTP LB server 2"

/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    nth=3,0 in-interface-list=WAN to-addresses=192.168.1.102 comment="HTTP LB server 3"
```
{% endcode %}

### Conditional NAT with packet marks

Use mangle marks for conditional NAT:

{% code overflow="wrap" %}
```bash
# Mark packets from specific network
/ip firewall mangle add chain=prerouting action=mark-packet src-address=192.168.2.0/24 \
    new-packet-mark=guest-network passthrough=yes comment="Mark guest traffic"

# Use different exit interface for marked packets
/ip firewall nat add chain=srcnat action=masquerade packet-mark=guest-network \
    out-interface=ether2 comment="Guest network via secondary WAN"
```
{% endcode %}

### Hair-pin NAT (NAT loopback)

Allow internal access to services using external IP:

{% code overflow="wrap" %}
```bash
# External to internal (normal DSTNAT)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    in-interface-list=WAN to-addresses=192.168.1.100 comment="HTTP from internet"

# Internal hair-pin NAT (access using external IP from LAN)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 \
    dst-address=203.0.113.10 src-address=192.168.1.0/24 \
    to-addresses=192.168.1.100 comment="HTTP hair-pin NAT"

/ip firewall nat add chain=srcnat action=masquerade protocol=tcp dst-port=80 \
    dst-address=192.168.1.100 src-address=192.168.1.0/24 comment="HTTP hair-pin SRCNAT"
```
{% endcode %}

***

## Multi-WAN NAT scenarios

### Dual WAN with policy routing

NAT configuration for multiple internet connections:

{% code overflow="wrap" %}
```bash
# Create WAN interface lists
/interface list add name=WAN1
/interface list add name=WAN2
/interface list member add list=WAN1 interface=ether1
/interface list member add list=WAN2 interface=ether2

# Mark connections for specific WANs
/ip firewall mangle add chain=input action=mark-connection in-interface-list=WAN1 \
    new-connection-mark=wan1-conn passthrough=yes comment="Mark WAN1 connections"

/ip firewall mangle add chain=input action=mark-connection in-interface-list=WAN2 \
    new-connection-mark=wan2-conn passthrough=yes comment="Mark WAN2 connections"

# Route reply packets back through same interface
/ip firewall mangle add chain=output action=mark-routing connection-mark=wan1-conn \
    new-routing-mark=wan1-route passthrough=yes comment="Route WAN1 replies"

/ip firewall mangle add chain=output action=mark-routing connection-mark=wan2-conn \
    new-routing-mark=wan2-route passthrough=yes comment="Route WAN2 replies"

# NAT rules for each WAN
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN1 comment="Masquerade WAN1"
/ip firewall nat add chain=srcnat action=masquerade out-interface-list=WAN2 comment="Masquerade WAN2"

# Routing tables for marked traffic
/ip route add gateway=203.0.113.1 routing-mark=wan1-route comment="WAN1 gateway"
/ip route add gateway=203.0.114.1 routing-mark=wan2-route comment="WAN2 gateway"
```
{% endcode %}

### Failover NAT configuration

Primary/backup WAN setup with NAT:

{% code overflow="wrap" %}
```bash
# Primary WAN NAT
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1 comment="Primary WAN masquerade"

# Backup WAN NAT (higher distance/lower priority)
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether2 comment="Backup WAN masquerade" disabled=yes

# Script to enable backup NAT when primary fails (simplified)
# /system script add name=wan-failover source={
#   if ([/ping 8.8.8.8 interface=ether1 count=3] = 0) {
#     /ip firewall nat enable [find comment="Backup WAN masquerade"]
#   } else {
#     /ip firewall nat disable [find comment="Backup WAN masquerade"]  
#   }
# }
```
{% endcode %}

***

## NAT for VPN and tunnels

### NAT with OpenVPN

Configure NAT for OpenVPN clients:

{% code overflow="wrap" %}
```bash
# Allow OpenVPN clients to access internet
/ip firewall nat add chain=srcnat action=masquerade src-address=10.8.0.0/24 \
    out-interface-list=WAN comment="OpenVPN clients to internet"

# Allow OpenVPN clients to access LAN
/ip firewall nat add chain=srcnat action=masquerade src-address=10.8.0.0/24 \
    dst-address=192.168.1.0/24 out-interface=bridge comment="OpenVPN to LAN"
```
{% endcode %}

### NAT with site-to-site VPN

Configure NAT for site-to-site connections:

{% code overflow="wrap" %}
```bash
# Site A: NAT local network to site B
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.1.0/24 \
    dst-address=192.168.2.0/24 to-addresses=10.255.0.1 out-interface=ipsec-site-b \
    comment="Site A to Site B NAT"

# Site B: NAT local network to site A  
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.2.0/24 \
    dst-address=192.168.1.0/24 to-addresses=10.255.0.2 out-interface=ipsec-site-a \
    comment="Site B to Site A NAT"
```
{% endcode %}

***

## Source NAT (SRCNAT) advanced

### Source NAT to specific addresses

Use specific source addresses instead of masquerade:

{% code overflow="wrap" %}
```bash
# Use specific public IP for source NAT
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.1.0/24 \
    out-interface-list=WAN to-addresses=203.0.113.10 comment="SRCNAT to specific IP"

# Use different source IPs for different internal networks
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.1.0/24 \
    out-interface-list=WAN to-addresses=203.0.113.10 comment="LAN to public IP 1"

/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.2.0/24 \
    out-interface-list=WAN to-addresses=203.0.113.11 comment="DMZ to public IP 2"
```
{% endcode %}

### Policy-based source NAT

Different source NAT based on destination:

{% code overflow="wrap" %}
```bash
# Different source IP for specific destinations
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.1.0/24 \
    dst-address=8.8.8.8 out-interface-list=WAN to-addresses=203.0.113.10 \
    comment="Google DNS via IP1"

/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.1.0/24 \
    dst-port=25 protocol=tcp out-interface-list=WAN to-addresses=203.0.113.11 \
    comment="SMTP via IP2"
```
{% endcode %}

***

## Redirect and local NAT

### Redirect to local services

Redirect traffic to router services:

{% code overflow="wrap" %}
```bash
# Redirect all DNS queries to local DNS
/ip firewall nat add chain=dstnat action=redirect protocol=udp dst-port=53 \
    to-ports=53 comment="Redirect DNS to router"

# Redirect HTTP to HTTPS
/ip firewall nat add chain=dstnat action=redirect protocol=tcp dst-port=80 \
    to-ports=443 comment="Redirect HTTP to HTTPS"

# Transparent proxy redirect
/ip firewall nat add chain=dstnat action=redirect protocol=tcp dst-port=80,8080 \
    to-ports=3128 comment="Redirect to proxy"
```
{% endcode %}

### Captive portal NAT

NAT rules for captive portal implementations:

{% code overflow="wrap" %}
```bash
# Redirect unauthorized users to captive portal
/ip firewall nat add chain=dstnat action=redirect src-address-list=!authorized \
    protocol=tcp dst-port=80 to-ports=8080 comment="Captive portal redirect"

# Allow authorized users normal internet access
/ip firewall nat add chain=srcnat action=masquerade src-address-list=authorized \
    out-interface-list=WAN comment="Authorized users to internet"
```
{% endcode %}

***

## Monitoring and troubleshooting NAT

### Monitor NAT rules

{% code overflow="wrap" %}
```bash
# Monitor NAT rule statistics
/ip firewall nat print stats

# Monitor active connections with NAT
/ip firewall connection print where nat=yes

# Check specific NAT connections
/ip firewall connection print where dst-address~"192.168.1.100"

# Monitor connection tracking
/ip firewall connection tracking print
```
{% endcode %}

### Debug NAT issues

{% code overflow="wrap" %}
```bash
# Enable NAT logging for debugging
/ip firewall nat add chain=dstnat action=log log-prefix="DSTNAT: " place-before=0
/ip firewall nat add chain=srcnat action=log log-prefix="SRCNAT: " place-before=0

# Test port forwarding
/tool netwatch add host=192.168.1.100 timeout=1s

# Check if packets reach destination
/tool sniffer start interface=bridge filter-ip-address=192.168.1.100

# Verify routing for NAT traffic
/ip route print where dst-address~"192.168.1.100"
```
{% endcode %}

### Performance monitoring

{% code overflow="wrap" %}
```bash
# Monitor NAT performance
/interface monitor-traffic interface=ether1

# Check connection table usage
/ip settings print

# Monitor CPU usage with NAT
/system resource print

# Check FastTrack effectiveness
/ip firewall filter print stats where action=fasttrack-connection
```
{% endcode %}

***

## NAT best practices

### Performance optimization

{% code overflow="wrap" %}
```bash
# Use FastTrack to bypass NAT for established connections
/ip firewall filter add chain=forward action=fasttrack-connection \
    connection-state=established,related comment="FastTrack established connections"

# Optimize connection tracking
/ip settings set tcp-syncookies=yes
/ip settings set max-fresh-connections=900

# Use interface lists instead of specific interfaces
/interface list add name=WAN
/interface list member add list=WAN interface=ether1
```
{% endcode %}

### Security considerations

{% code overflow="wrap" %}
```bash
# Limit port forwarding sources
/ip firewall address-list add list=ALLOWED-SOURCES address=203.0.113.0/24

/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=22 \
    src-address-list=ALLOWED-SOURCES in-interface-list=WAN \
    to-addresses=192.168.1.10 to-ports=22 comment="Secure SSH forwarding"

# Prevent NAT bypass
/ip firewall filter add chain=forward action=drop connection-nat-state=!dstnat \
    connection-state=new in-interface-list=WAN comment="Drop new WAN connections not DSTNATed"
```
{% endcode %}

***

<details>

<summary>Show complete NAT setup for small office</summary>

{% code overflow="wrap" %}
```bash
# Complete NAT configuration for small office with DMZ and services
# 1. Create interface and address lists
/interface list add name=WAN comment="Internet connection"
/interface list add name=LAN comment="Internal network"  
/interface list add name=DMZ comment="DMZ network"

/interface list member add list=WAN interface=ether1
/interface list member add list=LAN interface=bridge
/interface list member add list=DMZ interface=ether3

/ip firewall address-list add list=LAN-NET address=192.168.1.0/24 comment="LAN network"
/ip firewall address-list add list=DMZ-NET address=192.168.100.0/24 comment="DMZ network"
/ip firewall address-list add list=MGMT-NET address=203.0.113.0/24 comment="Management network"

# 2. Internet access (masquerade)
/ip firewall nat add chain=srcnat action=masquerade src-address-list=LAN-NET out-interface-list=WAN comment="LAN to internet"
/ip firewall nat add chain=srcnat action=masquerade src-address-list=DMZ-NET out-interface-list=WAN comment="DMZ to internet"

# 3. Public services (port forwarding)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 in-interface-list=WAN to-addresses=192.168.100.10 to-ports=80 comment="HTTP to web server"
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=443 in-interface-list=WAN to-addresses=192.168.100.10 to-ports=443 comment="HTTPS to web server"
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=25 in-interface-list=WAN to-addresses=192.168.100.20 to-ports=25 comment="SMTP to mail server"

# 4. Management access (restricted)
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=2222 src-address-list=MGMT-NET in-interface-list=WAN to-addresses=192.168.1.10 to-ports=22 comment="SSH from management"

# 5. Hair-pin NAT for internal access to public services
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=80 dst-address=203.0.113.10 src-address-list=LAN-NET to-addresses=192.168.100.10 comment="Internal HTTP access"
/ip firewall nat add chain=srcnat action=masquerade protocol=tcp dst-port=80 dst-address=192.168.100.10 src-address-list=LAN-NET comment="HTTP hair-pin SRCNAT"

# 6. LAN to DMZ access control
/ip firewall nat add chain=srcnat action=masquerade src-address-list=LAN-NET dst-address-list=DMZ-NET out-interface-list=DMZ comment="LAN to DMZ access"

# 7. Prevent direct DMZ to LAN access (no NAT rule = blocked by firewall)
# This is handled by firewall filter rules, not NAT
```
{% endcode %}

</details>

## Common NAT scenarios

### Home office setup

{% code overflow="wrap" %}
```bash
# Basic home office with services
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1 comment="Internet access"
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=3389 in-interface=ether1 to-addresses=192.168.1.100 comment="RDP to workstation"
/ip firewall nat add chain=dstnat action=dst-nat protocol=tcp dst-port=8080 in-interface=ether1 to-addresses=192.168.1.50 to-ports=80 comment="Web server on port 8080"
```
{% endcode %}

### Branch office with central services

{% code overflow="wrap" %}
```bash
# Branch office accessing central services
/ip firewall nat add chain=srcnat action=masquerade out-interface=ether1 comment="Branch to internet"
/ip firewall nat add chain=srcnat action=src-nat src-address=192.168.10.0/24 dst-address=192.168.1.0/24 to-addresses=10.0.0.10 out-interface=ipsec-hq comment="Branch to HQ via VPN"
```
{% endcode %}

## Troubleshooting checklist

### Common NAT issues

1. **Port forwarding not working**
   - Check firewall filter rules
   - Verify NAT rule order
   - Confirm destination server is accessible
   - Test from external source

2. **Internet access issues**
   - Verify masquerade interface
   - Check default route
   - Confirm DNS configuration
   - Test with direct IP addresses

3. **Hair-pin NAT problems**
   - Add both DSTNAT and SRCNAT rules
   - Check internal routing
   - Verify address lists
   - Test from correct source networks

4. **Performance issues**
   - Enable FastTrack
   - Optimize rule order
   - Monitor connection table
   - Check hardware capabilities
