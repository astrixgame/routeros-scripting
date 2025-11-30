---
description: Dynamic routing protocols enable automatic route discovery, distribution, and adaptation to network topology changes for scalable and resilient networks.
icon: git-branch
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

# Dynamic Routing Protocols

{% hint style="info" %}
Dynamic routing protocols automatically discover network topology, calculate optimal paths, and adapt to changes, providing scalable and self-healing network infrastructure compared to static routing.
{% endhint %}

RouterOS v7+ supports modern implementations of OSPF, BGP, and RIP with enhanced performance, better convergence times, and improved scalability for enterprise and service provider networks.

***

## Dynamic routing overview

### Protocol comparison

| Protocol | Type | Use Case | Convergence | Scalability | Complexity |
|----------|------|----------|-------------|-------------|------------|
| **OSPF** | Link State | Enterprise, Campus | Fast | Medium-Large | Medium |
| **BGP** | Path Vector | Internet, SP Core | Slow | Very Large | High |
| **RIP** | Distance Vector | Small Networks | Slow | Small | Low |

### Key advantages over static routing

**Automatic adaptation:**
- Routes update automatically when topology changes
- No manual intervention required for failures
- Optimal path calculation based on metrics

**Scalability benefits:**
- Supports large networks with hundreds of routers
- Hierarchical design reduces routing overhead
- Load balancing across multiple equal-cost paths

**Reduced administrative overhead:**
- Less manual configuration and maintenance
- Automatic discovery of network topology
- Built-in loop prevention mechanisms

***

## Protocol selection guide

### OSPF - Open Shortest Path First

**Best for:**
- Enterprise networks with multiple subnets
- Campus networks requiring fast convergence
- Networks needing hierarchical design
- Medium to large networks (up to 500 routers per area)

**Key features:**
- Area-based hierarchy for scalability
- Fast convergence (sub-second with proper tuning)
- Support for ECMP (Equal-Cost Multi-Path)
- Advanced authentication and security

{% code overflow="wrap" %}
```bash
# OSPF deployment scenario
# Enterprise campus with core, distribution, and access layers
/routing ospf instance add name=campus router-id=1.1.1.1 disabled=no

# Backbone area (Area 0) - Core layer
/routing ospf area add name=backbone area-id=0.0.0.0 instance=campus

# Regular areas - Distribution layers  
/routing ospf area add name=building-a area-id=0.0.0.1 instance=campus
/routing ospf area add name=building-b area-id=0.0.0.2 instance=campus

# Interface assignments
/routing ospf interface-template add area=backbone interfaces=ether1,ether2 \
    type=ptp disabled=no comment="Core links"
/routing ospf interface-template add area=building-a interfaces=ether3 \
    type=broadcast disabled=no comment="Building A distribution"
```
{% endcode %}

### BGP - Border Gateway Protocol

**Best for:**
- Internet connectivity and multi-homing
- Service provider networks
- Large enterprise WAN connectivity
- Policy-based routing requirements

**Key features:**
- Advanced policy control with route maps
- Support for multiple address families
- Extensive path attributes for traffic engineering
- Designed for internet-scale routing

{% code overflow="wrap" %}
```bash
# BGP deployment scenario
# Multi-homed enterprise with two ISP connections
/routing bgp template add name=enterprise-bgp as=65001 router-id=1.1.1.1 disabled=no

# eBGP connections to ISPs
/routing bgp connection add template=enterprise-bgp \
    remote.address=203.0.113.1 remote.as=65100 disabled=no comment="ISP1"
/routing bgp connection add template=enterprise-bgp \
    remote.address=203.0.114.1 remote.as=65200 disabled=no comment="ISP2"

# Advertise company networks
/routing filter rule add chain=bgp-out action=accept \
    prefix=203.0.115.0/24 comment="Advertise company network"
```
{% endcode %}

### RIP - Routing Information Protocol

**Best for:**
- Small networks (< 15 routers)
- Simple network topologies
- Legacy network integration
- Networks requiring minimal configuration

**Key features:**
- Simple configuration and troubleshooting
- Automatic route summarization
- Built-in loop prevention (split horizon)
- Low resource requirements

{% code overflow="wrap" %}
```bash
# RIP deployment scenario
# Small branch office network
/routing rip instance add name=branch-rip disabled=no

# Add networks to RIP
/routing rip interface-template add interfaces=ether1,ether2 \
    disabled=no comment="Branch office interfaces"

# Advertise local networks
/routing rip network add network=192.168.1.0/24 comment="LAN network"
/routing rip network add network=192.168.10.0/24 comment="Server network"
```
{% endcode %}

***

## Dynamic routing design principles

### Hierarchical design

Structure networks for optimal scalability and performance:

{% code overflow="wrap" %}
```bash
# Three-tier network design with OSPF
# Core tier (Area 0) - High-speed backbone
# Distribution tier (Regular areas) - Policy enforcement
# Access tier (Stub areas) - End-user connectivity

# Core area configuration (Area 0)
/routing ospf area add name=core area-id=0.0.0.0 instance=main \
    comment="Backbone area - core tier"

# Distribution areas
/routing ospf area add name=office-floors area-id=0.0.0.10 \
    instance=main type=default comment="Office floors distribution"

# Access areas (stub areas for efficiency)
/routing ospf area add name=floor1-access area-id=0.0.1.0 \
    instance=main type=stub default-cost=10 comment="Floor 1 access"
/routing ospf area add name=floor2-access area-id=0.0.2.0 \
    instance=main type=stub default-cost=10 comment="Floor 2 access"

# Interface assignments per tier
/routing ospf interface-template add area=core interfaces=ether1,ether2 \
    type=ptp cost=1 disabled=no comment="Core tier links"

/routing ospf interface-template add area=office-floors interfaces=ether3,ether4 \
    type=broadcast cost=10 disabled=no comment="Distribution links"

/routing ospf interface-template add area=floor1-access interfaces=ether5 \
    type=broadcast cost=100 disabled=no comment="Access tier"
```
{% endcode %}

### Redundancy and failover

Design for high availability:

{% code overflow="wrap" %}
```bash
# Dual-router redundancy with OSPF
# Primary and backup routers for critical paths

# Primary router OSPF configuration
/routing ospf instance add name=primary router-id=1.1.1.1 disabled=no

# Lower cost on primary paths
/routing ospf interface-template add area=backbone interfaces=ether1 \
    cost=10 disabled=no comment="Primary path - lower cost"

# Backup router OSPF configuration  
/routing ospf instance add name=backup router-id=1.1.1.2 disabled=no

# Higher cost on backup paths (used only when primary fails)
/routing ospf interface-template add area=backbone interfaces=ether1 \
    cost=50 disabled=no comment="Backup path - higher cost"

# BGP redundancy with path manipulation
/routing bgp template add name=redundant-bgp as=65001 router-id=1.1.1.1 disabled=no

# Primary ISP connection (preferred)
/routing bgp connection add template=redundant-bgp \
    remote.address=203.0.113.1 remote.as=65100 disabled=no comment="Primary ISP"

# Backup ISP connection (with AS path prepending to make less preferred)
/routing bgp connection add template=redundant-bgp \
    remote.address=203.0.114.1 remote.as=65200 disabled=no comment="Backup ISP"

# Route filtering to prefer primary path
/routing filter rule add chain=bgp-out dst-address=203.0.115.0/24 \
    set-bgp-prepend=3 bgp-as-path-length=1 comment="Prepend AS path on backup"
```
{% endcode %}

***

## Protocol integration scenarios

### OSPF + BGP integration

Combine internal OSPF with external BGP:

{% code overflow="wrap" %}
```bash
# Enterprise scenario: OSPF for internal routing, BGP for internet
# OSPF handles internal corporate networks
# BGP provides internet connectivity and multi-homing

# Internal OSPF configuration
/routing ospf instance add name=internal router-id=10.1.1.1 disabled=no
/routing ospf area add name=corporate area-id=0.0.0.0 instance=internal

# OSPF for internal networks
/routing ospf interface-template add area=corporate \
    interfaces=bridge,ether3,ether4 disabled=no comment="Internal networks"

# Advertise internal networks in OSPF
/routing ospf network add network=192.168.0.0/16 area=corporate

# External BGP for internet connectivity
/routing bgp template add name=internet-bgp as=65001 router-id=10.1.1.1 disabled=no

# BGP connections to ISPs
/routing bgp connection add template=internet-bgp \
    remote.address=203.0.113.1 remote.as=65100 disabled=no comment="ISP1"
/routing bgp connection add template=internet-bgp \
    remote.address=203.0.114.1 remote.as=65200 disabled=no comment="ISP2"

# Route redistribution: OSPF -> BGP (advertise company networks)
/routing filter rule add chain=bgp-out action=accept \
    prefix=192.168.0.0/16 prefix-length=16-24 comment="Advertise company networks"

# Route redistribution: BGP -> OSPF (default route injection)
/routing ospf instance set internal redistribute=bgp
/routing filter rule add chain=ospf-out action=accept \
    prefix=0.0.0.0/0 set-distance=1 comment="Inject default route"
```
{% endcode %}

### Multi-protocol environments

Handle mixed protocol environments:

{% code overflow="wrap" %}
```bash
# Complex network with OSPF, BGP, and RIP integration
# Main campus: OSPF
# Branch offices: RIP (simple management)
# Internet/WAN: BGP

# Main campus OSPF
/routing ospf instance add name=campus router-id=1.1.1.1 disabled=no
/routing ospf area add name=campus-core area-id=0.0.0.0 instance=campus

# Branch office RIP
/routing rip instance add name=branches disabled=no

# Add branch networks to RIP
/routing rip interface-template add interfaces=ether5 disabled=no comment="Branch connection"

# Internet BGP
/routing bgp template add name=wan-bgp as=65001 router-id=1.1.1.1 disabled=no
/routing bgp connection add template=wan-bgp remote.address=203.0.113.1 \
    remote.as=65100 disabled=no comment="WAN provider"

# Route redistribution between protocols
# RIP -> OSPF (branch networks into campus)
/routing ospf instance set campus redistribute=rip
/routing filter rule add chain=ospf-out action=accept \
    rip=yes set-distance=150 comment="Redistribute RIP routes"

# OSPF -> RIP (campus networks to branches)  
/routing rip instance set branches redistribute=ospf
/routing filter rule add chain=rip-out action=accept \
    ospf=yes comment="Redistribute OSPF routes to branches"

# BGP -> OSPF (default route injection)
/routing ospf instance set campus redistribute=bgp  
/routing filter rule add chain=ospf-out action=accept \
    prefix=0.0.0.0/0 bgp=yes comment="Inject internet default route"
```
{% endcode %}

***

## Performance optimization

### Convergence tuning

Optimize protocol timers for faster convergence:

{% code overflow="wrap" %}
```bash
# OSPF fast convergence tuning
/routing ospf area set backbone \
    lsa-min-interval=1s \
    spf-delay=1s \
    spf-hold-time=5s \
    spf-max-hold-time=10s

# OSPF interface tuning for fast hello/dead intervals
/routing ospf interface-template set [find area=backbone] \
    hello-interval=5s \
    dead-interval=15s \
    retransmit-interval=2s

# BGP convergence optimization
/routing bgp template set internet-bgp \
    keepalive-time=30s \
    hold-time=90s \
    connect-retry-time=10s

# Enable BGP fast external failover
/routing bgp connection set [find remote.as!=65001] \
    holdtime-profiles=default \
    tcp-md5-key=""  # Add authentication if needed
```
{% endcode %}

### Scalability optimization

Configure protocols for large-scale deployments:

{% code overflow="wrap" %}
```bash
# OSPF scalability improvements
# Use area summarization to reduce LSA flooding
/routing ospf area set building-networks \
    area-range=192.168.0.0/16 advertise=yes comment="Summarize building networks"

# Enable OSPF stub areas to reduce routing table size
/routing ospf area set access-networks type=stub \
    default-cost=100 comment="Stub area for access networks"

# BGP scalability with route reflectors
/routing bgp template add name=route-reflector as=65001 \
    cluster-id=1.1.1.1 disabled=no comment="BGP Route Reflector"

# Configure route reflector clients
/routing bgp connection add template=route-reflector \
    remote.address=2.2.2.2 remote.as=65001 \
    route-reflect=yes disabled=no comment="RR Client 1"

# Route filtering for scalability
/routing filter rule add chain=bgp-in action=discard \
    prefix=0.0.0.0/0 prefix-length=0-7 comment="Block too-general prefixes"

/routing filter rule add chain=ospf-in action=discard \
    prefix=127.0.0.0/8 comment="Block loopback networks"
```
{% endcode %}

***

## Monitoring and troubleshooting

### Protocol monitoring

Track routing protocol health and performance:

{% code overflow="wrap" %}
```bash
# OSPF monitoring commands
/routing ospf neighbor print  # Check neighbor states
/routing ospf lsa print  # Examine LSA database
/routing ospf area print  # Area status and statistics
/routing ospf interface print  # Interface status

# BGP monitoring commands  
/routing bgp session print  # Session status and statistics
/routing bgp advertisements print  # Routes being advertised
/routing bgp received-routes print  # Routes received from peers

# RIP monitoring commands
/routing rip neighbor print  # RIP neighbor status
/routing rip interface print  # RIP interface statistics

# General route monitoring
/ip route print where dynamic=yes  # All dynamic routes
/ip route print stats  # Route statistics and hit counts
```
{% endcode %}

### Troubleshooting procedures

Systematic approach to routing protocol issues:

{% code overflow="wrap" %}
```bash
# Step 1: Check basic connectivity
/ping neighbor-ip count=5

# Step 2: Verify protocol configuration
/routing ospf instance print detail
/routing ospf area print detail  
/routing ospf interface print detail

# Step 3: Check neighbor establishment
/routing ospf neighbor print detail
# Look for states: Down, Init, 2-Way, ExStart, Exchange, Loading, Full

# Step 4: Examine routing information
/ip route print where ospf=yes
/routing ospf lsa print where area=backbone

# Step 5: Check for filtering issues
/routing filter rule print where chain~"ospf"

# Step 6: Monitor logs for errors
/log print where topics~"ospf"

# Troubleshooting script for OSPF
:local targetArea "backbone";
:local expectedNeighbors 3;

/log info ("Troubleshooting OSPF area: " . $targetArea);

# Check area configuration
:local areaExists [/routing ospf area find name=$targetArea];
:if ([:len $areaExists] > 0) do={
    /log info ("Area " . $targetArea . " is configured");
} else={
    /log error ("Area " . $targetArea . " not found");
    :error "Area not configured";
};

# Check neighbor count
:local neighborCount [/routing ospf neighbor print count-only where area=$targetArea];
:if ($neighborCount >= $expectedNeighbors) do={
    /log info ("Area " . $targetArea . " has " . $neighborCount . " neighbors (expected: " . $expectedNeighbors . ")");
} else={
    /log warning ("Area " . $targetArea . " has only " . $neighborCount . " neighbors (expected: " . $expectedNeighbors . ")");
};

# Check for Full state neighbors
:local fullNeighbors [/routing ospf neighbor print count-only where area=$targetArea and state=Full];
/log info ("Full state neighbors: " . $fullNeighbors . "/" . $neighborCount);
```
{% endcode %}

***

## Best practices summary

### Design guidelines

1. **Start with hierarchy** - Design proper area/AS structure from beginning
2. **Plan addressing** - Use structured IP addressing for route summarization  
3. **Choose appropriate protocols** - Match protocol to network requirements
4. **Design for redundancy** - Multiple paths and failover mechanisms
5. **Document thoroughly** - Maintain network diagrams and protocol configurations

### Operational practices

1. **Monitor actively** - Track neighbor states and route convergence
2. **Test failover scenarios** - Regular testing of redundant paths
3. **Maintain consistent configuration** - Standardize protocol parameters
4. **Plan capacity** - Monitor routing table growth and convergence times
5. **Keep software updated** - Use latest RouterOS versions for stability

### Security considerations

1. **Enable authentication** - Use protocol authentication where supported
2. **Filter routes appropriately** - Control route advertisement and acceptance
3. **Monitor for anomalies** - Watch for unexpected routing changes
4. **Secure management** - Protect routing protocol configuration access
5. **Document security policies** - Maintain routing security procedures

