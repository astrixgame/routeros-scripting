---
description: This section covers how to configure EoIP interfaces for Ethernet over IP tunneling.
icon: ethernet
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

# EoIP

{% hint style="info" %}
EoIP (Ethernet over IP) is a MikroTik proprietary tunneling protocol that creates a layer-2 tunnel, allowing Ethernet frames to be transported over IP networks as if they were on the same physical network segment.
{% endhint %}

In WinBox you can configure EoIP in **Interfaces -> EoIP**, or you can use terminal with command `/interface eoip`

EoIP creates true layer-2 connectivity, making remote networks appear as if they are directly connected at the Ethernet level.

***

## EoIP fundamentals

### How EoIP works

**Key characteristics:**
- **Layer-2 tunnel** - Transports Ethernet frames, not just IP packets
- **MikroTik proprietary** - Only works between RouterOS devices
- **Protocol 47** - Uses GRE protocol (same as GRE tunnels)
- **Transparent bridging** - Can be added to bridge interfaces
- **MAC learning** - Supports standard Ethernet MAC address learning
- **VLAN transport** - Can carry VLAN tagged traffic

**Common use cases:**
- **Remote bridging** - Extend layer-2 networks across WAN
- **Site interconnection** - Connect remote sites at layer-2
- **VLAN extension** - Extend VLANs across geographic locations  
- **Legacy network support** - Connect networks requiring layer-2 adjacency
- **Cluster networking** - Connect servers/services requiring broadcast domains

***

## Basic EoIP configuration

### Simple point-to-point EoIP tunnel

Create a basic EoIP tunnel between two RouterOS devices:

**Router A (Local: 203.0.113.10, Remote: 203.0.113.20):**
{% code overflow="wrap" %}
```bash
# Create EoIP interface
/interface eoip add name=eoip-tunnel1 \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=10 \
    disabled=no

# Add to bridge (optional - for layer-2 extension)
/interface bridge add name=br-local disabled=no
/interface bridge port add bridge=br-local interface=eoip-tunnel1
/interface bridge port add bridge=br-local interface=ether2 comment="Local LAN"
```
{% endcode %}

**Router B (Local: 203.0.113.20, Remote: 203.0.113.10):**
{% code overflow="wrap" %}
```bash
# Create EoIP interface (same tunnel-id required)
/interface eoip add name=eoip-tunnel1 \
    local-address=203.0.113.20 \
    remote-address=203.0.113.10 \
    tunnel-id=10 \
    disabled=no

# Add to bridge for layer-2 extension
/interface bridge add name=br-remote disabled=no
/interface bridge port add bridge=br-remote interface=eoip-tunnel1
/interface bridge port add bridge=br-remote interface=ether2 comment="Remote LAN"
```
{% endcode %}

### Test EoIP connectivity

{% code overflow="wrap" %}
```bash
# Check interface status
/interface eoip print detail

# Monitor interface traffic  
/interface monitor-traffic interface=eoip-tunnel1

# Test bridge functionality
/interface bridge print
/interface bridge port print where bridge=br-local
```
{% endcode %}

***

## Advanced EoIP configuration

### EoIP with keepalive

Enable keepalive for tunnel failure detection:

{% code overflow="wrap" %}
```bash
# Configure EoIP with keepalive
/interface eoip add name=eoip-keepalive \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=20 \
    keepalive=10,3 \
    disabled=no

# keepalive=10,3 means send keepalive every 10 seconds, timeout after 3 failures
```
{% endcode %}

### Multiple EoIP tunnels

Create multiple tunnels with different tunnel IDs:

{% code overflow="wrap" %}
```bash
# Primary tunnel
/interface eoip add name=eoip-primary \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=100 \
    disabled=no

# Backup tunnel (different path)
/interface eoip add name=eoip-backup \
    local-address=203.0.113.11 \
    remote-address=203.0.113.21 \
    tunnel-id=101 \
    disabled=no

# Add both to same bridge for redundancy
/interface bridge add name=br-redundant disabled=no
/interface bridge port add bridge=br-redundant interface=eoip-primary
/interface bridge port add bridge=br-redundant interface=eoip-backup
```
{% endcode %}

### EoIP hub and spoke topology

Central site connecting multiple remote locations:

{% code overflow="wrap" %}
```bash
# Hub site - create tunnels to each spoke
/interface eoip add name=eoip-branch1 local-address=203.0.113.1 remote-address=203.0.113.10 tunnel-id=201 disabled=no
/interface eoip add name=eoip-branch2 local-address=203.0.113.1 remote-address=203.0.113.20 tunnel-id=202 disabled=no  
/interface eoip add name=eoip-branch3 local-address=203.0.113.1 remote-address=203.0.113.30 tunnel-id=203 disabled=no

# Add all tunnels to main bridge
/interface bridge add name=br-corporate disabled=no
/interface bridge port add bridge=br-corporate interface=eoip-branch1
/interface bridge port add bridge=br-corporate interface=eoip-branch2
/interface bridge port add bridge=br-corporate interface=eoip-branch3
/interface bridge port add bridge=br-corporate interface=ether2 comment="Head office LAN"
```
{% endcode %}

***

## EoIP with VLANs

### Transport VLANs over EoIP

EoIP can carry VLAN tagged traffic transparently:

{% code overflow="wrap" %}
```bash
# Site A - Configure EoIP tunnel
/interface eoip add name=eoip-vlan-transport \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=300 \
    disabled=no

# Create VLAN interfaces on local side
/interface vlan add name=vlan100 vlan-id=100 interface=ether2
/interface vlan add name=vlan200 vlan-id=200 interface=ether2

# Create bridge and add interfaces
/interface bridge add name=br-vlan-transport disabled=no
/interface bridge port add bridge=br-vlan-transport interface=eoip-vlan-transport
/interface bridge port add bridge=br-vlan-transport interface=ether3 comment="VLAN trunk port"

# Configure VLAN filtering on bridge
/interface bridge vlan add bridge=br-vlan-transport tagged=eoip-vlan-transport,ether3 vlan-ids=100
/interface bridge vlan add bridge=br-vlan-transport tagged=eoip-vlan-transport,ether3 vlan-ids=200
/interface bridge set br-vlan-transport vlan-filtering=yes
```
{% endcode %}

### Separate tunnels per VLAN

For better control and security, use separate EoIP tunnels:

{% code overflow="wrap" %}
```bash
# Tunnel for VLAN 100
/interface eoip add name=eoip-vlan100 \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=400 \
    disabled=no

# Tunnel for VLAN 200  
/interface eoip add name=eoip-vlan200 \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=401 \
    disabled=no

# Create separate bridges per VLAN
/interface bridge add name=br-vlan100 disabled=no
/interface bridge add name=br-vlan200 disabled=no

/interface bridge port add bridge=br-vlan100 interface=eoip-vlan100
/interface bridge port add bridge=br-vlan200 interface=eoip-vlan200

# Create VLAN interfaces
/interface vlan add name=vlan100 vlan-id=100 interface=br-vlan100
/interface vlan add name=vlan200 vlan-id=200 interface=br-vlan200
```
{% endcode %}

***

## EoIP security

### Secure EoIP with IPSec

EoIP itself provides no encryption, use IPSec for security:

{% code overflow="wrap" %}
```bash
# Create IPSec proposal
/ip ipsec proposal add name=eoip-proposal enc-algorithms=aes-256-cbc auth-algorithms=sha256 pfs-group=modp2048

# Create IPSec peer  
/ip ipsec peer add name=eoip-peer address=203.0.113.20 exchange-mode=ike2 passive=no

# Create IPSec identity
/ip ipsec identity add peer=eoip-peer secret="your-shared-secret" my-id=address:203.0.113.10 remote-id=address:203.0.113.20

# Create IPSec policy for GRE traffic (EoIP uses GRE protocol)
/ip ipsec policy add src-address=203.0.113.10/32 dst-address=203.0.113.20/32 protocol=gre action=encrypt proposal=eoip-proposal peer=eoip-peer

# Create EoIP tunnel (will be encrypted by IPSec)
/interface eoip add name=eoip-secure \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=500 \
    disabled=no
```
{% endcode %}

### Firewall protection for EoIP

{% code overflow="wrap" %}
```bash
# Allow GRE protocol from specific sources only
/ip firewall filter add chain=input action=accept protocol=gre src-address=203.0.113.20 comment="Allow EoIP from trusted site"
/ip firewall filter add chain=input action=drop protocol=gre comment="Drop all other GRE traffic"

# Control bridge traffic if needed
/interface bridge filter add chain=forward action=drop in-interface=eoip-tunnel1 src-mac-address=!aa:bb:cc:dd:ee:ff comment="Allow only specific MAC"
```
{% endcode %}

***

## EoIP over NAT

### Handle EoIP behind NAT

EoIP uses GRE protocol which requires special handling behind NAT:

{% code overflow="wrap" %}
```bash
# On NAT router - forward GRE protocol to internal EoIP router
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=192.168.1.100 protocol=gre comment="Forward EoIP to internal router"

# Alternative: Use port forwarding if NAT supports it
/ip firewall nat add chain=dstnat action=dst-nat to-addresses=192.168.1.100 to-ports=1701 protocol=udp dst-port=1701 comment="Forward if using EoIP over UDP"
```
{% endcode %}

### Dynamic IP with EoIP

One end with dynamic IP address:

{% code overflow="wrap" %}
```bash
# Server side (static public IP)
/interface eoip add name=eoip-dynamic \
    local-address=203.0.113.10 \
    remote-address=0.0.0.0 \
    tunnel-id=600 \
    allow-fast-path=no \
    disabled=no

# Client side (dynamic IP) 
/interface eoip add name=eoip-server \
    local-address=0.0.0.0 \
    remote-address=203.0.113.10 \
    tunnel-id=600 \
    disabled=no
```
{% endcode %}

***

## Monitoring and troubleshooting

### Monitor EoIP tunnels

{% code overflow="wrap" %}
```bash
# Check EoIP interface status
/interface eoip print detail

# Monitor tunnel traffic
/interface monitor-traffic interface=eoip-tunnel1

# Check interface statistics
/interface print stats where name=eoip-tunnel1

# Monitor bridge MAC table
/interface bridge host print where bridge=br-local

# Check keepalive status  
/log print where topics~"eoip"
```
{% endcode %}

### Troubleshoot EoIP issues

{% code overflow="wrap" %}
```bash
# Test underlying connectivity
/ping 203.0.113.20 interface=ether1

# Check tunnel ID conflicts
/interface eoip print detail where tunnel-id=10

# Test bridge functionality
/interface bridge monitor br-local

# Check firewall rules
/ip firewall filter print where protocol=gre

# Packet capture
/tool sniffer start interface=ether1 filter-protocol=gre file-name=eoip-debug.pcap

# Check for MAC address issues
/interface bridge host print where bridge=br-local
```
{% endcode %}

***

## Performance optimization

### Optimize EoIP performance

{% code overflow="wrap" %}
```bash
# Disable fast-path if experiencing issues  
/interface eoip set eoip-tunnel1 allow-fast-path=no

# Set appropriate MTU (account for EoIP overhead)
/interface eoip set eoip-tunnel1 mtu=1500

# Monitor performance
/interface monitor-traffic interface=eoip-tunnel1 duration=10

# Optimize bridge settings
/interface bridge set br-local admin-mac=auto frame-types=admit-all ingress-filtering=no
```
{% endcode %}

### Bridge optimization for EoIP

{% code overflow="wrap" %}
```bash
# Optimize bridge for EoIP performance
/interface bridge set br-local \
    admin-mac=auto \
    auto-mac=yes \
    ageing-time=5m \
    max-message-age=20s \
    forward-delay=15s \
    transmit-hold-count=6 \
    protocol-mode=rstp \
    igmp-snooping=yes

# Optimize bridge ports
/interface bridge port set [find interface=eoip-tunnel1] \
    edge=yes \
    point-to-point=yes \
    path-cost=10
```
{% endcode %}

***

## Common use cases

### Remote site layer-2 extension

Connect remote office as if it's on same LAN:

{% code overflow="wrap" %}
```bash
# Main office (192.168.1.0/24)
/interface eoip add name=eoip-branch local-address=203.0.113.1 remote-address=203.0.113.10 tunnel-id=700 disabled=no
/interface bridge add name=br-main disabled=no
/interface bridge port add bridge=br-main interface=ether2 comment="Main office LAN"  
/interface bridge port add bridge=br-main interface=eoip-branch comment="Branch tunnel"

# Branch office - same configuration but reverse addresses
/interface eoip add name=eoip-main local-address=203.0.113.10 remote-address=203.0.113.1 tunnel-id=700 disabled=no
/interface bridge add name=br-branch disabled=no
/interface bridge port add bridge=br-branch interface=ether2 comment="Branch office LAN"
/interface bridge port add bridge=br-branch interface=eoip-main comment="Main tunnel"

# Both sites now share same broadcast domain
```
{% endcode %}

### Server farm interconnection

Connect multiple data centers at layer-2:

{% code overflow="wrap" %}
```bash
# Data center A  
/interface eoip add name=eoip-dc-b local-address=203.0.113.10 remote-address=203.0.113.20 tunnel-id=800 disabled=no
/interface eoip add name=eoip-dc-c local-address=203.0.113.10 remote-address=203.0.113.30 tunnel-id=801 disabled=no

# Create server VLAN bridge
/interface bridge add name=br-servers disabled=no
/interface bridge port add bridge=br-servers interface=eoip-dc-b
/interface bridge port add bridge=br-servers interface=eoip-dc-c  
/interface bridge port add bridge=br-servers interface=ether3 comment="Local servers"

# Configure server VLAN
/interface vlan add name=servers-vlan vlan-id=100 interface=br-servers
/ip address add address=10.0.100.1/24 interface=servers-vlan
```
{% endcode %}

***

<details>

<summary>Show complete EoIP site-to-site setup</summary>

{% code overflow="wrap" %}
```bash
# Site A Configuration (Public IP: 203.0.113.10, LAN: 192.168.1.0/24)
# 1. Create EoIP tunnel
/interface eoip add name=eoip-to-site-b \
    local-address=203.0.113.10 \
    remote-address=203.0.113.20 \
    tunnel-id=1000 \
    keepalive=10,3 \
    disabled=no

# 2. Create bridge for layer-2 extension
/interface bridge add name=br-extended-lan \
    admin-mac=auto \
    auto-mac=yes \
    protocol-mode=rstp \
    disabled=no

# 3. Add local LAN and tunnel to bridge  
/interface bridge port add bridge=br-extended-lan interface=ether2 comment="Site A LAN"
/interface bridge port add bridge=br-extended-lan interface=eoip-to-site-b comment="Tunnel to Site B"

# 4. Configure firewall to allow GRE
/ip firewall filter add chain=input action=accept protocol=gre src-address=203.0.113.20 comment="Allow EoIP from Site B"

# 5. Optional: Configure IPSec encryption for security
/ip ipsec proposal add name=eoip-encrypt enc-algorithms=aes-256-cbc auth-algorithms=sha256 pfs-group=modp2048
/ip ipsec peer add name=site-b address=203.0.113.20 exchange-mode=ike2 passive=no  
/ip ipsec identity add peer=site-b secret="shared-secret-key" my-id=address:203.0.113.10 remote-id=address:203.0.113.20
/ip ipsec policy add src-address=203.0.113.10/32 dst-address=203.0.113.20/32 protocol=gre action=encrypt proposal=eoip-encrypt peer=site-b

# Site B Configuration (Public IP: 203.0.113.20, LAN: 192.168.2.0/24) - Mirror setup
/interface eoip add name=eoip-to-site-a local-address=203.0.113.20 remote-address=203.0.113.10 tunnel-id=1000 keepalive=10,3 disabled=no
/interface bridge add name=br-extended-lan admin-mac=auto auto-mac=yes protocol-mode=rstp disabled=no
/interface bridge port add bridge=br-extended-lan interface=ether2 comment="Site B LAN" 
/interface bridge port add bridge=br-extended-lan interface=eoip-to-site-a comment="Tunnel to Site A"
/ip firewall filter add chain=input action=accept protocol=gre src-address=203.0.113.10 comment="Allow EoIP from Site A"

# Now both sites share the same layer-2 broadcast domain
# Devices can communicate directly without routing
```
{% endcode %}

</details>

## Best practices

### Design recommendations

1. **Use unique tunnel IDs** - Avoid conflicts between tunnels
2. **Implement keepalives** - Detect failures quickly  
3. **Plan carefully** - Layer-2 extension affects broadcast domains
4. **Monitor bridge tables** - Watch MAC address learning
5. **Use VLANs wisely** - Segment traffic appropriately

### Security considerations  

1. **Always encrypt** - EoIP provides no security by itself
2. **Limit bridge scope** - Don't extend layer-2 unnecessarily  
3. **Filter MAC addresses** - Control which devices can communicate
4. **Monitor traffic** - Watch for unusual broadcast patterns
5. **Use separate tunnels** - Isolate different network segments

### Performance tips

1. **Optimize bridges** - Configure spanning tree properly
2. **Control broadcasts** - Limit broadcast/multicast traffic
3. **Monitor utilization** - Watch tunnel bandwidth usage  
4. **Use hardware acceleration** - When available
5. **Plan redundancy** - Multiple tunnels for critical connections