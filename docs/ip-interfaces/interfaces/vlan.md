---
description: This section covers how to configure VLAN interfaces for network segmentation.
icon: layer-group
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

# VLAN

{% hint style="info" %}
VLANs (Virtual Local Area Networks) allow you to create multiple logical networks on a single physical infrastructure, providing network segmentation and improved security.
{% endhint %}

In WinBox you can configure VLANs in **Interfaces -> VLAN**, or you can use terminal with command `/interface vlan`

VLANs operate at Layer 2 using 802.1Q tagging to separate traffic into different broadcast domains.

***

## VLAN fundamentals

### How VLANs work

**VLAN concepts:**
- **VLAN ID** - Unique identifier (1-4094) for each VLAN
- **Tagged traffic** - Frames with VLAN tags (802.1Q)
- **Untagged traffic** - Native VLAN frames without tags
- **Trunk ports** - Carry multiple VLANs (tagged)
- **Access ports** - Single VLAN (untagged)

**Benefits:**
- **Network segmentation** - Separate different types of traffic
- **Security** - Isolate sensitive systems
- **Broadcast control** - Reduce broadcast domains
- **Flexibility** - Easy network changes without rewiring
- **Efficiency** - Better bandwidth utilization

***

## Basic VLAN configuration

### Create VLAN interfaces

In WinBox go to **Interfaces -> VLAN** and click <kbd>**+**</kbd>:

* **Name** - VLAN interface name (e.g., vlan10-mgmt)
* **VLAN ID** - VLAN identifier (10)
* **Interface** - Parent interface (bridge1, ether1, etc.)

{% code overflow="wrap" %}
```bash
# Create VLAN interfaces
/interface vlan add name=vlan10-mgmt vlan-id=10 interface=bridge1 comment="Management VLAN"
/interface vlan add name=vlan20-users vlan-id=20 interface=bridge1 comment="User VLAN"
/interface vlan add name=vlan30-guest vlan-id=30 interface=bridge1 comment="Guest VLAN"
/interface vlan add name=vlan40-servers vlan-id=40 interface=bridge1 comment="Server VLAN"
```
{% endcode %}

### Assign IP addresses to VLANs

{% code overflow="wrap" %}
```bash
# Assign IP addresses to VLAN interfaces
/ip address add address=192.168.10.1/24 interface=vlan10-mgmt comment="Management gateway"
/ip address add address=192.168.20.1/24 interface=vlan20-users comment="User gateway"
/ip address add address=192.168.30.1/24 interface=vlan30-guest comment="Guest gateway"
/ip address add address=192.168.40.1/24 interface=vlan40-servers comment="Server gateway"
```
{% endcode %}

***

## VLAN with bridge configuration

### Enable VLAN filtering on bridge

{% code overflow="wrap" %}
```bash
# Create bridge with VLAN support
/interface bridge add name=bridge1 vlan-filtering=yes protocol-mode=rstp comment="Main bridge with VLANs"

# Add interfaces to bridge
/interface bridge port add interface=ether1 bridge=bridge1 comment="Uplink trunk"
/interface bridge port add interface=ether2 bridge=bridge1 comment="Management access"
/interface bridge port add interface=ether3 bridge=bridge1 comment="User access"
/interface bridge port add interface=ether4 bridge=bridge1 comment="Guest access"
```
{% endcode %}

### Configure bridge VLAN table

{% code overflow="wrap" %}
```bash
# Configure VLAN membership in bridge
/interface bridge vlan add bridge=bridge1 vlan-ids=10 tagged=bridge1,ether1 untagged=ether2 comment="Management VLAN"
/interface bridge vlan add bridge=bridge1 vlan-ids=20 tagged=bridge1,ether1 untagged=ether3 comment="User VLAN"
/interface bridge vlan add bridge=bridge1 vlan-ids=30 tagged=bridge1,ether1 untagged=ether4 comment="Guest VLAN"
/interface bridge vlan add bridge=bridge1 vlan-ids=40 tagged=bridge1,ether1 comment="Server VLAN (trunk only)"
```
{% endcode %}

### Configure bridge ports for VLANs

{% code overflow="wrap" %}
```bash
# Configure trunk port (carries all VLANs tagged)
/interface bridge port set [find interface=ether1] frame-types=admit-only-vlan-tagged

# Configure access ports (single VLAN, untagged)
/interface bridge port set [find interface=ether2] pvid=10 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether3] pvid=20 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether4] pvid=30 frame-types=admit-only-untagged-and-priority-tagged
```
{% endcode %}

***

## Advanced VLAN scenarios

### Inter-VLAN routing

Enable communication between VLANs through RouterOS:

{% code overflow="wrap" %}
```bash
# Inter-VLAN routing is automatic with IP addresses on VLAN interfaces
# Add specific routes if needed
/ip route add dst-address=192.168.40.0/24 gateway=192.168.10.1 comment="Route to servers via mgmt"

# Control inter-VLAN traffic with firewall
/ip firewall filter add chain=forward action=drop src-address=192.168.30.0/24 dst-address=192.168.40.0/24 comment="Block guest from servers"
/ip firewall filter add chain=forward action=accept src-address=192.168.20.0/24 dst-address=192.168.40.0/24 comment="Allow users to servers"
```
{% endcode %}

### VLAN on physical interfaces

Create VLANs directly on physical interfaces:

{% code overflow="wrap" %}
```bash
# Create VLANs on physical interface (for router-on-a-stick)
/interface vlan add name=vlan10-ether1 vlan-id=10 interface=ether1
/interface vlan add name=vlan20-ether1 vlan-id=20 interface=ether1
/interface vlan add name=vlan30-ether1 vlan-id=30 interface=ether1

# Assign IP addresses
/ip address add address=192.168.10.1/24 interface=vlan10-ether1
/ip address add address=192.168.20.1/24 interface=vlan20-ether1
/ip address add address=192.168.30.1/24 interface=vlan30-ether1
```
{% endcode %}

### Multiple trunk configurations

{% code overflow="wrap" %}
```bash
# Configure different trunk ports with specific VLANs
/interface bridge port add interface=ether5 bridge=bridge1 comment="Partial trunk"
/interface bridge port set [find interface=ether5] frame-types=admit-only-vlan-tagged

# Allow only specific VLANs on this trunk
/interface bridge vlan set [find vlan-ids=10] tagged=bridge1,ether1,ether5
/interface bridge vlan set [find vlan-ids=20] tagged=bridge1,ether1,ether5
# Don't add ether5 to VLAN 30 and 40 - they won't be carried on this trunk
```
{% endcode %}

***

## VLAN security and isolation

### VLAN isolation with firewall

{% code overflow="wrap" %}
```bash
# Block all inter-VLAN communication by default
/ip firewall filter add chain=forward action=drop comment="Default deny inter-VLAN"

# Allow specific inter-VLAN communication
/ip firewall filter add chain=forward action=accept src-address=192.168.20.0/24 dst-address=192.168.40.0/24 dst-port=80,443 protocol=tcp comment="Users to web servers"
/ip firewall filter add chain=forward action=accept src-address=192.168.10.0/24 dst-address=192.168.40.0/24 protocol=tcp comment="Mgmt to servers"

# Allow intra-VLAN communication
/ip firewall filter add chain=forward action=accept in-interface=vlan20-users out-interface=vlan20-users comment="Users intra-VLAN"
```
{% endcode %}

### Private VLANs concept

Implement private VLAN-like functionality:

{% code overflow="wrap" %}
```bash
# Create isolated VLANs for different purposes
/interface vlan add name=vlan100-dmz vlan-id=100 interface=bridge1 comment="DMZ servers"
/interface vlan add name=vlan101-web vlan-id=101 interface=bridge1 comment="Web servers"
/interface vlan add name=vlan102-db vlan-id=102 interface=bridge1 comment="Database servers"

# Configure bridge VLANs
/interface bridge vlan add bridge=bridge1 vlan-ids=100 tagged=bridge1,ether1
/interface bridge vlan add bridge=bridge1 vlan-ids=101 tagged=bridge1,ether1
/interface bridge vlan add bridge=bridge1 vlan-ids=102 tagged=bridge1,ether1

# Implement micro-segmentation with firewall
/ip firewall filter add chain=forward action=drop src-address=192.168.101.0/24 dst-address=192.168.102.0/24 comment="Isolate web from DB"
/ip firewall filter add chain=forward action=accept src-address=192.168.101.0/24 dst-address=192.168.102.100 dst-port=3306 protocol=tcp comment="Web to specific DB server"
```
{% endcode %}

***

## VLAN with WiFi integration

### WiFi SSID to VLAN mapping

{% code overflow="wrap" %}
```bash
# Map different SSIDs to VLANs
/interface wifi set wifi-corp vlan-id=20
/interface wifi set wifi-guest vlan-id=30
/interface wifi set wifi-iot vlan-id=50

# Create IoT VLAN
/interface vlan add name=vlan50-iot vlan-id=50 interface=bridge1 comment="IoT devices"
/ip address add address=192.168.50.1/24 interface=vlan50-iot

# Add WiFi interfaces to bridge with VLAN assignment
/interface bridge port add interface=wifi-corp bridge=bridge1 pvid=20
/interface bridge port add interface=wifi-guest bridge=bridge1 pvid=30
/interface bridge port add interface=wifi-iot bridge=bridge1 pvid=50

# Configure VLAN bridge table for WiFi
/interface bridge vlan set [find vlan-ids=20] tagged=bridge1,ether1 untagged=wifi-corp
/interface bridge vlan set [find vlan-ids=30] tagged=bridge1,ether1 untagged=wifi-guest
/interface bridge vlan add bridge=bridge1 vlan-ids=50 tagged=bridge1,ether1 untagged=wifi-iot
```
{% endcode %}

***

## VLAN troubleshooting

### Diagnostic commands

{% code overflow="wrap" %}
```bash
# Check VLAN interface status
/interface vlan print detail

# Check bridge VLAN configuration
/interface bridge vlan print detail

# Check bridge port configuration
/interface bridge port print detail

# Monitor VLAN traffic
/interface monitor-traffic interface=vlan20-users
```
{% endcode %}

### Common VLAN issues

{% code overflow="wrap" %}
```bash
# Check if VLAN filtering is enabled
/interface bridge print detail where name=bridge1

# Verify VLAN membership
/interface bridge vlan print where bridge=bridge1

# Check port VLAN settings
/interface bridge port print where bridge=bridge1

# Test connectivity between VLANs
/ping 192.168.20.100 src-address=192.168.10.1
```
{% endcode %}

### VLAN packet capture

{% code overflow="wrap" %}
```bash
# Capture VLAN tagged traffic
/tool sniffer quick interface=ether1 ip-protocol=any

# Filter by VLAN ID in packet details
/tool sniffer quick interface=bridge1 filter-stream=yes
```
{% endcode %}

***

## Dynamic VLAN assignment

### 802.1X with VLAN assignment

{% code overflow="wrap" %}
```bash
# Configure RADIUS for dynamic VLAN assignment
/radius add service=dot1x address=192.168.10.100 secret="radius_secret"

# Enable 802.1X on bridge ports
/interface dot1x server add interface=ether3 auth-types=dot1x

# RADIUS will assign VLAN based on user credentials
# RADIUS server returns VLAN assignment attributes
```
{% endcode %}

### MAC-based VLAN assignment

{% code overflow="wrap" %}
```bash
# Create MAC address to VLAN mapping (manual)
/interface bridge host add bridge=bridge1 mac-address=AA:BB:CC:DD:EE:FF interface=ether3 vlan-id=20

# Use scripts for dynamic assignment based on MAC OUI
/system script add name=mac-vlan-assignment source={
    :local macAddress [/interface bridge host get [find interface=ether3] mac-address]
    :local oui [:pick $macAddress 0 8]
    :if ($oui = "00:1A:2B") do={
        /interface bridge host set [find mac-address=$macAddress] vlan-id=20
    }
}
```
{% endcode %}

***

## VLAN monitoring and management

### VLAN statistics

{% code overflow="wrap" %}
```bash
# Monitor VLAN interface statistics
/interface print stats where type="vlan"

# Check VLAN bridge table
/interface bridge host print where vlan-id=20

# Monitor MAC address learning per VLAN
/interface bridge host print where bridge=bridge1
```
{% endcode %}

### VLAN documentation

{% code overflow="wrap" %}
```bash
# Document VLAN assignments with comments
/interface vlan set [find name=vlan10-mgmt] comment="Management - Network equipment, servers"
/interface vlan set [find name=vlan20-users] comment="Users - Employee workstations, laptops"
/interface vlan set [find name=vlan30-guest] comment="Guest - Visitor access, limited internet"
/interface vlan set [find name=vlan40-servers] comment="Servers - Application and database servers"
/interface vlan set [find name=vlan50-iot] comment="IoT - Smart devices, sensors, cameras"
```
{% endcode %}

***

<details>

<summary>Show complete VLAN setup with bridge</summary>

{% code overflow="wrap" %}
```bash
# 1. Create bridge with VLAN support
/interface bridge add name=bridge1 vlan-filtering=yes protocol-mode=rstp comment="Main VLAN bridge"

# 2. Add physical interfaces to bridge
/interface bridge port add interface=ether1 bridge=bridge1 comment="Trunk to switch"
/interface bridge port add interface=ether2 bridge=bridge1 comment="Management access"
/interface bridge port add interface=ether3 bridge=bridge1 comment="User access"
/interface bridge port add interface=ether4 bridge=bridge1 comment="Guest access"

# 3. Configure port types
/interface bridge port set [find interface=ether1] frame-types=admit-only-vlan-tagged
/interface bridge port set [find interface=ether2] pvid=10 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether3] pvid=20 frame-types=admit-only-untagged-and-priority-tagged
/interface bridge port set [find interface=ether4] pvid=30 frame-types=admit-only-untagged-and-priority-tagged

# 4. Create VLAN interfaces
/interface vlan add name=vlan10-mgmt vlan-id=10 interface=bridge1 comment="Management VLAN"
/interface vlan add name=vlan20-users vlan-id=20 interface=bridge1 comment="User VLAN"
/interface vlan add name=vlan30-guest vlan-id=30 interface=bridge1 comment="Guest VLAN"

# 5. Configure bridge VLAN table
/interface bridge vlan add bridge=bridge1 vlan-ids=10 tagged=bridge1,ether1 untagged=ether2 comment="Management"
/interface bridge vlan add bridge=bridge1 vlan-ids=20 tagged=bridge1,ether1 untagged=ether3 comment="Users"
/interface bridge vlan add bridge=bridge1 vlan-ids=30 tagged=bridge1,ether1 untagged=ether4 comment="Guests"

# 6. Assign IP addresses
/ip address add address=192.168.10.1/24 interface=vlan10-mgmt comment="Management gateway"
/ip address add address=192.168.20.1/24 interface=vlan20-users comment="User gateway"
/ip address add address=192.168.30.1/24 interface=vlan30-guest comment="Guest gateway"

# 7. Configure inter-VLAN firewall rules
/ip firewall filter add chain=forward action=drop src-address=192.168.30.0/24 dst-address=!192.168.30.0/24 comment="Isolate guest VLAN"
/ip firewall filter add chain=forward action=accept src-address=192.168.20.0/24 dst-address=192.168.10.0/24 comment="Users can access management"
```
{% endcode %}

</details>

## VLAN best practices

### Design recommendations

1. **Plan VLAN numbering** - Use consistent VLAN ID scheme
2. **Document everything** - Maintain VLAN documentation
3. **Use meaningful names** - Include purpose in VLAN interface names
4. **Implement security** - Control inter-VLAN communication
5. **Monitor performance** - Track VLAN utilization

### VLAN numbering scheme

**Recommended VLAN ranges:**
- **1-99** - Infrastructure (management, network equipment)
- **100-199** - Servers (web, database, application)
- **200-299** - Users (employees, departments)
- **300-399** - Guest networks
- **400-499** - IoT and devices
- **500-599** - Voice/VoIP
- **600-699** - Video/surveillance
- **700-799** - Wireless networks
- **800-899** - DMZ/public services
- **900-999** - Testing/development

### Security considerations

{% code overflow="wrap" %}
```bash
# Default deny inter-VLAN traffic
/ip firewall filter add chain=forward action=drop place-before=0 comment="Default deny inter-VLAN"

# Allow necessary inter-VLAN communication
/ip firewall filter add chain=forward action=accept src-address=192.168.20.0/24 dst-address=192.168.100.0/24 dst-port=80,443 protocol=tcp comment="Users to web servers"

# Log inter-VLAN attempts
/ip firewall filter add chain=forward action=log src-address=192.168.30.0/24 dst-address=!192.168.30.0/24 log-prefix="GUEST-VLAN-ATTEMPT"
```
{% endcode %}

