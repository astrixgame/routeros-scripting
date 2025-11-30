---
description: Multiprotocol Label Switching (MPLS) provides efficient packet forwarding, traffic engineering, and VPN services through label-based switching technology.
icon: tags
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

# MPLS - Multiprotocol Label Switching

{% hint style="info" %}
MPLS enables high-performance packet forwarding using labels instead of IP lookups, providing the foundation for advanced services like Layer 3 VPNs, traffic engineering, and Quality of Service.
{% endhint %}

RouterOS supports MPLS with LDP (Label Distribution Protocol), static labels, and MPLS VPN capabilities for service provider and enterprise networks requiring advanced routing and VPN services.

***

## MPLS fundamentals

### How MPLS works

**Label switching concepts:**
- **Labels** - Short fixed-length identifiers attached to packets
- **Label Switch Routers (LSR)** - Forward packets based on labels
- **Label Edge Routers (LER)** - Add/remove labels at network edge
- **Forwarding Equivalence Class (FEC)** - Group of packets with same forwarding treatment
- **Label Switch Paths (LSP)** - Path that packets with particular label follow

**MPLS advantages:**
- **High performance** - Simple label lookup vs complex IP routing
- **Traffic engineering** - Explicit path control for optimal resource usage
- **VPN services** - Isolated routing domains with label separation
- **QoS support** - Built-in quality of service capabilities
- **Protocol independence** - Works with any Layer 3 protocol

### MPLS architecture

{% code overflow="wrap" %}
```bash
# MPLS network components:

# 1. Core MPLS network (P routers)
#    - Pure label switching
#    - No knowledge of VPN routes
#    - High-speed forwarding

# 2. Provider Edge (PE) routers  
#    - Add/remove labels
#    - VPN route processing
#    - Customer interface

# 3. Customer Edge (CE) routers
#    - No MPLS knowledge
#    - Standard IP routing
#    - Connected to PE routers

# Label operations:
# PUSH - Add label (ingress LER)
# SWAP - Change label (transit LSR) 
# POP - Remove label (egress LER)
```
{% endcode %}

***

## Basic MPLS configuration

### Enable MPLS interfaces

Configure interfaces for MPLS forwarding:

{% code overflow="wrap" %}
```bash
# Enable MPLS on core interfaces
/mpls interface add interface=ether2 disabled=no comment="Core link to PE2"
/mpls interface add interface=ether3 disabled=no comment="Core link to PE3"
/mpls interface add interface=ether4 disabled=no comment="Core link to P1"

# Verify MPLS interface configuration
/mpls interface print

# Check MPLS interface status
/mpls interface monitor [find interface=ether2]
```
{% endcode %}

### LDP configuration

Label Distribution Protocol for automatic label distribution:

{% code overflow="wrap" %}
```bash
# Enable LDP globally
/mpls ldp set enabled=yes lsr-id=1.1.1.1

# Configure LDP interfaces
/mpls ldp interface add interface=ether2 disabled=no comment="LDP to PE2"
/mpls ldp interface add interface=ether3 disabled=no comment="LDP to PE3"
/mpls ldp interface add interface=ether4 disabled=no comment="LDP to P1"

# Verify LDP neighbors
/mpls ldp neighbor print

# Check LDP database
/mpls ldp local-mapping print
/mpls ldp remote-mapping print

# Monitor LDP sessions
/mpls ldp neighbor monitor [find peer=2.2.2.2]
```
{% endcode %}

### Static label configuration

Manual label configuration for specific scenarios:

{% code overflow="wrap" %}
```bash
# Static incoming label mapping
/mpls static-bind ingress add dst-address=192.168.10.0/24 \
    label=100 interface=ether5 comment="Static label for branch network"

# Static outgoing label mapping  
/mpls static-bind egress add dst-address=192.168.20.0/24 \
    nexthop-address=10.0.0.2 label=200 interface=ether2 \
    comment="Static egress label"

# Verify static bindings
/mpls static-bind ingress print
/mpls static-bind egress print
```
{% endcode %}

***

## MPLS VPN configuration

### Layer 3 VPN (BGP/MPLS VPN)

Configure MPLS L3VPN for customer isolation:

{% code overflow="wrap" %}
```bash
# Step 1: Configure VRF (Virtual Routing and Forwarding)
/ip route vrf add interfaces=ether5 route-distinguisher=65001:100 \
    import-route-targets=65001:100 export-route-targets=65001:100 \
    routing-mark=customer-a comment="Customer A VPN"

# Step 2: Configure BGP for VPNv4 routes
/routing bgp template add name=mpls-vpn as=65001 \
    router-id=1.1.1.1 disabled=no

# BGP neighbor for VPNv4 route exchange
/routing bgp connection add template=mpls-vpn remote.address=2.2.2.2 \
    remote.as=65001 address-families=vpnv4 disabled=no \
    comment="iBGP peer for VPN routes"

# Step 3: Configure customer interface
/ip address add address=192.168.100.1/24 interface=ether5 comment="Customer A interface"

# Step 4: Configure customer routing (static or dynamic)
/ip route add dst-address=10.100.0.0/16 gateway=192.168.100.2 \
    routing-table=customer-a comment="Customer A internal networks"

# Verify VPN configuration
/ip route vrf print
/routing bgp session print
/ip route print where routing-table=customer-a
```
{% endcode %}

### Multi-site VPN connectivity

Connect multiple customer sites through MPLS backbone:

{% code overflow="wrap" %}
```bash
# PE1 configuration (Site A)
/ip route vrf add interfaces=ether5 route-distinguisher=65001:100 \
    import-route-targets=65001:100,65001:200 \
    export-route-targets=65001:100 \
    routing-mark=site-a comment="Site A VPN"

# PE2 configuration (Site B)  
/ip route vrf add interfaces=ether6 route-distinguisher=65001:200 \
    import-route-targets=65001:100,65001:200 \
    export-route-targets=65001:200 \
    routing-mark=site-b comment="Site B VPN"

# Configure inter-site connectivity through route targets
# Sites can communicate because they import each other's route targets

# Verify inter-site reachability
/ip route print where routing-table=site-a
/ip route print where routing-table=site-b
```
{% endcode %}

***

## MPLS traffic engineering

### Explicit path configuration

Control traffic paths through the MPLS network:

{% code overflow="wrap" %}
```bash
# Define explicit path for traffic engineering
/mpls traffic-eng path add name=primary-path \
    hops=10.0.0.2,10.0.0.6,10.0.0.10 comment="Primary path via core routers"

/mpls traffic-eng path add name=backup-path \
    hops=10.0.0.3,10.0.0.7,10.0.0.10 comment="Backup path for failover"

# Create traffic-engineered tunnel
/mpls traffic-eng tunnel add name=te-tunnel1 \
    from-address=1.1.1.1 to-address=4.4.4.4 \
    primary-path=primary-path backup-path=backup-path \
    bandwidth=100M disabled=no comment="TE tunnel to remote site"

# Verify tunnel status
/mpls traffic-eng tunnel print
/mpls traffic-eng tunnel monitor te-tunnel1
```
{% endcode %}

### Bandwidth management

Configure bandwidth constraints and reservations:

{% code overflow="wrap" %}
```bash
# Configure interface bandwidth limits for TE
/mpls traffic-eng interface add interface=ether2 \
    bandwidth=1G reserved-bandwidth=0 disabled=no \
    comment="TE interface with 1Gbps capacity"

# Create tunnel with bandwidth reservation
/mpls traffic-eng tunnel add name=voice-tunnel \
    from-address=1.1.1.1 to-address=2.2.2.2 \
    bandwidth=10M priority=1 \
    comment="Voice traffic tunnel with guaranteed bandwidth"

# Monitor bandwidth utilization
/mpls traffic-eng interface monitor [find interface=ether2]
/mpls traffic-eng tunnel monitor voice-tunnel
```
{% endcode %}

***

## MPLS QoS configuration

### EXP bit marking

Quality of Service using MPLS EXP bits:

{% code overflow="wrap" %}
```bash
# Mark packets with EXP bits based on traffic type
/ip firewall mangle add chain=prerouting protocol=udp dst-port=5060 \
    action=set-priority new-priority=5 comment="VoIP signaling - high priority"

/ip firewall mangle add chain=prerouting protocol=udp dst-port=10000-20000 \
    action=set-priority new-priority=5 comment="VoIP RTP - high priority"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80,443 \
    action=set-priority new-priority=3 comment="Web traffic - normal priority"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=21,22 \
    action=set-priority new-priority=1 comment="File transfers - low priority"

# Configure queue trees based on priorities
/queue tree add name=voice-queue parent=ether2 priority=1 \
    max-limit=50M burst-limit=60M comment="Voice traffic queue"

/queue tree add name=data-queue parent=ether2 priority=8 \
    max-limit=100M comment="Data traffic queue"
```
{% endcode %}

### Per-VPN QoS policies

Different QoS policies for different VPN customers:

{% code overflow="wrap" %}
```bash
# Customer A - Premium service with guaranteed bandwidth
/queue tree add name=customer-a-queue parent=ether5 \
    max-limit=100M guaranteed-rate=50M comment="Customer A - Premium"

# Customer B - Standard service with best effort
/queue tree add name=customer-b-queue parent=ether6 \
    max-limit=50M comment="Customer B - Standard"

# Apply QoS based on VRF/routing table
/ip firewall mangle add chain=prerouting routing-mark=customer-a \
    action=set-priority new-priority=3 comment="Customer A traffic priority"

/ip firewall mangle add chain=prerouting routing-mark=customer-b \
    action=set-priority new-priority=5 comment="Customer B traffic priority"
```
{% endcode %}

***

## MPLS monitoring and troubleshooting

### Monitoring MPLS operations

Track MPLS performance and status:

{% code overflow="wrap" %}
```bash
# Monitor LDP neighbors and sessions
/mpls ldp neighbor print
/mpls ldp neighbor monitor [find peer=2.2.2.2]

# Check label forwarding table
/mpls forwarding-table print

# Monitor label bindings
/mpls ldp local-mapping print where prefix=192.168.10.0/24
/mpls ldp remote-mapping print where prefix=192.168.10.0/24

# VPN route monitoring
/ip route print where routing-table=customer-a
/routing bgp advertisements print where peer=2.2.2.2

# Traffic engineering monitoring
/mpls traffic-eng tunnel print
/mpls traffic-eng tunnel monitor te-tunnel1

# Interface statistics
/mpls interface print stats
/interface print stats where name~"ether"
```
{% endcode %}

### Troubleshooting MPLS issues

Common problems and diagnostic steps:

{% code overflow="wrap" %}
```bash
# 1. Check MPLS interface configuration
/mpls interface print
/mpls interface monitor [find interface=ether2]

# 2. Verify LDP neighbor establishment
/mpls ldp neighbor print detail
/log print where topics~"ldp"

# 3. Check label database consistency
/mpls ldp local-mapping print
/mpls ldp remote-mapping print where peer=2.2.2.2

# 4. Test MPLS connectivity with ping
/ping 2.2.2.2 mpls-reply=yes

# 5. Trace MPLS path
/tool traceroute 192.168.10.1 use-dns=no

# 6. Check VPN route propagation
/routing bgp advertisements print where peer=2.2.2.2
/routing bgp session print detail

# 7. Verify VRF configuration
/ip route vrf print detail
/ip route print where routing-table=customer-a

# 8. Monitor for label allocation issues
/log print where topics~"mpls,ldp"

# Debug script for MPLS connectivity
:local testDestination "192.168.10.1";
:local peerAddress "2.2.2.2";

/log info ("Testing MPLS connectivity to " . $testDestination);

# Check if LDP peer is up
:local ldpNeighbor [/mpls ldp neighbor find peer=$peerAddress];
:if ([:len $ldpNeighbor] > 0) do={
    /log info ("LDP neighbor " . $peerAddress . " is active");
} else={
    /log error ("LDP neighbor " . $peerAddress . " is down");
};

# Test ping through MPLS
:do {
    /ping $testDestination count=3;
    /log info ("Ping to " . $testDestination . " successful");
} on-error={
    /log error ("Ping to " . $testDestination . " failed");
};
```
{% endcode %}

***

## MPLS best practices

### Design considerations

1. **Network planning** - Design hierarchical MPLS topology
2. **Label management** - Plan label distribution and allocation
3. **Route reflectors** - Use RR for BGP scalability in large networks
4. **Redundancy** - Implement multiple paths and failover mechanisms
5. **Security** - Secure LDP sessions and PE-CE connections

### Performance optimization

1. **Hardware acceleration** - Use MPLS-capable hardware when available
2. **Label stack optimization** - Minimize label stack depth
3. **BGP optimization** - Tune BGP parameters for VPN scalability
4. **Interface tuning** - Optimize MPLS interface parameters
5. **Monitoring** - Implement comprehensive MPLS monitoring

### Security guidelines

1. **PE-CE security** - Secure customer connections and routing
2. **LDP authentication** - Enable LDP session authentication  
3. **Access control** - Limit administrative access to MPLS configuration
4. **Route filtering** - Implement proper VPN route filtering
5. **Audit trails** - Monitor and log MPLS configuration changes

***

## Complete MPLS example

### Service provider MPLS network

{% code overflow="wrap" %}
```bash
# Complete MPLS service provider configuration
# PE router configuration for multiple customers

# 1. Basic MPLS and LDP setup
/mpls interface add interface=ether1 disabled=no comment="Core link 1"
/mpls interface add interface=ether2 disabled=no comment="Core link 2"

/mpls ldp set enabled=yes lsr-id=1.1.1.1
/mpls ldp interface add interface=ether1,ether2 disabled=no

# 2. BGP configuration for VPNv4
/routing bgp template add name=mpls-pe as=65000 router-id=1.1.1.1 disabled=no

/routing bgp connection add template=mpls-pe remote.address=2.2.2.2 \
    remote.as=65000 address-families=vpnv4 disabled=no comment="PE2"

/routing bgp connection add template=mpls-pe remote.address=3.3.3.3 \
    remote.as=65000 address-families=vpnv4 disabled=no comment="PE3"

# 3. Customer A VPN (Finance)
/ip route vrf add interfaces=ether3 route-distinguisher=65000:100 \
    import-route-targets=65000:100 export-route-targets=65000:100 \
    routing-mark=finance-vpn comment="Finance VPN"

/ip address add address=192.168.100.1/30 interface=ether3 comment="Finance CE link"

# Static routes for Finance customer
/ip route add dst-address=10.100.0.0/16 gateway=192.168.100.2 \
    routing-table=finance-vpn comment="Finance internal networks"

# 4. Customer B VPN (Manufacturing)
/ip route vrf add interfaces=ether4 route-distinguisher=65000:200 \
    import-route-targets=65000:200 export-route-targets=65000:200 \
    routing-mark=manufacturing-vpn comment="Manufacturing VPN"

/ip address add address=192.168.200.1/30 interface=ether4 comment="Manufacturing CE link"

# OSPF with Manufacturing customer
/routing ospf instance add name=manufacturing router-id=1.1.1.1 \
    vrf=manufacturing-vpn disabled=no

/routing ospf area add name=customer-area area-id=0.0.0.0 instance=manufacturing

/routing ospf interface-template add area=customer-area interfaces=ether4 \
    type=ptp disabled=no

# 5. Inter-site connectivity for Customer A
# Additional sites import/export same route targets automatically

# 6. QoS configuration
/ip firewall mangle add chain=prerouting in-interface=ether3 \
    protocol=udp dst-port=5060 action=set-priority new-priority=1 \
    comment="Finance VoIP priority"

/queue tree add name=finance-voice parent=ether3 priority=1 \
    max-limit=10M comment="Finance voice queue"

/queue tree add name=finance-data parent=ether3 priority=8 \
    max-limit=50M comment="Finance data queue"

# 7. Monitoring and health checks
/tool netwatch add host=2.2.2.2 timeout=2s interval=10s comment="Monitor PE2"
/tool netwatch add host=192.168.100.2 timeout=1s interval=5s comment="Finance CE"
/tool netwatch add host=192.168.200.2 timeout=1s interval=5s comment="Manufacturing CE"

# 8. Logging for troubleshooting
/system logging add topics=mpls,ldp,bgp action=memory

# Verification commands
/mpls ldp neighbor print
/routing bgp session print
/ip route vrf print
/ip route print where routing-table=finance-vpn
/ip route print where routing-table=manufacturing-vpn
```
{% endcode %}

