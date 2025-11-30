---
description: Overview of all interface types available in RouterOS for network connectivity and virtualization.
icon: network-wired
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Interfaces

RouterOS supports a wide variety of interface types for different networking scenarios, from basic Ethernet connections to advanced virtualization and wireless technologies.

***

## Physical interfaces

### Ethernet interfaces
- **Standard Ethernet** - Basic wired network connectivity
- **SFP/SFP+** - Fiber optic connections for high-speed links
- **Switch chip** - Hardware-accelerated switching on supported devices

### Wireless interfaces
- **WiFi** - 802.11 wireless networking (a/b/g/n/ac/ax)
- **LTE** - Cellular connectivity for internet access and backup

***

## Virtual interfaces

### Layer 2 virtualization
- **[Bridge](bridge.md)** - Connect multiple interfaces at Layer 2
  - VLAN filtering and RSTP/MSTP support
  - Hardware offloading capabilities
  - IGMP snooping and multicast control
- **[VLAN](vlan.md)** - 802.1Q VLAN tagging for network segmentation
  - Inter-VLAN routing and switching
  - VLAN trunk and access port configuration
  - QinQ double tagging support
- **[VXLAN](vxlan.md)** - Layer 2 overlay networks over Layer 3 infrastructure
  - 24-bit VNI namespace for massive scale
  - Multicast and unicast replication modes
  - EVPN integration for advanced deployments

### Layer 3 tunneling
- **[PPPoE](pppoe.md)** - Point-to-Point Protocol over Ethernet for ISP connections
- **[GRE](gre.md)** - Generic Routing Encapsulation tunnels
  - Point-to-point IP tunneling over IP networks
  - Routing protocol transport and site connectivity
- **[EoIP](eoip.md)** - Ethernet over IP tunneling (MikroTik proprietary)
  - Layer-2 transparent bridging over IP
  - Remote site integration at Ethernet level
- **IPSec** - Secure tunnels with encryption

### VPN interfaces
- **OpenVPN** - SSL/TLS-based VPN connections
- **WireGuard** - Modern, fast VPN protocol
- **L2TP** - Layer 2 Tunneling Protocol
- **SSTP** - Secure Socket Tunneling Protocol
- **PPTP** - Point-to-Point Tunneling Protocol (deprecated)

***

## Advanced interfaces

### Bonding and aggregation
- **Bonding** - Link aggregation for increased bandwidth and redundancy
- **LACP** - Link Aggregation Control Protocol for dynamic bonding

### Specialized interfaces  
- **Loopback** - Virtual interfaces for router identification
- **ZeroTier** - Software-defined networking
- **VPLS** - Virtual Private LAN Service
- **MPLS** - Multiprotocol Label Switching

***

## Interface configuration principles

### Common parameters
All interfaces share some common configuration options:

{% code overflow="wrap" %}
```bash
# Basic interface configuration
/interface set [interface] \
    name="descriptive-name" \
    mtu=1500 \
    disabled=no \
    comment="Interface description"

# Monitor interface status
/interface print stats
/interface monitor-traffic [interface]
```
{% endcode %}

### Interface naming conventions
- Use descriptive names that indicate purpose
- Include VLAN ID in VLAN interface names
- Use consistent prefixes for similar interface types
- Example: `ether1-wan`, `vlan10-mgmt`, `bridge1-lan`

***

## Interface selection guide

### Choose the right interface type

**For basic connectivity:**
- **Ethernet** - Wired LAN connections
- **WiFi** - Wireless LAN connections  
- **PPPoE** - DSL/cable internet connections

**For network segmentation:**
- **VLAN** - Segment traffic within same physical network
- **Bridge** - Connect multiple network segments
- **VXLAN** - Extend Layer 2 networks across Layer 3 boundaries

**For remote connectivity:**
- **VPN interfaces** - Secure remote access
- **GRE/EoIP** - Site-to-site tunnels
- **LTE** - Cellular backup connections

**For redundancy and performance:**
- **Bonding** - Combine multiple links
- **Bridge** with STP - Loop prevention in redundant topologies

***

## Best practices

### Interface design
1. **Plan addressing** - Use consistent IP addressing schemes
2. **Document everything** - Comment all interfaces with their purpose
3. **Use VLANs** - Segment traffic appropriately
4. **Consider redundancy** - Plan for link failures
5. **Monitor performance** - Track utilization and errors

### Security considerations
1. **Disable unused interfaces** - Reduce attack surface
2. **Use proper VLANs** - Isolate different network segments
3. **Implement access control** - Control who can access what
4. **Monitor traffic** - Watch for unusual patterns
5. **Regular maintenance** - Keep configurations up to date

### Performance optimization
1. **Choose appropriate MTU** - Match network requirements
2. **Enable hardware offloading** - When available and beneficial
3. **Monitor utilization** - Identify bottlenecks
4. **Use bonding** - For increased bandwidth where needed
5. **Optimize switch settings** - For bridge configurations

