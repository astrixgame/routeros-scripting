---
description: >-
  A decentralized VPN that allows devices to connect directly in a Peer-To-Peer
  network. It's designed for mesh networking and is ideal for private, virtual
  networks.
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

# ZeroTier

{% hint style="info" %}
ZeroTier creates secure peer-to-peer networks that work like a traditional Ethernet switch, but across the internet. It's ideal for connecting devices across different locations seamlessly.
{% endhint %}

In RouterOS v7+, you can configure ZeroTier in **ZeroTier** menu, or use terminal with command `/zerotier`

ZeroTier is different from traditional VPNs - it creates a **software-defined network** where all devices appear to be on the same LAN, regardless of their physical location.

***

## ZeroTier concepts

### How ZeroTier works

**Network model:**
- Creates virtual Ethernet networks (like VLANs but global)
- Each device gets a ZeroTier address (like a MAC address)
- Devices communicate directly when possible (P2P)
- Falls back to relay servers when direct connection isn't possible

**Key components:**
- **Network ID** - Unique identifier for your virtual network (16-character hex)
- **Node ID** - Unique identifier for each device (10-character hex)  
- **Controllers** - Manage network membership and configuration
- **Roots** - ZeroTier's infrastructure servers for coordination

### Advantages over traditional VPN

**Benefits:**
- **True mesh networking** - All devices can communicate with each other
- **Automatic NAT traversal** - Works behind firewalls and NAT
- **No central server required** - Devices connect directly when possible
- **Easy management** - Web-based network controller
- **Cross-platform** - Works on virtually any device
- **Scalable** - Networks can grow to thousands of devices

***

## Prerequisites

Before setting up ZeroTier on RouterOS:

1. **RouterOS v7.1+** - ZeroTier support was added in version 7.1
2. **Internet connectivity** - Required for initial network join and coordination
3. **ZeroTier account** - Sign up at https://my.zerotier.com
4. **Network created** - Create a network in ZeroTier Central

***

## ZeroTier Central setup

### Create ZeroTier account and network

1. **Sign up** at https://my.zerotier.com
2. **Create new network**:
   - Click "Create A Network"
   - Note the **Network ID** (16-character hex string)
   - Configure network settings:
     - **Name** - Give your network a descriptive name
     - **Access Control** - Set to "Private" (recommended)
     - **IPv4 Auto-Assign** - Enable and configure IP range
     - **IPv6 Auto-Assign** - Enable if needed

### Network configuration in ZeroTier Central

**Basic settings:**
- **Network Name** - "Company Network" or similar
- **Description** - Brief description of network purpose
- **Access Control** - Private (devices must be authorized)
- **Certificate** - Auto-generated, shows network is valid

**IP assignment:**
- **IPv4 Auto-Assign Pool** - e.g., 192.168.192.0/24
- **IPv6 Auto-Assign** - Enable if using IPv6
- **Managed Routes** - Routes to advertise to network members

***

## RouterOS ZeroTier configuration

### Install and enable ZeroTier

{% code overflow="wrap" %}
```bash
# Check if ZeroTier is available (RouterOS v7.1+)
/zerotier print

# Enable ZeroTier service
/zerotier set enabled=yes

# Join a ZeroTier network (replace with your Network ID)
/zerotier join network=8056c2e21c000001 name=company-net
```
{% endcode %}

### Check connection status

{% code overflow="wrap" %}
```bash
# Check ZeroTier status
/zerotier print detail

# View network interface created
/interface print where type="zerotier"

# Check if connected to network
/zerotier peer print
```
{% endcode %}

### Authorize device in ZeroTier Central

After joining the network:

1. **Go to ZeroTier Central** - https://my.zerotier.com
2. **Select your network** - Click on the network you created
3. **Find your RouterOS device** - Look for new device in "Members" section
4. **Authorize device** - Check the "Auth?" checkbox
5. **Assign IP** - ZeroTier will auto-assign IP, or set manually
6. **Set name** - Give device a descriptive name

***

## Interface configuration

### Configure ZeroTier interface IP

The ZeroTier interface acts like a regular network interface:

{% code overflow="wrap" %}
```bash
# Check ZeroTier interface name
/interface print where type="zerotier"

# Add IP address to ZeroTier interface (if not auto-assigned)
/ip address add address=192.168.192.1/24 interface=zerotier1

# Check assigned IP
/ip address print where interface~"zerotier"
```
{% endcode %}

### Bridge ZeroTier with local network

To allow ZeroTier devices to access local network:

{% code overflow="wrap" %}
```bash
# Create bridge including ZeroTier interface
/interface bridge add name=zt-bridge

# Add ZeroTier interface to bridge
/interface bridge port add interface=zerotier1 bridge=zt-bridge

# Add local interface to bridge (be careful with this!)
/interface bridge port add interface=ether2 bridge=zt-bridge

# Assign IP to bridge
/ip address add address=192.168.1.1/24 interface=zt-bridge
```
{% endcode %}

***

## Routing configuration

### Basic routing for ZeroTier

{% code overflow="wrap" %}
```bash
# ZeroTier automatically creates routes, check them
/ip route print where dst-address~"192.168.192"

# Add manual route if needed
/ip route add dst-address=192.168.192.0/24 gateway=zerotier1

# Route local network through ZeroTier (site-to-site)
/ip route add dst-address=192.168.1.0/24 gateway=192.168.192.1 distance=10
```
{% endcode %}

### Site-to-site connectivity

Configure RouterOS to route local networks over ZeroTier:

**Router A (Site A - 192.168.1.0/24):**
{% code overflow="wrap" %}
```bash
# Join ZeroTier network
/zerotier join network=8056c2e21c000001 name=site-a

# Configure IP on ZeroTier interface
/ip address add address=192.168.192.10/24 interface=zerotier1

# Advertise local network to ZeroTier
# (Configure in ZeroTier Central - Managed Routes)
# Add route: 192.168.1.0/24 via 192.168.192.10
```
{% endcode %}

**Router B (Site B - 192.168.2.0/24):**
{% code overflow="wrap" %}
```bash
# Join same ZeroTier network
/zerotier join network=8056c2e21c000001 name=site-b

# Configure IP on ZeroTier interface
/ip address add address=192.168.192.20/24 interface=zerotier1

# Advertise local network to ZeroTier
# (Configure in ZeroTier Central - Managed Routes)
# Add route: 192.168.2.0/24 via 192.168.192.20
```
{% endcode %}

***

## Firewall configuration

### Allow ZeroTier traffic

{% code overflow="wrap" %}
```bash
# Allow ZeroTier protocol traffic (UDP 9993)
/ip firewall filter add chain=input action=accept protocol=udp dst-port=9993 comment="Allow ZeroTier"

# Allow traffic from ZeroTier network
/ip firewall filter add chain=input action=accept in-interface=zerotier1 comment="Allow ZeroTier network"

# Allow forwarding between ZeroTier and local networks
/ip firewall filter add chain=forward action=accept in-interface=zerotier1 comment="ZeroTier forward in"
/ip firewall filter add chain=forward action=accept out-interface=zerotier1 comment="ZeroTier forward out"
```
{% endcode %}

### NAT configuration (if needed)

If ZeroTier devices need internet access through RouterOS:

{% code overflow="wrap" %}
```bash
# NAT for ZeroTier network
/ip firewall nat add chain=srcnat src-address=192.168.192.0/24 out-interface=ether1 action=masquerade comment="ZeroTier NAT"
```
{% endcode %}

***

## Advanced ZeroTier configuration

### Multiple ZeroTier networks

RouterOS can join multiple ZeroTier networks:

{% code overflow="wrap" %}
```bash
# Join primary company network
/zerotier join network=8056c2e21c000001 name=company-main

# Join secondary network (e.g., for partners)
/zerotier join network=8056c2e21c000002 name=partner-net

# Join IoT network
/zerotier join network=8056c2e21c000003 name=iot-devices

# Check all networks
/zerotier print
```
{% endcode %}

### Custom ZeroTier controller

For enterprise use, you can run your own ZeroTier controller:

{% code overflow="wrap" %}
```bash
# Connect to self-hosted controller
/zerotier join network=8056c2e21c000001 controller=192.168.1.100:9993
```
{% endcode %}

### ZeroTier network segmentation

Use VLANs with ZeroTier for network segmentation:

{% code overflow="wrap" %}
```bash
# Create VLAN on ZeroTier interface
/interface vlan add name=zt-vlan100 vlan-id=100 interface=zerotier1

# Assign IP to VLAN
/ip address add address=10.100.1.1/24 interface=zt-vlan100

# Create firewall rules for segmentation
/ip firewall filter add chain=forward action=drop in-interface=zt-vlan100 out-interface=zerotier1 comment="Block VLAN 100 to main ZT"
```
{% endcode %}

***

## Monitoring and management

### Check ZeroTier status

{% code overflow="wrap" %}
```bash
# View ZeroTier service status
/zerotier print detail

# Check network peers
/zerotier peer print

# Monitor interface statistics
/interface monitor-traffic interface=zerotier1

# Check connectivity to ZeroTier roots
/ping 8.8.8.8 interface=zerotier1
```
{% endcode %}

### Network troubleshooting

{% code overflow="wrap" %}
```bash
# Test connectivity to other ZeroTier devices
/ping 192.168.192.20 interface=zerotier1

# Check routing table
/ip route print where dst-address~"192.168.192"

# View logs
/log print where message~"zerotier"

# Check interface status
/interface print stats where type="zerotier"
```
{% endcode %}

### ZeroTier Central management

**From ZeroTier Central web interface:**
1. **View network topology** - See all connected devices
2. **Manage device authorization** - Approve/deny devices
3. **Configure IP assignments** - Set static IPs for devices
4. **Set up managed routes** - Define network routing
5. **Monitor network activity** - View connection statistics

***

## Use cases and scenarios

### Remote office connectivity

**Scenario:** Connect branch offices to main office

**Configuration:**
1. **Main office router** joins ZeroTier network
2. **Branch office routers** join same network  
3. **Configure managed routes** in ZeroTier Central
4. **Set up local routing** on each router

### Remote worker access

**Scenario:** Allow remote workers to access office resources

**Setup:**
1. **Office router** joins company ZeroTier network
2. **Remote workers** install ZeroTier client and join network
3. **Configure firewall rules** to allow appropriate access
4. **Set up DNS** for internal resource resolution

### IoT device management

**Scenario:** Securely manage IoT devices across locations

**Implementation:**
1. **Create dedicated IoT ZeroTier network**
2. **Configure IoT devices** to join ZeroTier network
3. **Set up management server** on ZeroTier network
4. **Implement network segmentation** for security

***

## Security considerations

### Network access control

{% code overflow="wrap" %}
```bash
# Implement strict firewall rules for ZeroTier
/ip firewall filter add chain=input action=drop in-interface=zerotier1 dst-port=22,23,21,80,8291 comment="Block management access from ZeroTier"

# Allow only specific services
/ip firewall filter add chain=input action=accept in-interface=zerotier1 dst-port=53 protocol=udp comment="Allow DNS from ZeroTier"
```
{% endcode %}

### Network segmentation

{% code overflow="wrap" %}
```bash
# Separate ZeroTier traffic from local networks
/ip firewall filter add chain=forward action=drop in-interface=zerotier1 out-interface=bridge1 comment="Block ZeroTier to LAN"
/ip firewall filter add chain=forward action=accept in-interface=zerotier1 out-interface=bridge1 dst-address=192.168.1.100 comment="Allow specific server"
```
{% endcode %}

### Monitoring and logging

{% code overflow="wrap" %}
```bash
# Log ZeroTier connections
/ip firewall filter add chain=input action=log in-interface=zerotier1 log-prefix="ZT-INPUT"
/ip firewall filter add chain=forward action=log in-interface=zerotier1 log-prefix="ZT-FORWARD"

# Monitor for unusual activity
/log print where message~"ZT-" and message~"192.168.192"
```
{% endcode %}

***

<details>

<summary>Show complete ZeroTier setup</summary>

{% code overflow="wrap" %}
```bash
# 1. Enable ZeroTier service
/zerotier set enabled=yes

# 2. Join ZeroTier network (replace with your Network ID)
/zerotier join network=8056c2e21c000001 name=company-network

# 3. Configure IP address (after authorization in ZeroTier Central)
/ip address add address=192.168.192.10/24 interface=zerotier1

# 4. Configure firewall rules
/ip firewall filter add chain=input action=accept protocol=udp dst-port=9993 comment="ZeroTier protocol"
/ip firewall filter add chain=input action=accept in-interface=zerotier1 comment="ZeroTier network access"
/ip firewall filter add chain=forward action=accept in-interface=zerotier1 comment="ZeroTier forward in"
/ip firewall filter add chain=forward action=accept out-interface=zerotier1 comment="ZeroTier forward out"

# 5. Configure NAT for internet access (optional)
/ip firewall nat add chain=srcnat src-address=192.168.192.0/24 out-interface=ether1 action=masquerade comment="ZeroTier NAT"

# 6. Add routes for local network (configure in ZeroTier Central)
# In ZeroTier Central, add managed route: 192.168.1.0/24 via 192.168.192.10
```
{% endcode %}

</details>

## Performance optimization

### Optimize ZeroTier performance

{% code overflow="wrap" %}
```bash
# Monitor ZeroTier interface performance
/interface monitor-traffic interface=zerotier1

# Check peer connection quality
/zerotier peer print

# Test direct connectivity (bypass relays)
/ping 192.168.192.20 interface=zerotier1 count=10
```
{% endcode %}

### Network optimization tips

1. **Direct connections** - Ensure devices can establish direct P2P connections
2. **Firewall configuration** - Allow UDP 9993 for optimal performance  
3. **Geographic placement** - Consider ZeroTier root server locations
4. **Network sizing** - Keep networks under 100 devices for best performance
5. **Route optimization** - Use managed routes efficiently

***

## Troubleshooting common issues

### Connection problems

**ZeroTier not connecting:**
- Check internet connectivity
- Verify UDP 9993 is allowed through firewall
- Confirm network ID is correct
- Check device authorization in ZeroTier Central

**Poor performance:**
- Check if direct P2P connection is established
- Monitor for relay usage (indicates NAT/firewall issues)
- Test network latency and packet loss
- Consider ZeroTier root server proximity

### Network access issues

**Can't reach other devices:**
- Verify IP assignments in ZeroTier Central
- Check managed routes configuration
- Test basic connectivity with ping
- Review local firewall rules

**Intermittent connectivity:**
- Check for NAT session timeouts
- Monitor ZeroTier peer connection status
- Verify network stability
- Check for IP address conflicts

### Diagnostic commands

{% code overflow="wrap" %}
```bash
# Comprehensive ZeroTier diagnostics
/zerotier print detail
/zerotier peer print detail
/interface print stats where type="zerotier"
/ip route print where gateway~"zerotier"
/log print where message~"zerotier"
```
{% endcode %}

