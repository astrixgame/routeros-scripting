---
description: This section covers how to configure bridge interfaces for connecting multiple network segments.
icon: link
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

# Bridge

{% hint style="info" %}
Bridges in RouterOS connect multiple network interfaces at Layer 2, creating a single broadcast domain. They are essential for creating LANs and connecting different network segments.
{% endhint %}

In WinBox you can configure bridges in **Bridge**, or you can use terminal with command `/interface bridge`

Bridges operate at Layer 2 (Data Link layer) and forward traffic based on MAC addresses, similar to a traditional network switch.

***

## Bridge fundamentals

### How bridges work

**Bridge functionality:**
- **MAC learning** - Automatically learns device MAC addresses
- **Frame forwarding** - Forwards frames between bridge ports
- **Broadcast domain** - All bridge ports share the same broadcast domain
- **STP support** - Spanning Tree Protocol for loop prevention
- **VLAN support** - 802.1Q VLAN tagging and filtering

**Key concepts:**
- **Bridge interface** - Virtual interface representing the bridge
- **Bridge ports** - Physical/virtual interfaces added to the bridge
- **MAC table** - Database of learned MAC addresses per port
- **Aging time** - How long MAC entries remain in table

***

## Basic bridge configuration

### Create a simple bridge

In WinBox go to **Bridge** and click <kbd>**+**</kbd> to create new bridge:

* **Name** - Bridge name (e.g., "bridge1")
* **MTU** - Maximum transmission unit (usually 1500)
* **ARP** - Address Resolution Protocol setting (enabled/disabled)
* **Protocol Mode** - STP protocol (none, stp, rstp, mstp)

{% code overflow="wrap" %}
```bash
# Create basic bridge
/interface bridge add name=bridge1 protocol-mode=rstp comment="Main LAN bridge"
```
{% endcode %}

### Add interfaces to bridge

Add physical interfaces as bridge ports:

In WinBox go to **Bridge -> Ports** and click <kbd>**+**</kbd>:

* **Interface** - Select physical interface (e.g., ether2, ether3)
* **Bridge** - Select bridge interface (bridge1)
* **PVID** - Port VLAN ID for untagged traffic

{% code overflow="wrap" %}
```bash
# Add interfaces to bridge
/interface bridge port add interface=ether2 bridge=bridge1
/interface bridge port add interface=ether3 bridge=bridge1
/interface bridge port add interface=ether4 bridge=bridge1
/interface bridge port add interface=ether5 bridge=bridge1
```
{% endcode %}

### Assign IP address to bridge

{% code overflow="wrap" %}
```bash
# Assign IP address to bridge interface
/ip address add address=192.168.1.1/24 interface=bridge1 comment="LAN gateway"
```
{% endcode %}

***

## Advanced bridge configuration

### Hardware offloading

Modern RouterOS devices support hardware bridge acceleration:

{% code overflow="wrap" %}
```bash
# Enable hardware offloading (if supported)
/interface bridge set bridge1 hardware=yes

# Check hardware offload status
/interface bridge print detail
```
{% endcode %}

### Bridge settings optimization

{% code overflow="wrap" %}
```bash
# Optimize bridge settings
/interface bridge set bridge1 \
    ageing-time=5m \
    forward-delay=4s \
    max-message-age=6s \
    priority=0x8000 \
    transmit-hold-count=6 \
    protocol-mode=rstp \
    igmp-snooping=yes
```
{% endcode %}

### Port-specific settings

{% code overflow="wrap" %}
```bash
# Configure individual bridge ports
/interface bridge port set [find interface=ether2] \
    edge=yes \
    point-to-point=yes \
    path-cost=10 \
    priority=0x80

# Set port as access port (untagged)
/interface bridge port set [find interface=ether3] \
    pvid=10 \
    frame-types=admit-only-untagged-and-priority-tagged
```
{% endcode %}

***

## VLAN configuration with bridges

### Enable VLAN filtering

{% code overflow="wrap" %}
```bash
# Enable VLAN filtering on bridge
/interface bridge set bridge1 vlan-filtering=yes
```
{% endcode %}

### Create VLAN interfaces

{% code overflow="wrap" %}
```bash
# Create VLAN interfaces on bridge
/interface vlan add name=vlan10-mgmt vlan-id=10 interface=bridge1
/interface vlan add name=vlan20-users vlan-id=20 interface=bridge1
/interface vlan add name=vlan30-guest vlan-id=30 interface=bridge1

# Assign IP addresses to VLANs
/ip address add address=192.168.10.1/24 interface=vlan10-mgmt comment="Management VLAN"
/ip address add address=192.168.20.1/24 interface=vlan20-users comment="User VLAN"  
/ip address add address=192.168.30.1/24 interface=vlan30-guest comment="Guest VLAN"
```
{% endcode %}

### Configure VLAN bridge table

{% code overflow="wrap" %}
```bash
# Add VLANs to bridge VLAN table
/interface bridge vlan add bridge=bridge1 vlan-ids=10 tagged=bridge1,ether2 untagged=ether3
/interface bridge vlan add bridge=bridge1 vlan-ids=20 tagged=bridge1,ether2 untagged=ether4
/interface bridge vlan add bridge=bridge1 vlan-ids=30 tagged=bridge1,ether2 untagged=ether5

# Configure management VLAN (bridge access)
/interface bridge vlan add bridge=bridge1 vlan-ids=99 tagged=bridge1 comment="Management VLAN"
```
{% endcode %}

### Port VLAN configuration

{% code overflow="wrap" %}
```bash
# Configure access ports (untagged)
/interface bridge port set [find interface=ether3] pvid=10 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether4] pvid=20 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether5] pvid=30 frame-types=admit-only-untagged-and-priority-tagged

# Configure trunk port (tagged)
/interface bridge port set [find interface=ether2] frame-types=admit-only-vlan-tagged
```
{% endcode %}

***

## Spanning Tree Protocol (STP)

### Enable RSTP

{% code overflow="wrap" %}
```bash
# Configure RSTP on bridge
/interface bridge set bridge1 protocol-mode=rstp priority=0x1000

# Set bridge as root
/interface bridge set bridge1 priority=0x0000

# Configure port priorities and costs
/interface bridge port set [find interface=ether2] priority=0x10 path-cost=10
/interface bridge port set [find interface=ether3] priority=0x80 path-cost=100
```
{% endcode %}

### Port edge configuration

{% code overflow="wrap" %}
```bash
# Configure edge ports (connected to end devices)
/interface bridge port set [find interface=ether3] edge=yes point-to-point=yes
/interface bridge port set [find interface=ether4] edge=yes point-to-point=yes
/interface bridge port set [find interface=ether5] edge=yes point-to-point=yes

# Configure non-edge ports (connected to switches)
/interface bridge port set [find interface=ether2] edge=no point-to-point=yes
```
{% endcode %}

***

## Bridge monitoring and troubleshooting

### Monitor bridge status

{% code overflow="wrap" %}
```bash
# Check bridge configuration
/interface bridge print detail

# View bridge ports
/interface bridge port print detail

# Check MAC address table
/interface bridge host print

# Monitor STP status
/interface bridge monitor bridge1
```
{% endcode %}

### Bridge statistics

{% code overflow="wrap" %}
```bash
# Monitor bridge traffic
/interface monitor-traffic interface=bridge1

# Check port statistics
/interface bridge port monitor [find interface=ether2]

# View error counters
/interface print stats where name=bridge1
```
{% endcode %}

### Troubleshoot bridge issues

{% code overflow="wrap" %}
```bash
# Check for loops
/interface bridge host print where !active

# Monitor STP state changes
/log print where topics~"bridge"

# Check VLAN configuration
/interface bridge vlan print detail

# Test connectivity between bridge ports
/ping 192.168.1.100 interface=bridge1
```
{% endcode %}

***

## Special bridge configurations

### Bridge with WiFi

{% code overflow="wrap" %}
```bash
# Add WiFi interface to bridge
/interface bridge port add interface=wlan1 bridge=bridge1

# Configure WiFi bridge settings
/interface bridge port set [find interface=wlan1] \
    horizon=1 \
    multicast-router=disabled
```
{% endcode %}

### Bridge with VPN

{% code overflow="wrap" %}
```bash
# Add VPN interfaces to bridge (for bridged VPN)
/interface bridge port add interface=ovpn-server1 bridge=bridge1
/interface bridge port add interface=l2tp-server1 bridge=bridge1

# Configure VPN bridge ports
/interface bridge port set [find interface=ovpn-server1] \
    edge=yes \
    point-to-point=yes
```
{% endcode %}

### Multiple bridges

{% code overflow="wrap" %}
```bash
# Create separate bridges for different purposes
/interface bridge add name=bridge-lan protocol-mode=rstp comment="LAN Bridge"
/interface bridge add name=bridge-dmz protocol-mode=rstp comment="DMZ Bridge"
/interface bridge add name=bridge-guest protocol-mode=none comment="Guest Bridge"

# Add interfaces to appropriate bridges
/interface bridge port add interface=ether2 bridge=bridge-lan
/interface bridge port add interface=ether3 bridge=bridge-dmz
/interface bridge port add interface=wlan1 bridge=bridge-guest

# Assign IP addresses
/ip address add address=192.168.1.1/24 interface=bridge-lan
/ip address add address=192.168.100.1/24 interface=bridge-dmz
/ip address add address=192.168.200.1/24 interface=bridge-guest
```
{% endcode %}

***

## Bridge security

### MAC address filtering

{% code overflow="wrap" %}
```bash
# Enable MAC address learning limits
/interface bridge port set [find interface=ether3] learn=yes max-learned-entries=10

# Add static MAC entries
/interface bridge host add bridge=bridge1 mac-address=AA:BB:CC:DD:EE:FF interface=ether3

# Disable learning on specific ports
/interface bridge port set [find interface=ether4] learn=no
```
{% endcode %}

### Port isolation

{% code overflow="wrap" %}
```bash
# Isolate ports using horizon
/interface bridge port set [find interface=ether3] horizon=1
/interface bridge port set [find interface=ether4] horizon=1
/interface bridge port set [find interface=ether5] horizon=1

# Allow communication through bridge interface (horizon=none)
/interface bridge port set [find interface=bridge1] horizon=none
```
{% endcode %}

***

<details>

<summary>Show complete bridge setup with VLANs</summary>

{% code overflow="wrap" %}
```bash
# 1. Create bridge
/interface bridge add name=bridge1 protocol-mode=rstp vlan-filtering=yes comment="Main LAN Bridge"

# 2. Add physical interfaces to bridge
/interface bridge port add interface=ether2 bridge=bridge1 comment="Trunk to switch"
/interface bridge port add interface=ether3 bridge=bridge1 pvid=10 comment="Management port"
/interface bridge port add interface=ether4 bridge=bridge1 pvid=20 comment="User port"
/interface bridge port add interface=ether5 bridge=bridge1 pvid=30 comment="Guest port"

# 3. Configure port types
/interface bridge port set [find interface=ether2] frame-types=admit-only-vlan-tagged
/interface bridge port set [find interface=ether3] frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether4] frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether5] frame-types=admit-only-untagged-and-priority-tagged

# 4. Create VLAN interfaces
/interface vlan add name=vlan10-mgmt vlan-id=10 interface=bridge1
/interface vlan add name=vlan20-users vlan-id=20 interface=bridge1
/interface vlan add name=vlan30-guest vlan-id=30 interface=bridge1

# 5. Configure VLAN bridge table
/interface bridge vlan add bridge=bridge1 vlan-ids=10 tagged=bridge1,ether2 untagged=ether3
/interface bridge vlan add bridge=bridge1 vlan-ids=20 tagged=bridge1,ether2 untagged=ether4
/interface bridge vlan add bridge=bridge1 vlan-ids=30 tagged=bridge1,ether2 untagged=ether5

# 6. Assign IP addresses
/ip address add address=192.168.10.1/24 interface=vlan10-mgmt comment="Management"
/ip address add address=192.168.20.1/24 interface=vlan20-users comment="Users"
/ip address add address=192.168.30.1/24 interface=vlan30-guest comment="Guests"
```
{% endcode %}

</details>

## Performance considerations

### Bridge optimization

{% code overflow="wrap" %}
```bash
# Enable hardware offloading
/interface bridge set bridge1 hardware=yes

# Optimize aging time for large networks
/interface bridge set bridge1 ageing-time=10m

# Disable unnecessary features for performance
/interface bridge set bridge1 igmp-snooping=no multicast-querier=no
```
{% endcode %}

### Monitoring performance

{% code overflow="wrap" %}
```bash
# Monitor CPU usage
/system resource print

# Check interface utilization
/interface monitor-traffic interface=bridge1 duration=10

# Monitor bridge table size
/interface bridge host print count-only
```
{% endcode %}

***

## Best practices

### Bridge design principles

1. **Keep it simple** - Avoid overly complex bridge configurations
2. **Use VLANs** - Segment traffic using VLANs instead of multiple bridges
3. **Enable RSTP** - Use Rapid Spanning Tree for faster convergence
4. **Configure edge ports** - Set edge=yes for ports connected to end devices
5. **Monitor MAC table** - Keep track of learned MAC addresses

### Security recommendations

1. **Limit MAC learning** - Set maximum learned entries per port
2. **Use port isolation** - Isolate untrusted ports using horizon
3. **VLAN segmentation** - Separate different types of traffic
4. **Monitor loops** - Watch for bridging loops and STP issues
5. **Regular maintenance** - Clean up unused bridge configurations

### Troubleshooting tips

1. **Check STP status** - Ensure proper STP operation
2. **Monitor MAC table** - Look for MAC address flapping
3. **Test connectivity** - Verify communication between bridge segments
4. **Review logs** - Check for bridge-related error messages
5. **Verify VLAN config** - Ensure VLAN configuration is correct

