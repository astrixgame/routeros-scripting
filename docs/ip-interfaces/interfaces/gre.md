---
description: This section covers how to configure GRE interfaces for IP tunneling.
icon: tunnel
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

# GRE

{% hint style="info" %}
GRE (Generic Routing Encapsulation) creates point-to-point tunnels that can carry various protocols over IP networks, commonly used for site-to-site connections and routing protocol transport.
{% endhint %}

In WinBox you can configure GRE in **Interfaces -> GRE**, or you can use terminal with command `/interface gre`

GRE provides a simple tunneling mechanism that encapsulates packets inside IP packets, allowing different network protocols to be transported over IP networks.

***

## GRE fundamentals

### How GRE works

**Key characteristics:**
- **Protocol 47** - GRE uses IP protocol number 47
- **Stateless** - No connection state maintained
- **Bidirectional** - Traffic can flow in both directions
- **No encryption** - GRE itself provides no security (use with IPSec for encryption)
- **Low overhead** - Minimal encapsulation overhead

**Common use cases:**
- **Site-to-site connectivity** - Connect remote networks
- **Routing protocol transport** - Carry routing protocols over internet
- **MPLS over IP** - Transport MPLS traffic over IP networks
- **Multicast distribution** - Distribute multicast traffic across WAN
- **Remote network access** - Provide access to remote subnets

***

## Basic GRE configuration

### Simple point-to-point GRE tunnel

Create a basic GRE tunnel between two RouterOS devices:

**Router A (Local: 203.0.113.10, Remote: 203.0.113.20):**
{% code overflow="wrap" %}
```bash
# Create GRE interface
/interface gre add name=gre-tunnel1 local-address=203.0.113.10 remote-address=203.0.113.20 disabled=no

# Assign IP address to tunnel
/ip address add address=10.10.10.1/30 interface=gre-tunnel1 comment="GRE tunnel to Site B"
```
{% endcode %}

**Router B (Local: 203.0.113.20, Remote: 203.0.113.10):**
{% code overflow="wrap" %}
```bash
# Create GRE interface (reverse addresses)
/interface gre add name=gre-tunnel1 local-address=203.0.113.20 remote-address=203.0.113.10 disabled=no

# Assign IP address to tunnel
/ip address add address=10.10.10.2/30 interface=gre-tunnel1 comment="GRE tunnel to Site A"
```
{% endcode %}

### Test connectivity

{% code overflow="wrap" %}
```bash
# Test tunnel connectivity
/ping 10.10.10.2 interface=gre-tunnel1

# Check interface status
/interface gre print detail
```
{% endcode %}

***

## Advanced GRE configuration

### GRE with keepalive

Enable keepalive to detect tunnel failures:

{% code overflow="wrap" %}
```bash
# Configure GRE with keepalive
/interface gre add name=gre-keepalive \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    keepalive=10,3 \
    disabled=no

# keepalive=10,3 means send keepalive every 10 seconds, timeout after 3 failures
```
{% endcode %}

### GRE over dynamic IP

For scenarios where one end has dynamic IP:

{% code overflow="wrap" %}
```bash
# Server side (static IP)
/interface gre add name=gre-dynamic \
    local-address=203.0.113.10 \
    remote-address=0.0.0.0 \
    allow-fast-path=no \
    disabled=no

# Client side (dynamic IP) 
/interface gre add name=gre-server \
    local-address=0.0.0.0 \
    remote-address=203.0.113.10 \
    disabled=no
```
{% endcode %}

### Multiple GRE tunnels

Create multiple tunnels for redundancy or load balancing:

{% code overflow="wrap" %}
```bash
# Primary tunnel
/interface gre add name=gre-primary \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    disabled=no

# Backup tunnel (different path)
/interface gre add name=gre-backup \
    local-address=203.0.113.11 \
    remote-address=203.0.113.21 \
    disabled=no

# Assign IP addresses
/ip address add address=10.10.10.1/30 interface=gre-primary
/ip address add address=10.10.11.1/30 interface=gre-backup
```
{% endcode %}

***

## GRE with routing protocols

### OSPF over GRE

Extend OSPF across WAN using GRE:

{% code overflow="wrap" %}
```bash
# Create GRE tunnel
/interface gre add name=gre-ospf local-address=203.0.113.10 remote-address=203.0.113.20 disabled=no
/ip address add address=172.16.0.1/30 interface=gre-ospf

# Configure OSPF on tunnel
/routing ospf instance add name=main router-id=1.1.1.1 disabled=no
/routing ospf area add name=backbone area-id=0.0.0.0 instance=main
/routing ospf interface-template add area=backbone interfaces=gre-ospf type=ptp disabled=no

# Add local networks to OSPF
/routing ospf interface-template add area=backbone interfaces=ether2 disabled=no
```
{% endcode %}

### BGP over GRE

Use GRE for BGP peering across internet:

{% code overflow="wrap" %}
```bash
# Create GRE tunnel
/interface gre add name=gre-bgp local-address=203.0.113.10 remote-address=203.0.113.20 disabled=no
/ip address add address=172.16.1.1/30 interface=gre-bgp

# Configure BGP over tunnel
/routing bgp template add name=ebgp-peers as=65001 router-id=1.1.1.1 disabled=no
/routing bgp connection add name=peer-site-b template=ebgp-peers remote.address=172.16.1.2 remote.as=65002 local.role=ebgp disabled=no
```
{% endcode %}

***

## GRE security with IPSec

### Encrypt GRE traffic with IPSec

Since GRE provides no encryption, use IPSec for security:

{% code overflow="wrap" %}
```bash
# Create IPSec proposal
/ip ipsec proposal add name=gre-proposal enc-algorithms=aes-256-cbc auth-algorithms=sha256 pfs-group=modp2048

# Create IPSec peer
/ip ipsec peer add name=gre-peer address=203.0.113.20 exchange-mode=ike2 passive=no

# Create IPSec identity
/ip ipsec identity add peer=gre-peer secret="your-shared-secret" my-id=address:203.0.113.10 remote-id=address:203.0.113.20

# Create IPSec policy for GRE traffic
/ip ipsec policy add src-address=203.0.113.10/32 dst-address=203.0.113.20/32 protocol=gre action=encrypt proposal=gre-proposal peer=gre-peer

# Create GRE tunnel (will be encrypted by IPSec)
/interface gre add name=gre-secure local-address=203.0.113.10 remote-address=203.0.113.20 disabled=no
/ip address add address=192.168.100.1/30 interface=gre-secure
```
{% endcode %}

***

## GRE over NAT

### Handle GRE behind NAT

GRE can be problematic behind NAT due to protocol 47:

{% code overflow="wrap" %}
```bash
# If RouterOS is behind NAT, use port forwarding for protocol 47
# On the NAT router:
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=192.168.1.100 protocol=gre comment="Forward GRE to internal router"

# Or use GRE-in-UDP (if supported by both ends)
/interface gre add name=gre-udp \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    gre-udp=yes \
    gre-udp-port=4754 \
    disabled=no
```
{% endcode %}

***

## Monitoring and troubleshooting

### Monitor GRE tunnels

{% code overflow="wrap" %}
```bash
# Check GRE interface status
/interface gre print detail

# Monitor tunnel traffic
/interface monitor-traffic interface=gre-tunnel1

# Check tunnel statistics
/interface print stats where name=gre-tunnel1

# Monitor keepalive status
/log print where topics~"gre"
```
{% endcode %}

### Troubleshoot GRE issues

{% code overflow="wrap" %}
```bash
# Test underlying connectivity
/ping 203.0.113.20 interface=ether1

# Check firewall rules
/ip firewall filter print where protocol=gre

# Test tunnel connectivity
/ping 10.10.10.2 interface=gre-tunnel1

# Check routing
/ip route print where gateway~"gre"

# Packet capture
/tool sniffer start interface=ether1 filter-protocol=gre file-name=gre-debug.pcap
```
{% endcode %}

***

## Performance optimization

### Optimize GRE performance

{% code overflow="wrap" %}
```bash
# Disable fast-path if having issues (enables software processing)
/interface gre set gre-tunnel1 allow-fast-path=no

# Set appropriate MTU (account for GRE overhead)
/interface gre set gre-tunnel1 mtu=1476

# Monitor performance
/interface monitor-traffic interface=gre-tunnel1 duration=10
```
{% endcode %}

### Load balancing with multiple tunnels

{% code overflow="wrap" %}
```bash
# Create equal-cost routes over multiple GRE tunnels
/ip route add dst-address=192.168.2.0/24 gateway=gre-tunnel1 distance=1
/ip route add dst-address=192.168.2.0/24 gateway=gre-tunnel2 distance=1

# Or use ECMP routing
/ip route add dst-address=192.168.2.0/24 gateway=gre-tunnel1,gre-tunnel2 distance=1
```
{% endcode %}

***

## Common use cases

### Site-to-site network extension

{% code overflow="wrap" %}
```bash
# Router A (Site A: 192.168.1.0/24)
/interface gre add name=gre-site-b local-address=203.0.113.10 remote-address=203.0.113.20 disabled=no
/ip address add address=10.255.0.1/30 interface=gre-site-b
/ip route add dst-address=192.168.2.0/24 gateway=gre-site-b

# Router B (Site B: 192.168.2.0/24)  
/interface gre add name=gre-site-a local-address=203.0.113.20 remote-address=203.0.113.10 disabled=no
/ip address add address=10.255.0.2/30 interface=gre-site-a
/ip route add dst-address=192.168.1.0/24 gateway=gre-site-a
```
{% endcode %}

### Remote office connectivity

{% code overflow="wrap" %}
```bash
# Main office (hub)
/interface gre add name=gre-branch1 local-address=203.0.113.1 remote-address=203.0.113.10 disabled=no
/interface gre add name=gre-branch2 local-address=203.0.113.1 remote-address=203.0.113.20 disabled=no

/ip address add address=10.0.1.1/30 interface=gre-branch1
/ip address add address=10.0.2.1/30 interface=gre-branch2

# Add routes to branch networks
/ip route add dst-address=192.168.10.0/24 gateway=gre-branch1
/ip route add dst-address=192.168.20.0/24 gateway=gre-branch2
```
{% endcode %}

***

<details>

<summary>Show complete GRE tunnel setup</summary>

{% code overflow="wrap" %}
```bash
# Site A Configuration (Public IP: 203.0.113.10)
# 1. Create GRE tunnel
/interface gre add name=gre-to-site-b \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    keepalive=10,3 \
    disabled=no

# 2. Assign tunnel IP
/ip address add address=172.16.0.1/30 interface=gre-to-site-b comment="Tunnel to Site B"

# 3. Configure firewall to allow GRE
/ip firewall filter add chain=input action=accept protocol=gre src-address=203.0.113.20 comment="Allow GRE from Site B"

# 4. Add routes to remote networks
/ip route add dst-address=192.168.2.0/24 gateway=gre-to-site-b distance=1 comment="Route to Site B LAN"

# 5. Configure routing protocol (optional)
/routing ospf instance add name=main router-id=1.1.1.1 disabled=no
/routing ospf area add name=backbone area-id=0.0.0.0 instance=main
/routing ospf interface-template add area=backbone interfaces=gre-to-site-b type=ptp disabled=no
/routing ospf interface-template add area=backbone interfaces=ether2 disabled=no

# Site B Configuration (Public IP: 203.0.113.20) - Mirror setup
/interface gre add name=gre-to-site-a local-address=203.0.113.20 remote-address=203.0.113.10 keepalive=10,3 disabled=no
/ip address add address=172.16.0.2/30 interface=gre-to-site-a comment="Tunnel to Site A"
/ip firewall filter add chain=input action=accept protocol=gre src-address=203.0.113.10 comment="Allow GRE from Site A"
/ip route add dst-address=192.168.1.0/24 gateway=gre-to-site-a distance=1 comment="Route to Site A LAN"
```
{% endcode %}

</details>

## Best practices

### Design recommendations

1. **Use keepalives** - Detect tunnel failures quickly
2. **Implement redundancy** - Multiple tunnels for critical connections  
3. **Secure tunnels** - Use IPSec for encryption over untrusted networks
4. **Monitor actively** - Watch for tunnel state changes
5. **Plan addressing** - Use consistent tunnel addressing schemes

### Security considerations

1. **Always encrypt** - GRE provides no security by itself
2. **Firewall properly** - Allow only necessary GRE traffic
3. **Use authentication** - IPSec or other authentication mechanisms
4. **Monitor traffic** - Watch for unusual patterns
5. **Regular maintenance** - Keep tunnel configurations updated

### Performance tips

1. **Optimize MTU** - Account for GRE overhead
2. **Use hardware acceleration** - When available
3. **Monitor bandwidth** - Track utilization on tunnels
4. **Load balance** - Use multiple tunnels for high bandwidth needs
5. **Minimize latency** - Choose optimal routing paths