---
description: Equal-Cost Multi-Path (ECMP) routing enables load balancing across multiple equal-cost paths for improved bandwidth utilization and redundancy.
icon: shuffle
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

# Equal-Cost Multi-Path (ECMP)

{% hint style="info" %}
ECMP allows RouterOS to utilize multiple paths of equal cost simultaneously, providing both load balancing and automatic failover capabilities for enhanced network performance and resilience.
{% endhint %}

ECMP is available in RouterOS v7+ and provides intelligent traffic distribution across multiple equal-cost paths, improving bandwidth utilization and network redundancy.

***

## ECMP fundamentals

### How ECMP works

**Core concepts:**
- **Equal-cost paths** - Multiple routes with same administrative distance
- **Load balancing** - Traffic distributed across available paths  
- **Hash-based distribution** - Consistent path selection per flow
- **Automatic failover** - Traffic redistributes when paths fail
- **Per-flow load balancing** - Maintains packet order within flows

**ECMP benefits:**
- **Increased bandwidth** - Aggregate capacity of multiple links
- **Improved redundancy** - Automatic failover without route reconvergence
- **Better resource utilization** - Even distribution of network load
- **Enhanced performance** - Reduced congestion and latency

### ECMP vs traditional routing

{% code overflow="wrap" %}
```bash
# Traditional routing (single active path)
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.1 distance=1 comment="Primary path"
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.2 distance=2 comment="Backup path (inactive)"

# ECMP routing (multiple active paths)
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.1 distance=1 comment="ECMP path 1"
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.2 distance=1 comment="ECMP path 2"
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.3 distance=1 comment="ECMP path 3"

# Verify ECMP installation
/ip route print where dst-address=192.168.10.0/24 and active=yes
```
{% endcode %}

***

## Basic ECMP configuration

### Simple dual-path ECMP

Load balancing across two equal paths:

{% code overflow="wrap" %}
```bash
# Configure two default routes with equal distance
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="ISP1 - ECMP Path 1"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=1 comment="ISP2 - ECMP Path 2"

# Verify both routes are active
/ip route print where dst-address=0.0.0.0/0 and active=yes

# Test load balancing
/tool traceroute 8.8.8.8 count=10  # Multiple traces may show different paths
```
{% endcode %}

### Multi-path ECMP setup

Load balancing across multiple paths:

{% code overflow="wrap" %}
```bash
# Four-way ECMP for data center connectivity
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.1 distance=1 comment="DC Path 1"
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.2 distance=1 comment="DC Path 2"  
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.3 distance=1 comment="DC Path 3"
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.4 distance=1 comment="DC Path 4"

# Verify all paths are installed
/ip route print where dst-address=10.0.0.0/8 and active=yes

# Monitor path utilization
/interface monitor-traffic ether2,ether3,ether4,ether5 duration=30
```
{% endcode %}

### ECMP with interface specification

Ensuring proper path selection:

{% code overflow="wrap" %}
```bash
# ECMP with specific interfaces (recommended for clarity)
/ip route add dst-address=192.168.100.0/24 gateway=10.1.1.1 interface=ether2 distance=1 comment="Path via ether2"
/ip route add dst-address=192.168.100.0/24 gateway=10.1.2.1 interface=ether3 distance=1 comment="Path via ether3"
/ip route add dst-address=192.168.100.0/24 gateway=10.1.3.1 interface=ether4 distance=1 comment="Path via ether4"

# Check route installation with interface details
/ip route print detail where dst-address=192.168.100.0/24
```
{% endcode %}

***

## Advanced ECMP scenarios

### Weighted load balancing

Distributing traffic proportionally across unequal links:

{% code overflow="wrap" %}
```bash
# Scenario: ISP1 has 100Mbps, ISP2 has 50Mbps
# Create multiple routes to achieve 2:1 ratio

# ISP1 routes (100Mbps) - 2 routes for double weight
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="ISP1 Path 1"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="ISP1 Path 2"

# ISP2 routes (50Mbps) - 1 route
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=1 comment="ISP2 Path 1"

# Result: ISP1 gets 2/3 of traffic, ISP2 gets 1/3
# Note: This creates multiple identical routes, which may not be supported in all scenarios
```
{% endcode %}

### ECMP with route monitoring

Health checking for ECMP paths:

{% code overflow="wrap" %}
```bash
# ECMP routes with health monitoring
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 \
    target-scope=30 comment="ISP1 with health check"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=1 \
    target-scope=30 comment="ISP2 with health check"

# Advanced monitoring with netwatch
/tool netwatch add host=8.8.8.8 src-address=203.0.113.100 timeout=3s interval=10s \
    up-script="/log info \"ISP1 connectivity OK\"" \
    down-script="/log warning \"ISP1 connectivity FAILED\"; \
                /ip route disable [find gateway=203.0.113.1 and dst-address=0.0.0.0/0]" \
    comment="Monitor ISP1"

/tool netwatch add host=8.8.4.4 src-address=203.0.114.100 timeout=3s interval=10s \
    up-script="/log info \"ISP2 connectivity OK\"; \
               /ip route enable [find gateway=203.0.114.1 and dst-address=0.0.0.0/0]" \
    down-script="/log warning \"ISP2 connectivity FAILED\"; \
                /ip route disable [find gateway=203.0.114.1 and dst-address=0.0.0.0/0]" \
    comment="Monitor ISP2"
```
{% endcode %}

### ECMP with routing tables

Using routing tables for advanced ECMP scenarios:

{% code overflow="wrap" %}
```bash
# Create routing tables for different path groups
/routing table add name=ecmp-group1 fib
/routing table add name=ecmp-group2 fib

# ECMP within each routing table
/ip route add dst-address=192.168.10.0/24 gateway=10.1.1.1 distance=1 \
    routing-table=ecmp-group1 comment="Group1 Path 1"
/ip route add dst-address=192.168.10.0/24 gateway=10.1.2.1 distance=1 \
    routing-table=ecmp-group1 comment="Group1 Path 2"

/ip route add dst-address=192.168.20.0/24 gateway=10.2.1.1 distance=1 \
    routing-table=ecmp-group2 comment="Group2 Path 1"
/ip route add dst-address=192.168.20.0/24 gateway=10.2.2.1 distance=1 \
    routing-table=ecmp-group2 comment="Group2 Path 2"

# Use mangle rules to direct traffic to appropriate routing table
# (See Policy-Based Routing documentation for mangle rule examples)
```
{% endcode %}

***

## ECMP with dynamic routing

### OSPF and ECMP

OSPF naturally supports ECMP for equal-cost paths:

{% code overflow="wrap" %}
```bash
# OSPF configuration that enables ECMP
/routing ospf instance add name=ecmp-ospf router-id=1.1.1.1 disabled=no

# Configure OSPF area
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ecmp-ospf

# Add multiple interfaces to OSPF (creates ECMP opportunities)
/routing ospf interface-template add area=backbone interfaces=ether2 \
    type=ptp cost=10 disabled=no comment="Path to core 1"
/routing ospf interface-template add area=backbone interfaces=ether3 \
    type=ptp cost=10 disabled=no comment="Path to core 2"

# Verify OSPF ECMP routes
/ip route print where ospf=yes and active=yes
/routing ospf lsa print where type=router
```
{% endcode %}

### BGP and ECMP

BGP ECMP requires specific configuration:

{% code overflow="wrap" %}
```bash
# Configure BGP instance
/routing bgp template add name=ecmp-bgp as=65001 disabled=no

# BGP connections to multiple peers with same AS path
/routing bgp connection add template=ecmp-bgp remote.address=203.0.113.1 \
    remote.as=65000 disabled=no comment="BGP Peer 1"
/routing bgp connection add template=ecmp-bgp remote.address=203.0.114.1 \
    remote.as=65000 disabled=no comment="BGP Peer 2"

# Enable multipath for BGP (allows ECMP)
/routing bgp template set ecmp-bgp output.redistribute=connected,static
/routing filter rule add chain=bgp-out action=accept comment="Allow all routes out"

# Verify BGP ECMP installation
/routing bgp session print
/ip route print where bgp=yes and active=yes
```
{% endcode %}

***

## ECMP performance tuning

### Hash algorithm optimization

RouterOS uses hash-based load balancing for consistent flow distribution:

{% code overflow="wrap" %}
```bash
# View current ECMP hash configuration (RouterOS handles this automatically)
/ip settings print

# ECMP uses these fields for hashing:
# - Source IP address
# - Destination IP address  
# - Source port (for TCP/UDP)
# - Destination port (for TCP/UDP)
# - Protocol type

# This ensures packets from the same flow use the same path
# preventing out-of-order delivery
```
{% endcode %}

### Monitoring ECMP performance

Track load distribution and performance:

{% code overflow="wrap" %}
```bash
# Monitor interface statistics for load balancing verification
/interface monitor-traffic ether2,ether3,ether4 duration=60 once

# Check route statistics
/ip route print stats where active=yes

# Monitor per-path utilization
/tool bandwidth-test address=192.168.10.1 direction=both protocol=tcp threads=4

# Script to monitor ECMP balance
:local interfaces {"ether2";"ether3";"ether4"};
:local totalRx 0;
:local totalTx 0;

:foreach iface in=$interfaces do={
    :local stats [/interface get $iface];
    :local rx ($stats->"rx-byte");
    :local tx ($stats->"tx-byte");
    :set totalRx ($totalRx + $rx);
    :set totalTx ($totalTx + $tx);
    
    /log info ("Interface " . $iface . ": RX=" . $rx . " TX=" . $tx);
};

/log info ("Total: RX=" . $totalRx . " TX=" . $totalTx);
```
{% endcode %}

### ECMP troubleshooting

Common issues and solutions:

{% code overflow="wrap" %}
```bash
# Check if routes are properly installed as ECMP
/ip route print where active=yes and dst-address=0.0.0.0/0

# Verify gateway reachability
/ping 203.0.113.1 count=5
/ping 203.0.114.1 count=5

# Test path utilization
:for i from=1 to=10 do={
    /tool traceroute 8.8.8.8 count=1
    :delay 1s
}

# Check for asymmetric routing issues
/tool traceroute 8.8.8.8
/tool traceroute 8.8.4.4

# Monitor route flapping
/log print where topics~"route" and message~"changed"

# Disable/enable routes for testing
/ip route disable [find gateway=203.0.113.1 and dst-address=0.0.0.0/0]
/ip route enable [find gateway=203.0.113.1 and dst-address=0.0.0.0/0]
```
{% endcode %}

***

## ECMP best practices

### Design considerations

1. **Path symmetry** - Ensure return paths are also load balanced
2. **Bandwidth matching** - Use equal or proportional link speeds
3. **Latency consistency** - Similar delay characteristics across paths
4. **MTU alignment** - Consistent MTU across all paths
5. **Monitoring setup** - Implement comprehensive health checking

### Implementation guidelines

1. **Start small** - Begin with dual-path ECMP before scaling
2. **Test thoroughly** - Verify load balancing and failover behavior
3. **Monitor continuously** - Track utilization and performance metrics
4. **Document paths** - Maintain clear documentation of ECMP configuration
5. **Plan for growth** - Design scalable ECMP architecture

### Common pitfalls

1. **Unequal path costs** - Ensure truly equal-cost paths for ECMP
2. **Missing health checks** - Implement proper gateway monitoring
3. **Asymmetric routing** - Consider return path behavior
4. **Over-engineering** - Don't use ECMP where simple redundancy suffices
5. **Insufficient testing** - Test failure scenarios and recovery behavior

***

## Real-world ECMP examples

### Dual-ISP load balancing

{% code overflow="wrap" %}
```bash
# Complete dual-ISP ECMP setup with monitoring
# ISP1: 203.0.113.1 via ether1
# ISP2: 203.0.114.1 via ether2

# Configure IP addresses
/ip address add address=203.0.113.100/30 interface=ether1 comment="ISP1"
/ip address add address=203.0.114.100/30 interface=ether2 comment="ISP2"

# ECMP default routes
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 \
    target-scope=30 comment="ISP1 ECMP"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=1 \
    target-scope=30 comment="ISP2 ECMP"

# Source NAT for both ISPs
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip firewall nat add chain=srcnat out-interface=ether2 action=masquerade

# Health monitoring
/tool netwatch add host=8.8.8.8 src-address=203.0.113.100 timeout=2s interval=10s \
    comment="ISP1 Health Check"
/tool netwatch add host=8.8.4.4 src-address=203.0.114.100 timeout=2s interval=10s \
    comment="ISP2 Health Check"

# Verify ECMP operation
/ip route print where active=yes and dst-address=0.0.0.0/0
```
{% endcode %}

### Data center multi-path connectivity

{% code overflow="wrap" %}
```bash
# Four-way ECMP to data center with LACP alternative
# Links: ether2, ether3, ether4, ether5 to core switches

# Configure uplink interfaces
/ip address add address=172.16.1.10/30 interface=ether2 comment="Core1 Link1"
/ip address add address=172.16.1.14/30 interface=ether3 comment="Core1 Link2"  
/ip address add address=172.16.1.18/30 interface=ether4 comment="Core2 Link1"
/ip address add address=172.16.1.22/30 interface=ether5 comment="Core2 Link2"

# ECMP routes to data center networks
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.9 distance=1 comment="DC via Core1 Link1"
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.13 distance=1 comment="DC via Core1 Link2"
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.17 distance=1 comment="DC via Core2 Link1"  
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.21 distance=1 comment="DC via Core2 Link2"

# Monitor all paths
/tool netwatch add host=10.0.0.1 timeout=1s interval=5s comment="DC Connectivity Test"

# Performance monitoring script
:local interfaces {"ether2";"ether3";"ether4";"ether5"};
:foreach iface in=$interfaces do={
    /log info ("ECMP Path " . $iface . " status: " . [/interface get $iface running]);
};
```
{% endcode %}