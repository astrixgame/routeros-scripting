---
description: Open Shortest Path First (OSPF) is a link-state routing protocol providing fast convergence, hierarchical design, and optimal path selection for enterprise networks.
icon: map
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

# OSPF - Open Shortest Path First

{% hint style="info" %}
OSPF is a link-state interior gateway protocol that builds a complete topology database and calculates shortest paths using Dijkstra's algorithm, providing fast convergence and load balancing capabilities.
{% endhint %}

RouterOS v7+ includes an enhanced OSPF implementation with improved performance, better scalability, and support for modern OSPF features including graceful restart and traffic engineering extensions.

***

## OSPF fundamentals

### How OSPF works

**Link-state operation:**
- Each router maintains complete network topology database
- Routers exchange Link State Advertisements (LSAs)
- Shortest Path First (SPF) algorithm calculates optimal paths
- Hierarchical area design provides scalability

**Key concepts:**
- **Areas** - Logical subdivisions that limit LSA flooding scope
- **Router ID** - Unique identifier for each OSPF router
- **Cost** - Metric based on bandwidth (100Mbps/interface bandwidth)
- **Designated Router (DR)** - Reduces LSA flooding on broadcast networks
- **Adjacencies** - Full neighbor relationships for LSA synchronization

### OSPF advantages

**Fast convergence:**
- Sub-second convergence with proper tuning
- Immediate detection of link failures
- Incremental SPF calculations for efficiency

**Scalability features:**
- Area hierarchy reduces routing overhead
- Route summarization at area boundaries
- Support for thousands of routes

**Load balancing:**
- Equal-Cost Multi-Path (ECMP) support
- Automatic load distribution across equal paths
- Unequal cost load balancing with careful design

{% code overflow="wrap" %}
```bash
# OSPF network topology example
#
#     Area 0 (Backbone)
#    Router1 ---- Router2
#       |            |
#   Area 1       Area 2
#  (Marketing)  (Engineering)
#
# Each area maintains separate LSA database
# Area 0 connects all other areas
# Inter-area routing through Area 0 (ABR routers)
```
{% endcode %}

***

## Basic OSPF configuration

### Single area OSPF

Simple OSPF deployment for small to medium networks:

{% code overflow="wrap" %}
```bash
# Basic OSPF configuration for single area
# Enable OSPF instance
/routing ospf instance add name=main router-id=1.1.1.1 disabled=no comment="Main OSPF instance"

# Create backbone area (Area 0)
/routing ospf area add name=backbone area-id=0.0.0.0 instance=main comment="OSPF backbone area"

# Add interfaces to OSPF
/routing ospf interface-template add area=backbone interfaces=ether1,ether2,bridge \
    type=broadcast disabled=no comment="OSPF interfaces"

# Verify OSPF operation
/routing ospf neighbor print
/routing ospf lsa print
/ip route print where ospf=yes

# Check OSPF database
/routing ospf lsa print where area=backbone
```
{% endcode %}

### Multi-area OSPF

Hierarchical OSPF design for larger networks:

{% code overflow="wrap" %}
```bash
# Multi-area OSPF configuration
# Router acting as Area Border Router (ABR)

# OSPF instance with unique router ID
/routing ospf instance add name=enterprise router-id=10.1.1.1 disabled=no \
    comment="Enterprise OSPF"

# Backbone area (Area 0) - mandatory for multi-area OSPF
/routing ospf area add name=backbone area-id=0.0.0.0 instance=enterprise \
    comment="OSPF backbone - connects all areas"

# Regular areas connected to backbone
/routing ospf area add name=sales area-id=0.0.0.1 instance=enterprise \
    comment="Sales department area"

/routing ospf area add name=engineering area-id=0.0.0.2 instance=enterprise \
    comment="Engineering department area"

# Interface assignments
# Backbone connections (core network)
/routing ospf interface-template add area=backbone interfaces=ether1,ether2 \
    type=ptp cost=10 disabled=no comment="Backbone area interfaces"

# Sales area interfaces
/routing ospf interface-template add area=sales interfaces=ether3 \
    type=broadcast cost=100 disabled=no comment="Sales area interface"

# Engineering area interfaces  
/routing ospf interface-template add area=engineering interfaces=ether4 \
    type=broadcast cost=100 disabled=no comment="Engineering area interface"

# Verify multi-area operation
/routing ospf area print
/routing ospf neighbor print
/routing ospf lsa print where type=summary  # Inter-area routes
```
{% endcode %}

***

## Advanced OSPF features

### OSPF area types

Different area types for optimization:

{% code overflow="wrap" %}
```bash
# Standard area (default) - accepts all LSA types
/routing ospf area add name=standard area-id=0.0.0.10 instance=main \
    type=default comment="Standard area - all LSA types"

# Stub area - blocks external LSAs, reduces routing table
/routing ospf area add name=branch-offices area-id=0.0.0.20 instance=main \
    type=stub default-cost=100 comment="Stub area for branch offices"

# Totally stubby area - blocks external and summary LSAs
/routing ospf area add name=access-layer area-id=0.0.0.30 instance=main \
    type=stub no-summaries=yes default-cost=50 \
    comment="Totally stubby area for access layer"

# Not-So-Stubby Area (NSSA) - allows limited external routes
/routing ospf area add name=regional area-id=0.0.0.40 instance=main \
    type=nssa default-cost=200 comment="NSSA for regional offices"

# Interface assignments for different area types
/routing ospf interface-template add area=branch-offices interfaces=ether5 \
    type=broadcast disabled=no comment="Branch office stub area"

/routing ospf interface-template add area=access-layer interfaces=ether6 \
    type=broadcast disabled=no comment="Access layer totally stubby"

# Verify area types and their effect on LSA database
/routing ospf area print detail
/routing ospf lsa print where area=branch-offices  # Should show fewer LSAs
/routing ospf lsa print where area=access-layer   # Should show minimal LSAs
```
{% endcode %}

### OSPF authentication

Secure OSPF communications:

{% code overflow="wrap" %}
```bash
# Area-wide authentication
/routing ospf area add name=secure-area area-id=0.0.1.0 instance=main \
    auth-type=md5 comment="Area with MD5 authentication"

# Interface-specific authentication (overrides area setting)
/routing ospf interface-template add area=secure-area interfaces=ether3 \
    type=ptp auth-type=md5 auth-key="SecureOSPFKey123!" auth-id=1 \
    disabled=no comment="Secure OSPF interface"

# Simple password authentication (less secure)
/routing ospf interface-template add area=backbone interfaces=ether7 \
    type=broadcast auth-type=simple auth-key="SimplePassword" \
    disabled=no comment="Simple authentication interface"

# Verify authentication is working
/routing ospf neighbor print detail  # Should show authenticated neighbors
/log print where topics~"ospf" and message~"auth"  # Check for auth errors
```
{% endcode %}

### OSPF route filtering and summarization

Control route advertisement and summarization:

{% code overflow="wrap" %}
```bash
# Route filtering for OSPF
# Filter specific networks from OSPF advertisement
/routing filter rule add chain=ospf-out action=discard \
    prefix=192.168.100.0/24 comment="Block test network from OSPF"

# Allow only specific networks  
/routing filter rule add chain=ospf-out action=accept \
    prefix=10.0.0.0/8 prefix-length=8-24 comment="Allow corporate networks"

/routing filter rule add chain=ospf-out action=discard \
    comment="Block all other networks"

# Route summarization at area borders
/routing ospf area add name=branch-summary area-id=0.0.2.0 instance=main \
    area-range=192.168.0.0/16 advertise=yes comment="Summarize branch networks"

# Multiple summarization ranges for different network segments
/routing ospf area set branch-summary \
    area-range=192.168.0.0/16,10.100.0.0/16 \
    comment="Multiple summary ranges"

# External route summarization (for redistributed routes)
/routing ospf instance set main \
    asbr-summary-lsa=yes comment="Enable ASBR summary LSAs"

# Verify summarization is working
/routing ospf lsa print where type=summary  # Check summary LSAs
/ip route print where ospf=yes  # Verify summarized routes are installed
```
{% endcode %}

***

## OSPF network types

### Point-to-Point networks

Direct connections between two routers:

{% code overflow="wrap" %}
```bash
# Point-to-Point configuration (WAN links, dedicated connections)
/routing ospf interface-template add area=backbone interfaces=ether1 \
    type=ptp cost=100 hello-interval=10s dead-interval=30s \
    disabled=no comment="P2P WAN link"

# P2P characteristics:
# - No DR/BDR election needed
# - Faster convergence
# - Lower overhead
# - Suitable for WAN links

# Verify P2P operation
/routing ospf interface print where type=ptp
/routing ospf neighbor print  # Should show P2P neighbors without DR election
```
{% endcode %}

### Broadcast networks

Ethernet LANs with multiple routers:

{% code overflow="wrap" %}
```bash
# Broadcast network configuration (Ethernet LANs)
/routing ospf interface-template add area=backbone interfaces=bridge \
    type=broadcast cost=10 priority=100 hello-interval=10s dead-interval=40s \
    disabled=no comment="LAN broadcast network"

# Broadcast characteristics:
# - DR/BDR election for efficiency
# - Higher priority routers become DR
# - All routers form adjacency with DR/BDR
# - Reduces LSA flooding on multi-access networks

# Control DR election with priority
/routing ospf interface-template set [find interfaces=bridge] priority=200
# Priority 0 = never DR, higher values preferred for DR election

# Verify DR/BDR election
/routing ospf neighbor print  # Check DR/BDR status
/routing ospf interface print detail  # Show DR/BDR for each interface
```
{% endcode %}

### NBMA and Point-to-Multipoint

Non-broadcast networks (Frame Relay, etc.):

{% code overflow="wrap" %}
```bash
# NBMA (Non-Broadcast Multiple Access) configuration
/routing ospf interface-template add area=backbone interfaces=ether2 \
    type=nbma cost=200 hello-interval=30s dead-interval=120s \
    disabled=no comment="NBMA network (Frame Relay)"

# Manually configure NBMA neighbors (no broadcast capability)
/routing ospf nbma-neighbor add interface=ether2 address=10.1.1.2 \
    priority=1 comment="NBMA neighbor 1"

/routing ospf nbma-neighbor add interface=ether2 address=10.1.1.3 \
    priority=1 comment="NBMA neighbor 2"

# Point-to-Multipoint (alternative to NBMA)
/routing ospf interface-template add area=backbone interfaces=ether3 \
    type=ptmp cost=150 hello-interval=30s dead-interval=120s \
    disabled=no comment="Point-to-multipoint network"

# P2MP characteristics:
# - No DR/BDR election
# - Automatic neighbor discovery
# - More resilient than NBMA
# - Better for partial mesh topologies
```
{% endcode %}

***

## OSPF performance tuning

### Convergence optimization

Tune OSPF for faster convergence:

{% code overflow="wrap" %}
```bash
# Fast convergence tuning
# Reduce hello and dead intervals for faster failure detection
/routing ospf interface-template set [find area=backbone] \
    hello-interval=1s \
    dead-interval=3s \
    retransmit-interval=1s \
    comment="Fast convergence timers"

# Tune SPF calculation timers
/routing ospf area set backbone \
    spf-delay=200ms \
    spf-hold-time=1s \
    spf-max-hold-time=5s \
    comment="Fast SPF calculation"

# LSA generation throttling
/routing ospf area set backbone \
    lsa-min-interval=1s \
    comment="Reduce LSA generation delay"

# Enable BFD (Bidirectional Forwarding Detection) for sub-second detection
/routing ospf interface-template set [find area=backbone] \
    bfd=yes comment="Enable BFD for fast failure detection"

# Configure BFD parameters
/routing bfd interface add interface=ether1 interval=100ms multiplier=3 \
    comment="BFD for OSPF fast convergence"
```
{% endcode %}

### Scalability optimization

Configure OSPF for large-scale deployments:

{% code overflow="wrap" %}
```bash
# Scalability improvements
# Use area summarization to reduce routing table size
/routing ospf area set engineering \
    area-range=10.20.0.0/16 advertise=yes \
    comment="Summarize engineering networks"

# Implement stub areas to reduce LSA flooding
/routing ospf area set access-networks type=stub \
    default-cost=100 comment="Stub area reduces LSA count"

# Tune LSA refresh and aging
/routing ospf area set backbone \
    lsa-refresh-time=1800s \
    lsa-max-age=3600s \
    comment="Optimize LSA refresh timers"

# Control external route advertisement
/routing ospf instance set main \
    redistribute=connected,static \
    metric-default=20 \
    metric-type=2 \
    comment="Control external route redistribution"

# Use route filtering to limit route propagation
/routing filter rule add chain=ospf-out action=discard \
    prefix=169.254.0.0/16 comment="Block link-local addresses"

/routing filter rule add chain=ospf-out action=discard \
    prefix=0.0.0.0/0 prefix-length=0-7 comment="Block too-general prefixes"
```
{% endcode %}

***

## OSPF monitoring and troubleshooting

### Monitoring OSPF health

Track OSPF performance and status:

{% code overflow="wrap" %}
```bash
# Basic OSPF monitoring
/routing ospf neighbor print  # Neighbor states and timers
/routing ospf interface print  # Interface status and costs
/routing ospf area print  # Area information and LSA counts
/routing ospf lsa print  # LSA database contents

# Detailed neighbor information
/routing ospf neighbor print detail  # Full neighbor details
/routing ospf neighbor monitor [find router-id=2.2.2.2]  # Monitor specific neighbor

# LSA database analysis
/routing ospf lsa print where type=router  # Router LSAs (topology)
/routing ospf lsa print where type=network  # Network LSAs (broadcast segments)
/routing ospf lsa print where type=summary  # Summary LSAs (inter-area routes)
/routing ospf lsa print where type=external  # External LSAs (redistributed routes)

# Route table verification
/ip route print where ospf=yes  # All OSPF routes
/ip route print where ospf=yes and distance=110  # Standard OSPF routes
/ip route print stats where ospf=yes  # Route usage statistics

# Interface statistics
/routing ospf interface print stats  # OSPF packet counters
/interface print stats where name~"ether"  # Physical interface stats
```
{% endcode %}

### Troubleshooting procedures

Systematic OSPF troubleshooting approach:

{% code overflow="wrap" %}
```bash
# Step 1: Check basic connectivity
/ping neighbor-ip count=5  # Test IP connectivity to neighbors

# Step 2: Verify OSPF configuration
/routing ospf instance print detail  # Instance configuration
/routing ospf area print detail  # Area configuration and ranges
/routing ospf interface print detail  # Interface parameters

# Step 3: Check neighbor establishment
/routing ospf neighbor print detail
# Neighbor states: Down -> Init -> 2-Way -> ExStart -> Exchange -> Loading -> Full
# Stuck states indicate specific problems

# Step 4: Examine LSA database synchronization
/routing ospf lsa print where area=backbone count-only  # LSA count per area
/routing ospf lsa print where router-id=problem-router  # LSAs from specific router

# Step 5: Check route installation
/ip route print where ospf=yes and dst-address~"problem-network"

# Step 6: Monitor OSPF logs
/log print where topics~"ospf"  # OSPF-related log messages
/system logging add topics=ospf action=memory  # Enable OSPF logging

# Common problems and solutions:
# 1. Neighbor stuck in Init state - Check area ID match
# 2. Neighbor stuck in 2-Way state - Check DR election process  
# 3. Neighbor stuck in ExStart - Check authentication
# 4. Missing routes - Check area types and filtering
# 5. Slow convergence - Check timers and BFD configuration

# OSPF diagnostic script
:local checkArea "backbone";
:local expectedNeighbors 3;

/log info ("OSPF Diagnostic for area: " . $checkArea);

# Check area configuration
:local areaConfig [/routing ospf area find name=$checkArea];
:if ([:len $areaConfig] > 0) do={
    :local areaInfo [/routing ospf area get $areaConfig];
    /log info ("Area " . $checkArea . " type: " . ($areaInfo->"type"));
} else={
    /log error ("Area " . $checkArea . " not configured");
};

# Check neighbor states
:local neighbors [/routing ospf neighbor find area=$checkArea];
:local fullNeighbors 0;
:foreach neighbor in=$neighbors do={
    :local neighborInfo [/routing ospf neighbor get $neighbor];
    :local state ($neighborInfo->"state");
    :local routerId ($neighborInfo->"router-id");
    
    /log info ("Neighbor " . $routerId . " state: " . $state);
    
    :if ($state = "Full") do={
        :set fullNeighbors ($fullNeighbors + 1);
    };
};

/log info ("Full neighbors: " . $fullNeighbors . "/" . [:len $neighbors] . " (expected: " . $expectedNeighbors . ")");

# Check LSA database health
:local lsaCount [/routing ospf lsa print count-only where area=$checkArea];
/log info ("LSA count in " . $checkArea . ": " . $lsaCount);

# Check route installation
:local ospfRoutes [/ip route print count-only where ospf=yes];
/log info ("OSPF routes installed: " . $ospfRoutes);
```
{% endcode %}

***

## OSPF design best practices

### Network design principles

1. **Hierarchical design** - Always use Area 0 as backbone, connect other areas to it
2. **Area sizing** - Keep areas under 50 routers for optimal performance
3. **Router ID planning** - Use loopback addresses or planned IDs for stability
4. **Cost planning** - Design consistent cost metrics for predictable paths
5. **Redundancy design** - Multiple ABRs between areas for resilience

### Configuration guidelines

1. **Authentication** - Always enable OSPF authentication in production
2. **Timer consistency** - Ensure consistent hello/dead intervals per network
3. **Area types** - Use stub areas to reduce routing overhead where appropriate
4. **Summarization** - Implement route summarization at area boundaries
5. **Filtering** - Control route advertisement with appropriate filters

### Operational practices

1. **Monitor actively** - Track neighbor states and convergence times
2. **Document topology** - Maintain current network diagrams and area designs  
3. **Test changes** - Verify OSPF behavior after configuration changes
4. **Capacity planning** - Monitor LSA database growth and CPU utilization
5. **Backup configuration** - Regular backups of OSPF configuration

***

## Complete OSPF examples

### Enterprise campus network

{% code overflow="wrap" %}
```bash
# Complete enterprise OSPF deployment
# Multi-area design with backbone, distribution, and access areas

# Main OSPF instance
/routing ospf instance add name=campus router-id=1.1.1.1 disabled=no \
    redistribute=connected metric-default=20 comment="Campus OSPF"

# Area hierarchy
# Backbone area (Area 0) - Core network
/routing ospf area add name=backbone area-id=0.0.0.0 instance=campus \
    comment="Campus backbone - core switches"

# Distribution areas - Building/department networks
/routing ospf area add name=admin-building area-id=0.0.0.1 instance=campus \
    area-range=10.10.0.0/16 advertise=yes comment="Administration building"

/routing ospf area add name=engineering area-id=0.0.0.2 instance=campus \
    area-range=10.20.0.0/16 advertise=yes comment="Engineering building"

/routing ospf area add name=manufacturing area-id=0.0.0.3 instance=campus \
    type=stub default-cost=100 comment="Manufacturing - stub area"

# Access areas - Floor/subnet networks  
/routing ospf area add name=student-access area-id=0.0.1.0 instance=campus \
    type=stub default-cost=50 comment="Student access - stub area"

# Interface assignments
# Backbone interfaces (core network)
/routing ospf interface-template add area=backbone interfaces=ether1,ether2 \
    type=ptp cost=10 hello-interval=5s dead-interval=15s \
    auth-type=md5 auth-key="CampusOSPF2024!" auth-id=1 \
    disabled=no comment="Backbone core links"

# Distribution interfaces
/routing ospf interface-template add area=admin-building interfaces=ether3 \
    type=broadcast cost=100 priority=200 \
    auth-type=md5 auth-key="CampusOSPF2024!" auth-id=1 \
    disabled=no comment="Admin building distribution"

/routing ospf interface-template add area=engineering interfaces=ether4 \
    type=broadcast cost=100 priority=150 \
    auth-type=md5 auth-key="CampusOSPF2024!" auth-id=1 \
    disabled=no comment="Engineering distribution"

# Access interfaces (stub areas)
/routing ospf interface-template add area=manufacturing interfaces=ether5 \
    type=broadcast cost=1000 \
    auth-type=md5 auth-key="CampusOSPF2024!" auth-id=1 \
    disabled=no comment="Manufacturing access"

/routing ospf interface-template add area=student-access interfaces=ether6 \
    type=broadcast cost=1000 \
    auth-type=md5 auth-key="CampusOSPF2024!" auth-id=1 \
    disabled=no comment="Student access"

# Route filtering for security and efficiency
/routing filter rule add chain=ospf-out action=discard \
    prefix=169.254.0.0/16 comment="Block APIPA addresses"

/routing filter rule add chain=ospf-out action=discard \
    prefix=224.0.0.0/4 comment="Block multicast addresses"

/routing filter rule add chain=ospf-in action=discard \
    prefix=0.0.0.0/0 prefix-length=0-7 comment="Block too-general prefixes"

# External route redistribution (for internet access)
/routing filter rule add chain=ospf-out action=accept \
    prefix=0.0.0.0/0 set-distance=1 comment="Advertise default route"

# QoS integration - prioritize OSPF traffic
/ip firewall mangle add chain=prerouting protocol=ospf \
    action=set-priority new-priority=2 comment="Prioritize OSPF packets"

# BFD for fast convergence on critical links
/routing bfd interface add interface=ether1,ether2 interval=200ms \
    multiplier=3 comment="BFD for backbone links"

/routing ospf interface-template set [find area=backbone] bfd=yes

# Monitoring and health checks
/tool netwatch add host=1.1.1.2 timeout=1s interval=5s \
    comment="Monitor backbone peer"

/tool netwatch add host=10.10.1.1 timeout=2s interval=10s \
    comment="Monitor admin building"

# Logging configuration
/system logging add topics=ospf,bfd action=memory

# Verification and monitoring commands
/routing ospf neighbor print  # Check all neighbor relationships
/routing ospf area print  # Verify area configuration
/routing ospf lsa print count-only  # Monitor LSA database size
/ip route print where ospf=yes  # Verify route installation
/routing ospf interface print stats  # Monitor OSPF packet counters

# Performance monitoring script
:local areas {"backbone"; "admin-building"; "engineering"; "manufacturing"; "student-access"};

:foreach area in=$areas do={
    :local neighborCount [/routing ospf neighbor print count-only where area=$area];
    :local fullNeighbors [/routing ospf neighbor print count-only where area=$area and state="Full"];
    :local lsaCount [/routing ospf lsa print count-only where area=$area];
    
    /log info ("Area " . $area . ": " . $fullNeighbors . "/" . $neighborCount . " neighbors Full, " . $lsaCount . " LSAs");
};
```
{% endcode %}

