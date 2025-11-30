---
description: This section covers RouterOS routing configuration including static routes, dynamic protocols, policy-based routing, and advanced routing features.
icon: route
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

# Routing

{% hint style="info" %}
RouterOS provides comprehensive routing capabilities including static routing, dynamic routing protocols (OSPF, BGP, RIP), policy-based routing, multicast routing, VRF, ECMP, and MPLS for complex network topologies and traffic engineering.
{% endhint %}

In WinBox you can configure routing in **Routing**, or you can use terminal with command `/routing`

RouterOS v7+ introduced a completely redesigned routing subsystem with improved performance, better scalability, and enhanced features for modern networking requirements.

***

## Routing fundamentals

### RouterOS v7 routing architecture

**New routing features:**
- **Unified routing table** - Single FIB (Forwarding Information Base)
- **Improved performance** - Hardware acceleration support
- **Better scalability** - Support for larger routing tables
- **Enhanced protocols** - Updated implementations of OSPF, BGP, RIP
- **Policy framework** - Advanced routing policies and filters

**Core routing concepts:**
- **Static routes** - Manually configured routes
- **Dynamic protocols** - Automatic route learning and distribution
- **Administrative distance** - Route preference system
- **Routing filters** - Control route advertisement and acceptance
- **Route maps** - Policy-based routing modifications

### Routing table structure

{% code overflow="wrap" %}
```bash
# View routing table
/ip route print

# View detailed route information
/ip route print detail

# View specific route types
/ip route print where dynamic=yes
/ip route print where static=yes
```
{% endcode %}

***

## Basic routing configuration

### Default route configuration

Essential for internet connectivity:

{% code overflow="wrap" %}
```bash
# Add default route to internet gateway
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="Default route to internet"

# Default route with interface (when gateway is on same subnet)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 interface=ether1 distance=1 comment="Default via ether1"

# Default route for DHCP client (automatic)
/ip dhcp-client add interface=ether1 disabled=no add-default-route=yes comment="DHCP client with default route"
```
{% endcode %}

### Basic static routing

{% code overflow="wrap" %}
```bash
# Route to specific network
/ip route add dst-address=192.168.2.0/24 gateway=192.168.1.1 distance=1 comment="Route to branch office"

# Route via specific interface  
/ip route add dst-address=10.0.0.0/8 gateway=192.168.1.100 interface=ether2 distance=1 comment="Route to internal networks"

# Multiple gateways for redundancy
/ip route add dst-address=192.168.3.0/24 gateway=192.168.1.1 distance=1 comment="Primary path"
/ip route add dst-address=192.168.3.0/24 gateway=192.168.1.2 distance=2 comment="Backup path"
```
{% endcode %}

### Route verification

{% code overflow="wrap" %}
```bash
# Test connectivity
/ping 192.168.2.1

# Trace route path
/tool traceroute 192.168.2.1

# Check route resolution  
/ip route print where dst-address=192.168.2.0/24

# Monitor route changes
/log print where topics~"route"
```
{% endcode %}

***

## Routing topics

### Static routing

- **[Static Routes](static.md)** - Manual route configuration
  - Basic static route configuration
  - Route priorities and administrative distance
  - Redundant path configuration
  - Static route monitoring and troubleshooting

### Dynamic routing protocols

- **[Dynamic Routing](dynamic/README.md)** - Automatic route discovery
  - **[OSPF](dynamic/ospf.md)** - Open Shortest Path First
    - Area configuration and design
    - LSA types and database management
    - Multi-area OSPF deployments
    - OSPF authentication and security
  - **[BGP](dynamic/bgp.md)** - Border Gateway Protocol  
    - eBGP and iBGP configuration
    - Route filtering and policy
    - BGP communities and attributes
    - Advanced BGP features
  - **[RIP](dynamic/rip.md)** - Routing Information Protocol
    - RIPv1 and RIPv2 configuration
    - Route redistribution
    - Legacy network support

### Advanced routing features

- **[Policy-Based Routing (PBR)](pbr.md)** - Route based on criteria other than destination
  - Source-based routing
  - Application-based routing
  - Load balancing with PBR
  - Multi-WAN scenarios

- **[Equal-Cost Multi-Path (ECMP)](ecmp.md)** - Load balancing across multiple paths
  - ECMP configuration and tuning
  - Hash-based load balancing
  - Failover scenarios
  - Performance optimization

- **[Virtual Routing and Forwarding (VRF)](vrf.md)** - Network virtualization
  - VRF instance creation and management
  - Route leaking between VRFs
  - MPLS VPN integration
  - Multi-tenant routing

- **[MPLS](mpls.md)** - Multiprotocol Label Switching
  - MPLS fundamentals and LDP
  - MPLS VPN configuration
  - Traffic engineering
  - QoS with MPLS

### Multicast routing

- **[Multicast Routing](multicast/README.md)** - Efficient one-to-many communication
  - **[IGMP Proxy](multicast/igmp-proxy.md)** - Internet Group Management Protocol
  - **[PIM](multicast/pim.md)** - Protocol Independent Multicast

***

## Route filtering and policies

### Basic route filtering

Control route advertisement and acceptance:

{% code overflow="wrap" %}
```bash
# Create routing filter to block specific networks
/routing filter rule add chain=ospf-in action=discard prefix=10.0.0.0/8 comment="Block RFC1918 Class A"

# Allow only specific prefixes
/routing filter rule add chain=bgp-out action=accept prefix=203.0.113.0/24 prefix-length=24 comment="Advertise our network"
/routing filter rule add chain=bgp-out action=discard comment="Block all other routes"

# Filter by AS path (BGP)
/routing filter rule add chain=bgp-in action=discard bgp-as-path=".*65001.*" comment="Block routes from AS65001"
```
{% endcode %}

### Route manipulation

Modify route attributes:

{% code overflow="wrap" %}
```bash
# Set route preferences
/routing filter rule add chain=ospf-in action=accept set-distance=100 comment="Set OSPF distance to 100"

# Modify BGP attributes
/routing filter rule add chain=bgp-out action=accept set-bgp-local-pref=200 prefix=192.168.1.0/24 comment="Increase local preference"

# Set route tags
/routing filter rule add chain=ospf-out action=accept set-route-tag=100 comment="Tag exported routes"
```
{% endcode %}

***

## Multi-WAN routing

### Load balancing setup

Distribute traffic across multiple internet connections:

{% code overflow="wrap" %}
```bash
# Create routing marks for different WANs
/routing table add name=wan1 fib
/routing table add name=wan2 fib

# Mark traffic for load balancing (requires mangle rules)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=wan1 distance=1 comment="WAN1 default route"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=wan2 distance=1 comment="WAN2 default route"

# Main routing table for unmarked traffic  
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="Primary default route"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 comment="Backup default route"
```
{% endcode %}

### Failover configuration

Automatic failover between connections:

{% code overflow="wrap" %}
```bash
# Primary route with gateway monitoring
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 \
    target-scope=30 comment="Primary WAN with monitoring"

# Backup route (higher distance)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 \
    comment="Backup WAN"

# Enable gateway health checking
/tool netwatch add host=8.8.8.8 timeout=1s interval=10s comment="Monitor primary WAN connectivity"
```
{% endcode %}

***

## IPv6 routing

### Basic IPv6 routing

{% code overflow="wrap" %}
```bash
# IPv6 default route
/ipv6 route add dst-address=::/0 gateway=2001:db8::1 distance=1 comment="IPv6 default route"

# IPv6 static routes
/ipv6 route add dst-address=2001:db8:2::/64 gateway=2001:db8:1::1 distance=1 comment="IPv6 static route"

# View IPv6 routing table
/ipv6 route print
```
{% endcode %}

### IPv6 dynamic routing

{% code overflow="wrap" %}
```bash
# Enable IPv6 OSPF
/routing ospf instance set default router-id=1.1.1.1 disabled=no

# Configure IPv6 OSPF area
/routing ospf area add name=backbone area-id=0.0.0.0 instance=default

# Add IPv6 networks to OSPF
/routing ospf interface-template add area=backbone interfaces=ether2 type=broadcast disabled=no
```
{% endcode %}

***

## Monitoring and troubleshooting

### Route monitoring

{% code overflow="wrap" %}
```bash
# Monitor routing table changes
/log print where topics~"route"

# Check route statistics
/ip route print stats

# Monitor protocol status
/routing ospf neighbor print
/routing bgp session print

# Check route resolution
/tool traceroute 8.8.8.8
```
{% endcode %}

### Troubleshooting connectivity

{% code overflow="wrap" %}
```bash
# Debug routing issues
/ip route print where dst-address=192.168.1.0/24

# Check interface status
/interface print where disabled=no

# Test gateway connectivity
/ping 203.0.113.1

# Monitor route convergence
/routing ospf lsa print
/routing bgp advertisement print
```
{% endcode %}

### Performance monitoring

{% code overflow="wrap" %}
```bash
# Monitor routing table size
/ip route print count-only

# Check memory usage
/system resource print

# Monitor CPU usage during convergence
/system resource monitor numbers=0 duration=30

# Check hardware acceleration
/interface print detail where name=ether1
```
{% endcode %}

***

## Routing best practices

### Design recommendations

1. **Hierarchical design** - Use structured network topology
2. **Redundancy planning** - Multiple paths for critical routes
3. **Scalability** - Design for growth and expansion
4. **Security** - Filter and authenticate routing updates
5. **Documentation** - Document all static routes and policies

### Performance optimization

1. **Hardware acceleration** - Use supported features when available
2. **Route summarization** - Reduce routing table size
3. **Filter optimization** - Efficient routing filters
4. **Protocol tuning** - Adjust timers for network requirements  
5. **Memory management** - Monitor resource usage

### Troubleshooting guidelines

1. **Systematic approach** - Check layer by layer
2. **Use tools** - Leverage built-in diagnostic tools
3. **Monitor logs** - Watch for routing protocol messages
4. **Test connectivity** - Verify end-to-end reachability
5. **Document issues** - Keep troubleshooting records

***

<details>

<summary>Show complete basic routing setup</summary>

{% code overflow="wrap" %}
```bash
# Complete basic routing setup for small to medium enterprise
# 1. Basic static routes for essential connectivity
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="Primary internet gateway"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 comment="Backup internet gateway"

# 2. Internal network routes
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.10 distance=1 comment="DMZ network"
/ip route add dst-address=192.168.20.0/24 gateway=192.168.1.20 distance=1 comment="Guest network"

# 3. Site-to-site VPN routes
/ip route add dst-address=192.168.100.0/24 gateway=ipsec-site-b distance=1 comment="Branch office A"
/ip route add dst-address=192.168.200.0/24 gateway=ipsec-site-c distance=1 comment="Branch office B"

# 4. OSPF for internal routing
/routing ospf instance add name=internal router-id=1.1.1.1 disabled=no
/routing ospf area add name=backbone area-id=0.0.0.0 instance=internal

# 5. Add networks to OSPF
/routing ospf interface-template add area=backbone interfaces=bridge type=broadcast disabled=no
/routing ospf interface-template add area=backbone interfaces=ether2 type=ptp disabled=no

# 6. Route filtering for OSPF
/routing filter rule add chain=ospf-in action=discard prefix=0.0.0.0/0 comment="Block default route in OSPF"
/routing filter rule add chain=ospf-in action=accept comment="Accept other OSPF routes"

# 7. Monitoring and health checking
/tool netwatch add host=8.8.8.8 timeout=2s interval=30s comment="Monitor internet connectivity"
/tool netwatch add host=192.168.100.1 timeout=1s interval=10s comment="Monitor branch office A"

# 8. IPv6 routing (if used)
/ipv6 route add dst-address=::/0 gateway=2001:db8::1 distance=1 comment="IPv6 default route"
/ipv6 route add dst-address=2001:db8:100::/64 gateway=2001:db8:1::10 distance=1 comment="IPv6 DMZ"
```
{% endcode %}

</details>

## Common routing scenarios

### Branch office connectivity

{% code overflow="wrap" %}
```bash
# Hub and spoke topology
/ip route add dst-address=192.168.10.0/24 gateway=ipsec-branch1 comment="Branch 1"
/ip route add dst-address=192.168.20.0/24 gateway=ipsec-branch2 comment="Branch 2"
/ip route add dst-address=192.168.30.0/24 gateway=ipsec-branch3 comment="Branch 3"

# Summarized route advertisement from hub
/routing filter rule add chain=ospf-out action=accept prefix=192.168.0.0/16 prefix-length=16-24
```
{% endcode %}

### Data center connectivity

{% code overflow="wrap" %}
```bash
# Multiple paths to data center
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.1 distance=1 comment="Primary DC path"
/ip route add dst-address=10.0.0.0/8 gateway=172.16.2.1 distance=2 comment="Backup DC path"

# Server farm routing
/ip route add dst-address=10.1.0.0/16 gateway=10.0.1.1 comment="Web servers"
/ip route add dst-address=10.2.0.0/16 gateway=10.0.2.1 comment="Database servers"
```
{% endcode %}