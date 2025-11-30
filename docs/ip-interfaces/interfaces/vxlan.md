---
description: This section covers how to configure VXLAN interfaces for overlay networking and network virtualization.
icon: network-wired
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

# VXLAN

{% hint style="info" %}
VXLAN (Virtual eXtensible Local Area Network) enables Layer 2 network segments to be extended over Layer 3 networks, providing network virtualization and multi-tenancy capabilities.
{% endhint %}

In WinBox you can configure VXLAN in **Interfaces -> VXLAN**, or you can use terminal with command `/interface vxlan`

VXLAN encapsulates Layer 2 Ethernet frames within UDP packets, allowing virtual Layer 2 networks to span across different physical Layer 3 networks.

***

## VXLAN fundamentals

### How VXLAN works

**Key components:**
- **VXLAN Network Identifier (VNI)** - 24-bit identifier for the VXLAN segment (up to 16 million segments)
- **VTEP (VXLAN Tunnel Endpoint)** - Device that encapsulates/decapsulates VXLAN traffic
- **UDP encapsulation** - VXLAN frames are encapsulated in UDP packets (port 4789)
- **Multicast or unicast** - Can use multicast for BUM (Broadcast, Unknown unicast, Multicast) traffic or unicast with static FDB

**Benefits:**
- **Network virtualization** - Create multiple logical Layer 2 networks over single physical infrastructure
- **Scalability** - Support for up to 16 million network segments
- **Multi-tenancy** - Isolate different customers/applications
- **Cloud integration** - Standard protocol used in cloud environments
- **WAN extension** - Extend Layer 2 domains across WAN links

***

## Basic VXLAN configuration

### Create VXLAN interface

In WinBox go to **Interfaces -> VXLAN** and click <kbd>**+**</kbd> to create new VXLAN interface:

* **Name** - VXLAN interface name (e.g., "vxlan1")
* **VNI** - VXLAN Network Identifier (0-16777215)
* **Port** - UDP port (default 4789)
* **Group** - Multicast group address for BUM traffic
* **Interface** - Underlying Layer 3 interface

{% code overflow="wrap" %}
```bash
# Create basic VXLAN interface
/interface vxlan add name=vxlan1 vni=100 port=4789 group=239.1.1.1 interface=ether1 disabled=no
```
{% endcode %}

### Configure underlying network

{% code overflow="wrap" %}
```bash
# Configure IP address on underlay interface
/ip address add address=10.0.0.1/24 interface=ether1

# Enable multicast routing (for multicast VXLAN)
/routing igmp-proxy add interface=ether1 upstream=yes
/routing igmp-proxy set [find] disabled=no
```
{% endcode %}

### Add VXLAN to bridge

{% code overflow="wrap" %}
```bash
# Create bridge for VXLAN overlay
/interface bridge add name=bridge-vxlan protocol-mode=none

# Add VXLAN interface to bridge
/interface bridge port add interface=vxlan1 bridge=bridge-vxlan

# Add local interfaces to same bridge
/interface bridge port add interface=ether2 bridge=bridge-vxlan
```
{% endcode %}

***

## Point-to-Point VXLAN

### Unicast VXLAN tunnel

For simple point-to-point connections without multicast:

{% code overflow="wrap" %}
```bash
# Site A configuration
/interface vxlan add name=vxlan-site-b vni=200 port=4789 remote-address=203.0.113.20 interface=ether1 disabled=no

# Site B configuration  
/interface vxlan add name=vxlan-site-a vni=200 port=4789 remote-address=203.0.113.10 interface=ether1 disabled=no

# Add to bridges on both sides
/interface bridge add name=bridge-overlay
/interface bridge port add interface=vxlan-site-b bridge=bridge-overlay
/interface bridge port add interface=ether2 bridge=bridge-overlay
```
{% endcode %}

### Static FDB entries

For unicast VXLAN without dynamic learning:

{% code overflow="wrap" %}
```bash
# Add static FDB entries
/interface bridge fdb add bridge=bridge-vxlan mac-address=AA:BB:CC:DD:EE:FF interface=vxlan1 static=yes

# View FDB table
/interface bridge fdb print
```
{% endcode %}

***

## Multi-point VXLAN with multicast

### Configure multicast underlay

{% code overflow="wrap" %}
```bash
# Enable PIM-SM for multicast VXLAN
/routing pim interface add interface=ether1 hello-interval=30s

# Configure RP (Rendezvous Point) - on designated router
/routing pim rp add address=10.0.0.100 group-prefix=239.0.0.0/8

# Enable PIM
/routing pim set [find] disabled=no
```
{% endcode %}

### Create VXLAN with multicast

{% code overflow="wrap" %}
```bash
# Create VXLAN interface with multicast group
/interface vxlan add name=vxlan-multicast vni=300 port=4789 group=239.1.1.100 interface=ether1 disabled=no

# Enable learning on bridge
/interface bridge add name=bridge-multicast-vxlan learning=yes
/interface bridge port add interface=vxlan-multicast bridge=bridge-multicast-vxlan learning=yes
```
{% endcode %}

***

## VXLAN with EVPN (Ethernet VPN)

### Basic EVPN configuration

{% code overflow="wrap" %}
```bash
# Configure BGP for EVPN
/routing bgp instance add name=evpn as=65001 router-id=10.0.0.1

# Configure EVPN address family
/routing bgp address-family add instance=evpn afi=l2vpn safi=evpn

# Add BGP peer for EVPN
/routing bgp peer add instance=evpn remote-address=10.0.0.2 remote-as=65001 address-families=l2vpn-evpn

# Create VXLAN with EVPN
/interface vxlan add name=vxlan-evpn vni=400 port=4789 interface=ether1 disabled=no

# Enable EVPN on VXLAN
/interface bridge add name=bridge-evpn
/interface bridge port add interface=vxlan-evpn bridge=bridge-evpn
```
{% endcode %}

***

## VXLAN over WAN

### Configure VXLAN over Internet

{% code overflow="wrap" %}
```bash
# Site A (Public IP: 203.0.113.10)
/interface vxlan add name=vxlan-wan vni=500 port=4789 remote-address=203.0.113.20 interface=ether1 disabled=no

# Configure firewall to allow VXLAN
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4789 comment="Allow VXLAN"

# Add to bridge
/interface bridge add name=bridge-wan-extend
/interface bridge port add interface=vxlan-wan bridge=bridge-wan-extend
/interface bridge port add interface=ether2 bridge=bridge-wan-extend

# Assign IP to bridge for management
/ip address add address=192.168.100.1/24 interface=bridge-wan-extend
```

**Site B configuration:**
{% code overflow="wrap" %}
```bash
# Site B (Public IP: 203.0.113.20)
/interface vxlan add name=vxlan-wan vni=500 port=4789 remote-address=203.0.113.10 interface=ether1 disabled=no

# Configure firewall
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4789 comment="Allow VXLAN"

# Add to bridge
/interface bridge add name=bridge-wan-extend  
/interface bridge port add interface=vxlan-wan bridge=bridge-wan-extend
/interface bridge port add interface=ether2 bridge=bridge-wan-extend

# Assign IP to bridge
/ip address add address=192.168.100.2/24 interface=bridge-wan-extend
```
{% endcode %}

***

## VXLAN with VLANs

### VXLAN per VLAN mapping

Map different VLANs to different VXLAN segments:

{% code overflow="wrap" %}
```bash
# Create VXLAN interfaces for different VLANs
/interface vxlan add name=vxlan-vlan10 vni=10 port=4789 remote-address=203.0.113.20 interface=ether1
/interface vxlan add name=vxlan-vlan20 vni=20 port=4789 remote-address=203.0.113.20 interface=ether1
/interface vxlan add name=vxlan-vlan30 vni=30 port=4789 remote-address=203.0.113.20 interface=ether1

# Create bridge with VLAN filtering
/interface bridge add name=bridge-vxlan-vlans vlan-filtering=yes

# Add VXLAN interfaces to bridge
/interface bridge port add interface=vxlan-vlan10 bridge=bridge-vxlan-vlans pvid=10
/interface bridge port add interface=vxlan-vlan20 bridge=bridge-vxlan-vlans pvid=20
/interface bridge port add interface=vxlan-vlan30 bridge=bridge-vxlan-vlans pvid=30

# Add local trunk port
/interface bridge port add interface=ether2 bridge=bridge-vxlan-vlans

# Configure VLAN table
/interface bridge vlan add bridge=bridge-vxlan-vlans vlan-ids=10 tagged=ether2 untagged=vxlan-vlan10
/interface bridge vlan add bridge=bridge-vxlan-vlans vlan-ids=20 tagged=ether2 untagged=vxlan-vlan20
/interface bridge vlan add bridge=bridge-vxlan-vlans vlan-ids=30 tagged=ether2 untagged=vxlan-vlan30
```
{% endcode %}

***

## Advanced VXLAN features

### Load balancing across multiple tunnels

{% code overflow="wrap" %}
```bash
# Create multiple VXLAN tunnels for load balancing
/interface vxlan add name=vxlan-lb1 vni=600 port=4789 remote-address=203.0.113.20 interface=ether1
/interface vxlan add name=vxlan-lb2 vni=600 port=4789 remote-address=203.0.113.21 interface=ether1

# Create bonding interface
/interface bonding add name=bond-vxlan slaves=vxlan-lb1,vxlan-lb2 mode=802.3ad

# Add bonding to bridge
/interface bridge add name=bridge-lb-vxlan
/interface bridge port add interface=bond-vxlan bridge=bridge-lb-vxlan
```
{% endcode %}

### VXLAN with IPSec encryption

For secure VXLAN over untrusted networks:

{% code overflow="wrap" %}
```bash
# Create IPSec policy for VXLAN traffic
/ip ipsec policy add src-address=10.0.0.1/32 dst-address=10.0.0.2/32 protocol=udp src-port=4789 dst-port=4789 action=encrypt

# Create VXLAN over IPSec-protected connection
/interface vxlan add name=vxlan-secure vni=700 port=4789 remote-address=10.0.0.2 interface=ether1
```
{% endcode %}

***

## VXLAN monitoring and troubleshooting

### Monitor VXLAN status

{% code overflow="wrap" %}
```bash
# Check VXLAN interface status
/interface vxlan print detail

# Monitor VXLAN traffic
/interface monitor-traffic interface=vxlan1

# Check bridge learning
/interface bridge host print where interface=vxlan1

# View statistics
/interface print stats where name=vxlan1
```
{% endcode %}

### Troubleshoot connectivity

{% code overflow="wrap" %}
```bash
# Test underlay connectivity
/ping 203.0.113.20 interface=ether1

# Check multicast connectivity (if using multicast)
/ping 239.1.1.1 interface=ether1

# Monitor logs
/log print where message~"vxlan"

# Check firewall rules
/ip firewall filter print where dst-port=4789
```
{% endcode %}

### Packet capture for VXLAN

{% code overflow="wrap" %}
```bash
# Capture VXLAN traffic on underlay interface
/tool sniffer start interface=ether1 filter-protocol=udp filter-port=4789 file-name=vxlan-capture.pcap

# Stop capture
/tool sniffer stop

# Download capture file from Files menu
```
{% endcode %}

***

## Performance optimization

### Optimize VXLAN performance

{% code overflow="wrap" %}
```bash
# Increase MTU on underlay interface (account for VXLAN overhead)
/interface set ether1 mtu=1600

# Set VXLAN interface MTU
/interface vxlan set vxlan1 mtu=1500

# Disable unnecessary bridge features for performance
/interface bridge set bridge-vxlan multicast-querier=no igmp-snooping=no
```
{% endcode %}

### Hardware acceleration

{% code overflow="wrap" %}
```bash
# Enable hardware acceleration on bridge (if supported)
/interface bridge set bridge-vxlan hardware=yes

# Check hardware offload capabilities
/interface bridge print detail where name=bridge-vxlan
```
{% endcode %}

***

## VXLAN security considerations

### Access control

{% code overflow="wrap" %}
```bash
# Restrict VXLAN access to specific sources
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4789 src-address=10.0.0.0/24 comment="Allow VXLAN from trusted network"
/ip firewall filter add chain=input action=drop protocol=udp dst-port=4789 comment="Drop other VXLAN traffic"
```
{% endcode %}

### VNI isolation

{% code overflow="wrap" %}
```bash
# Use different VNIs for different security zones
/interface vxlan add name=vxlan-dmz vni=100 port=4789 remote-address=203.0.113.20 interface=ether1
/interface vxlan add name=vxlan-internal vni=200 port=4789 remote-address=203.0.113.20 interface=ether1

# Separate bridges for isolation
/interface bridge add name=bridge-dmz
/interface bridge add name=bridge-internal

/interface bridge port add interface=vxlan-dmz bridge=bridge-dmz
/interface bridge port add interface=vxlan-internal bridge=bridge-internal
```
{% endcode %}

***

## Use cases and scenarios

### Data center interconnect (DCI)

**Scenario:** Connect multiple data centers with Layer 2 extension

{% code overflow="wrap" %}
```bash
# Data Center A
/interface vxlan add name=vxlan-dc-b vni=1000 port=4789 remote-address=DC-B-PUBLIC-IP interface=wan1
/interface bridge add name=bridge-dc-extend
/interface bridge port add interface=vxlan-dc-b bridge=bridge-dc-extend
/interface bridge port add interface=server-vlan bridge=bridge-dc-extend

# Data Center B (similar configuration)
/interface vxlan add name=vxlan-dc-a vni=1000 port=4789 remote-address=DC-A-PUBLIC-IP interface=wan1
```
{% endcode %}

### Cloud hybrid connectivity

**Scenario:** Extend on-premises network to cloud

{% code overflow="wrap" %}
```bash
# On-premises router
/interface vxlan add name=vxlan-cloud vni=2000 port=4789 remote-address=CLOUD-VM-PUBLIC-IP interface=ether1

# Extend specific VLAN to cloud
/interface bridge add name=bridge-hybrid vlan-filtering=yes
/interface bridge port add interface=vxlan-cloud bridge=bridge-hybrid pvid=100
/interface bridge port add interface=local-servers bridge=bridge-hybrid

# Configure VLAN for cloud extension
/interface vlan add name=cloud-extend vlan-id=100 interface=bridge-hybrid
/ip address add address=192.168.100.1/24 interface=cloud-extend
```
{% endcode %}

### Multi-tenant service provider

**Scenario:** Provide isolated Layer 2 services to multiple customers

{% code overflow="wrap" %}
```bash
# Customer A - VNI 3001
/interface vxlan add name=vxlan-customer-a vni=3001 port=4789 group=239.1.1.1 interface=ether1
/interface bridge add name=bridge-customer-a
/interface bridge port add interface=vxlan-customer-a bridge=bridge-customer-a
/interface bridge port add interface=customer-a-port bridge=bridge-customer-a

# Customer B - VNI 3002  
/interface vxlan add name=vxlan-customer-b vni=3002 port=4789 group=239.1.1.2 interface=ether1
/interface bridge add name=bridge-customer-b
/interface bridge port add interface=vxlan-customer-b bridge=bridge-customer-b
/interface bridge port add interface=customer-b-port bridge=bridge-customer-b
```
{% endcode %}

***

<details>

<summary>Show complete VXLAN site-to-site setup</summary>

{% code overflow="wrap" %}
```bash
# Site A Configuration (Public IP: 203.0.113.10)
# 1. Configure underlay network
/ip address add address=203.0.113.10/30 interface=ether1 comment="WAN connection"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.9

# 2. Create VXLAN tunnel
/interface vxlan add name=vxlan-site-b vni=1000 port=4789 remote-address=203.0.113.20 interface=ether1 disabled=no

# 3. Configure overlay bridge
/interface bridge add name=bridge-overlay protocol-mode=none learning=yes
/interface bridge port add interface=vxlan-site-b bridge=bridge-overlay learning=yes
/interface bridge port add interface=ether2 bridge=bridge-overlay comment="Local LAN"

# 4. Configure firewall
/ip firewall filter add chain=input action=accept protocol=udp dst-port=4789 src-address=203.0.113.20 comment="Allow VXLAN from Site B"

# 5. Assign IP to overlay
/ip address add address=192.168.1.1/24 interface=bridge-overlay comment="Overlay network"

# Site B Configuration (Public IP: 203.0.113.20) - Mirror configuration
/ip address add address=203.0.113.20/30 interface=ether1 comment="WAN connection"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.21

/interface vxlan add name=vxlan-site-a vni=1000 port=4789 remote-address=203.0.113.10 interface=ether1 disabled=no

/interface bridge add name=bridge-overlay protocol-mode=none learning=yes
/interface bridge port add interface=vxlan-site-a bridge=bridge-overlay learning=yes
/interface bridge port add interface=ether2 bridge=bridge-overlay comment="Local LAN"

/ip firewall filter add chain=input action=accept protocol=udp dst-port=4789 src-address=203.0.113.10 comment="Allow VXLAN from Site A"

/ip address add address=192.168.1.2/24 interface=bridge-overlay comment="Overlay network"
```
{% endcode %}

</details>

## Best practices

### Design considerations

1. **Plan VNI space** - Use consistent VNI allocation across the network
2. **MTU planning** - Account for VXLAN overhead (50 bytes) in network design
3. **Underlay design** - Ensure robust and redundant underlay connectivity
4. **Multicast vs unicast** - Choose appropriate BUM handling method
5. **Security** - Implement proper access controls and encryption where needed

### Operational recommendations

1. **Monitor underlay health** - VXLAN depends on stable underlay connectivity
2. **Document VNI mappings** - Maintain clear documentation of VNI assignments
3. **Test failover scenarios** - Verify behavior during underlay failures
4. **Capacity planning** - Monitor bandwidth utilization on underlay links
5. **Regular maintenance** - Keep FDB tables clean and monitor for issues

### Troubleshooting tips

1. **Start with underlay** - Always verify underlay connectivity first
2. **Check MTU** - Ensure proper MTU configuration throughout the path
3. **Monitor learning** - Verify MAC learning is working correctly
4. **Test incrementally** - Start with simple point-to-point before complex scenarios
5. **Use packet capture** - Capture and analyze VXLAN encapsulated traffic when troubleshooting
