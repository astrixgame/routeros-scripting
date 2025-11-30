---
description: This section covers static route configuration for manual routing control and network connectivity.
icon: map-pin
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

# Static

{% hint style="info" %}
Static routes provide manual control over packet forwarding, essential for connecting networks, creating redundant paths, and implementing specific routing policies in RouterOS environments.
{% endhint %}

In WinBox you can configure static routes in **Routing -> Routes**, or you can use terminal with command `/ip route`

Static routing gives administrators complete control over traffic flow and is essential for basic network connectivity, backup paths, and policy routing.

***

## Static routing fundamentals

### How static routes work

**Route components:**
- **Destination network** - Target network or host (dst-address)
- **Gateway** - Next-hop router IP address
- **Interface** - Outbound interface (optional but recommended)
- **Distance** - Administrative preference (lower = preferred)
- **Scope** - Route validity scope for monitoring

**Route selection criteria:**
1. **Longest prefix match** - Most specific route wins
2. **Administrative distance** - Lower distance preferred
3. **Metric** - Used when distance is equal
4. **Route age** - Newer routes preferred when all else equal

### Administrative distance values

Standard RouterOS distance values for route preference:

{% code overflow="wrap" %}
```bash
# Default administrative distance values in RouterOS v7+
# Connected interfaces: 0 (always preferred)
# Static routes: 1 (default for manually configured routes)
# OSPF: 110 (Open Shortest Path First)
# RIP: 120 (Routing Information Protocol)
# BGP: 200 (Border Gateway Protocol - lowest preference)

# Custom distance examples
/ip route add dst-address=192.168.1.0/24 gateway=10.0.0.1 distance=1 comment="Primary path"
/ip route add dst-address=192.168.1.0/24 gateway=10.0.0.2 distance=5 comment="Backup path"
/ip route add dst-address=192.168.1.0/24 gateway=10.0.0.3 distance=10 comment="Last resort path"

# View distance values for installed routes
/ip route print detail where active=yes
connected = 0     # Directly connected networks
static = 1        # Static routes  
OSPF = 110        # OSPF routes
RIP = 120         # RIP routes  
BGP = 200         # eBGP routes
unknown = 255     # Invalid routes
```
{% endcode %}

***

## Basic static route configuration

### Default route (gateway of last resort)

Essential for internet connectivity:

{% code overflow="wrap" %}
```bash
# Basic default route
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="Default route to internet"

# Default route with specific interface
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 interface=ether1 distance=1 comment="Default via ether1"

# Check default route installation
/ip route print where dst-address=0.0.0.0/0
```
{% endcode %}

### Host routes

Routes to specific hosts:

{% code overflow="wrap" %}
```bash
# Route to specific server
/ip route add dst-address=192.168.100.50/32 gateway=192.168.1.1 interface=ether2 comment="Route to database server"

# Route to management interface
/ip route add dst-address=10.0.0.1/32 gateway=192.168.1.10 comment="Route to switch management"

# Route through VPN tunnel
/ip route add dst-address=172.16.1.10/32 gateway=ipsec-tunnel1 comment="Remote server via VPN"
```
{% endcode %}

### Network routes

Routes to subnets and network ranges:

{% code overflow="wrap" %}
```bash
# Route to branch office network
/ip route add dst-address=192.168.2.0/24 gateway=192.168.1.100 interface=ether3 distance=1 comment="Branch office LAN"

# Route to DMZ network
/ip route add dst-address=172.16.10.0/24 gateway=192.168.1.200 interface=ether4 distance=1 comment="DMZ servers"

# Route to guest network
/ip route add dst-address=10.10.0.0/16 gateway=192.168.1.50 comment="Guest network segment"
```
{% endcode %}

***

## Advanced static routing

### Multiple paths and redundancy

Configure primary and backup routes for high availability:

{% code overflow="wrap" %}
```bash
# Primary path to remote network
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.1 distance=1 \
    target-scope=30 comment="Primary path to remote site"

# Backup path with higher distance (automatic failover)
/ip route add dst-address=192.168.10.0/24 gateway=192.168.2.1 distance=5 \
    target-scope=30 comment="Backup path to remote site"

# Multiple default routes for internet redundancy
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 \
    target-scope=30 comment="Primary ISP"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 \
    target-scope=30 comment="Backup ISP"

# Load balancing with equal distance (ECMP)
/ip route add dst-address=192.168.20.0/24 gateway=192.168.1.1 distance=1 comment="Load balanced path 1"
/ip route add dst-address=192.168.20.0/24 gateway=192.168.1.2 distance=1 comment="Load balanced path 2"

# Verify route installation and active paths
/ip route print where dst-address=192.168.10.0/24
/ip route print where active=yes and dst-address=0.0.0.0/0
```
{% endcode %}

### Gateway monitoring and health checks

Automatic route manipulation based on gateway health:

{% code overflow="wrap" %}
```bash
# Route with gateway reachability checking
/ip route add dst-address=192.168.50.0/24 gateway=192.168.1.10 \
    distance=1 target-scope=30 comment="Route with health monitoring"

# Configure netwatch to monitor gateway health
/tool netwatch add host=192.168.1.10 timeout=2s interval=10s \
    up-script="/log info \"Gateway 192.168.1.10 is UP\"" \
    down-script="/log warning \"Gateway 192.168.1.10 is DOWN\"; \
                /ip route disable [find gateway=192.168.1.10]" \
    comment="Monitor primary gateway"

# Alternative: Use scope for automatic route deactivation
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.1 \
    distance=1 target-scope=10 comment="Route deactivates if gateway unreachable"
    
# Scope values:
# target-scope=10: Check if gateway is directly reachable
# target-scope=30: Check if gateway responds to ping (default)
# target-scope=200: Use gateway even if unreachable (not recommended)
```
{% endcode %}

### Route summarization

Efficient routing table management:

{% code overflow="wrap" %}
```bash
# Instead of multiple specific routes:
# /ip route add dst-address=192.168.1.0/24 gateway=10.0.0.1
# /ip route add dst-address=192.168.2.0/24 gateway=10.0.0.1  
# /ip route add dst-address=192.168.3.0/24 gateway=10.0.0.1
# /ip route add dst-address=192.168.4.0/24 gateway=10.0.0.1

# Use summarized route (covers 192.168.0.0 to 192.168.255.255)
/ip route add dst-address=192.168.0.0/16 gateway=10.0.0.1 \
    comment="Summarized route for all 192.168.x.x networks"

# Hierarchical routing with summary and specific routes
/ip route add dst-address=10.0.0.0/8 gateway=172.16.1.1 distance=10 \
    comment="Summary route for all 10.x.x.x networks"
/ip route add dst-address=10.1.0.0/16 gateway=172.16.1.2 distance=1 \
    comment="Specific route for 10.1.x.x (preferred over summary)"

# Verify longest prefix match behavior
/tool traceroute 10.1.1.1  # Should use specific route
/tool traceroute 10.2.1.1  # Should use summary route
```
{% endcode %}

### Route monitoring and health checking

Monitor route validity with scope and target-scope:

{% code overflow="wrap" %}
```bash
# Route with gateway health monitoring
/ip route add dst-address=192.168.50.0/24 gateway=192.168.1.10 \
    target-scope=30 distance=1 comment="Monitored route to server network"

# Multiple routes with different scopes
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 \
    target-scope=30 distance=1 comment="Primary internet - monitored"
    
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 \
    distance=5 comment="Backup internet - static"

# Check route status
/ip route print detail where dst-address=0.0.0.0/0
```
{% endcode %}

### Conditional routing with routing tables

Use routing marks for policy-based routing:

{% code overflow="wrap" %}
```bash
# Create custom routing table
/routing table add name=guest-internet fib

# Add routes to custom table
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 \
    routing-table=guest-internet distance=1 comment="Guest internet via secondary WAN"

# Route for specific source networks (requires mangle marking)
/ip route add dst-address=8.8.8.8/32 gateway=203.0.113.1 \
    routing-table=main distance=1 comment="DNS via primary WAN"
```
{% endcode %}

***

## Site-to-site connectivity

### VPN tunnel routing

Configure routes through various VPN types:

{% code overflow="wrap" %}
```bash
# Routes through IPSec tunnel
/ip route add dst-address=192.168.100.0/24 gateway=ipsec-peer1 distance=1 comment="Site A via IPSec"

# Routes through OpenVPN tunnel  
/ip route add dst-address=192.168.200.0/24 gateway=10.8.0.1 interface=ovpn-server1 distance=1 comment="Site B via OpenVPN"

# Routes through WireGuard
/ip route add dst-address=192.168.300.0/24 gateway=10.13.13.1 interface=wireguard1 distance=1 comment="Site C via WireGuard"

# Routes through GRE tunnel
/ip route add dst-address=192.168.400.0/24 gateway=172.16.0.2 interface=gre-tunnel1 distance=1 comment="Site D via GRE"
```
{% endcode %}

### Hub and spoke topology

Central site routing configuration:

{% code overflow="wrap" %}
```bash
# Hub site routes to all spokes
/ip route add dst-address=192.168.10.0/24 gateway=ipsec-spoke1 distance=1 comment="Spoke 1 - Sales office"
/ip route add dst-address=192.168.20.0/24 gateway=ipsec-spoke2 distance=1 comment="Spoke 2 - Branch office"  
/ip route add dst-address=192.168.30.0/24 gateway=ipsec-spoke3 distance=1 comment="Spoke 3 - Remote site"

# Spoke site routes (example from spoke 1)
/ip route add dst-address=0.0.0.0/0 gateway=ipsec-hub distance=1 comment="All traffic via hub"

# Or specific routes for spoke-to-spoke via hub
/ip route add dst-address=192.168.20.0/24 gateway=ipsec-hub distance=1 comment="Spoke 2 via hub"
/ip route add dst-address=192.168.30.0/24 gateway=ipsec-hub distance=1 comment="Spoke 3 via hub"
```
{% endcode %}

### Meshed connectivity

Full or partial mesh routing:

{% code overflow="wrap" %}
```bash
# Site A routes in full mesh
/ip route add dst-address=192.168.20.0/24 gateway=ipsec-site-b distance=1 comment="Direct to Site B"
/ip route add dst-address=192.168.30.0/24 gateway=ipsec-site-c distance=1 comment="Direct to Site C"

# Backup routes through hub (higher distance)
/ip route add dst-address=192.168.20.0/24 gateway=ipsec-hub distance=10 comment="Site B via hub backup"
/ip route add dst-address=192.168.30.0/24 gateway=ipsec-hub distance=10 comment="Site C via hub backup"
```
{% endcode %}

***

## Multi-WAN static routing

### Dual WAN configuration

Configure primary and backup internet connections:

{% code overflow="wrap" %}
```bash
# Primary WAN (lower distance)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 \
    interface=ether1 target-scope=10 distance=1 comment="Primary WAN"

# Backup WAN (higher distance)  
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 \
    interface=ether2 distance=5 comment="Backup WAN"

# Health checking routes for monitoring
/ip route add dst-address=8.8.8.8/32 gateway=203.0.113.1 \
    interface=ether1 scope=30 distance=1 comment="Primary WAN health check"
    
/ip route add dst-address=1.1.1.1/32 gateway=203.0.114.1 \
    interface=ether2 scope=30 distance=1 comment="Backup WAN health check"
```
{% endcode %}

### Load balancing static routes

Distribute traffic across multiple connections:

{% code overflow="wrap" %}
```bash
# Equal cost routes for load balancing
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 comment="WAN1 for load balancing"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=1 comment="WAN2 for load balancing"

# Specific routes for different traffic types
/ip route add dst-address=8.8.8.8/32 gateway=203.0.113.1 distance=1 comment="DNS via WAN1"
/ip route add dst-address=1.1.1.1/32 gateway=203.0.114.1 distance=1 comment="DNS via WAN2"

# Policy routes using routing tables (requires mangle)
/routing table add name=wan1-table fib
/routing table add name=wan2-table fib

/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=wan1-table distance=1
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=wan2-table distance=1
```
{% endcode %}

***

## IPv6 static routing

### Basic IPv6 routes

{% code overflow="wrap" %}
```bash
# IPv6 default route
/ipv6 route add dst-address=::/0 gateway=2001:db8::1 interface=ether1 distance=1 comment="IPv6 default route"

# IPv6 network routes
/ipv6 route add dst-address=2001:db8:100::/64 gateway=2001:db8:1::1 distance=1 comment="IPv6 branch network"

# IPv6 host route
/ipv6 route add dst-address=2001:db8:50::10/128 gateway=2001:db8:1::10 comment="IPv6 server route"

# Check IPv6 routes
/ipv6 route print
```
{% endcode %}

### IPv6 dual-stack routing

{% code overflow="wrap" %}
```bash
# Parallel IPv4 and IPv6 routes
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.1 distance=1 comment="IPv4 to branch"
/ipv6 route add dst-address=2001:db8:10::/64 gateway=2001:db8:1::1 distance=1 comment="IPv6 to branch"

# IPv6 over IPv4 tunnel routes
/ipv6 route add dst-address=2001:db8:200::/64 gateway=2001:db8:tunnel::1 interface=6to4-tunnel1 comment="IPv6 via tunnel"
```
{% endcode %}

***

## Static route monitoring

### Route verification commands

{% code overflow="wrap" %}
```bash
# Display all routes
/ip route print

# Show only static routes
/ip route print where static=yes

# Show routes to specific destination
/ip route print where dst-address~"192.168.1"

# Display detailed route information
/ip route print detail where dst-address=0.0.0.0/0

# Show route statistics
/ip route print stats
```
{% endcode %}

### Connectivity testing

{% code overflow="wrap" %}
```bash
# Test basic connectivity
/ping 192.168.10.1

# Trace route path
/tool traceroute 192.168.10.1

# Test specific interface
/ping 8.8.8.8 interface=ether1

# Monitor route resolution
/tool netwatch add host=192.168.10.1 timeout=1s interval=10s
```
{% endcode %}

### Route troubleshooting

{% code overflow="wrap" %}
```bash
# Check route conflicts
/ip route print where dst-address=192.168.1.0/24

# Verify gateway reachability
/ping 192.168.1.1

# Check interface status
/interface print where name=ether1

# Monitor routing logs
/log print where topics~"route"

# Test route changes
/ip route disable [find dst-address=0.0.0.0/0 and distance=1]
/ping 8.8.8.8
/ip route enable [find dst-address=0.0.0.0/0 and distance=1]
```
{% endcode %}

***

## Route management and maintenance

### Route organization

{% code overflow="wrap" %}
```bash
# Use descriptive comments
/ip route add dst-address=192.168.10.0/24 gateway=192.168.1.1 comment="Sales office - Primary link"

# Group related routes with consistent naming
/ip route add dst-address=192.168.20.0/24 gateway=192.168.1.2 comment="Branch-A - Main office link"
/ip route add dst-address=192.168.21.0/24 gateway=192.168.1.2 comment="Branch-A - WiFi network"
/ip route add dst-address=192.168.22.0/24 gateway=192.168.1.2 comment="Branch-A - Guest network"

# Use consistent distance values
# Distance 1: Primary paths
# Distance 5: Secondary paths  
# Distance 10: Backup paths
```
{% endcode %}

### Bulk route operations

{% code overflow="wrap" %}
```bash
# Disable multiple routes
/ip route disable [find comment~"Branch-A"]

# Enable routes by gateway
/ip route enable [find gateway=192.168.1.1]

# Remove old routes
/ip route remove [find comment~"old-connection"]

# Export route configuration
/ip route export file=static-routes-backup
```
{% endcode %}

***

## Performance considerations

### Route table optimization

{% code overflow="wrap" %}
```bash
# Check route table size
/ip route print count-only

# Monitor route lookup performance
/system resource print

# Use specific interfaces in routes for better performance
/ip route add dst-address=192.168.1.0/24 gateway=192.168.0.1 interface=ether2

# Avoid unnecessary routes - let dynamic routing handle when possible
```
{% endcode %}

### Hardware acceleration

{% code overflow="wrap" %}
```bash
# Check if routes use hardware acceleration
/ip route print detail where hw=yes

# Verify interface acceleration capabilities
/interface print detail where name=ether1

# Monitor hardware vs software forwarding
/interface monitor-traffic interface=ether1 duration=10
```
{% endcode %}

***

<details>

<summary>Show complete static routing setup for branch office</summary>

{% code overflow="wrap" %}
```bash
# Complete static routing configuration for branch office
# Scenario: Branch office with dual WAN and site-to-site VPN

# 1. Local network identification
# LAN: 192.168.10.0/24
# WAN1 (Primary): ether1, Gateway: 203.0.113.1
# WAN2 (Backup): ether2, Gateway: 203.0.114.1  
# VPN to HQ: 192.168.1.0/24

# 2. Default routes (internet access)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 interface=ether1 \
    target-scope=10 distance=1 comment="Primary internet via WAN1"
    
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 interface=ether2 \
    distance=5 comment="Backup internet via WAN2"

# 3. Site-to-site VPN routes
/ip route add dst-address=192.168.1.0/24 gateway=ipsec-hq distance=1 \
    comment="HQ network via IPSec tunnel"
    
# 4. Corporate server routes via VPN
/ip route add dst-address=10.0.0.0/8 gateway=ipsec-hq distance=1 \
    comment="Corporate servers via HQ"

# 5. Management and monitoring routes
/ip route add dst-address=8.8.8.8/32 gateway=203.0.113.1 interface=ether1 \
    scope=30 distance=1 comment="Primary WAN health check"
    
/ip route add dst-address=1.1.1.1/32 gateway=203.0.114.1 interface=ether2 \
    scope=30 distance=1 comment="Backup WAN health check"

# 6. Local service routes
/ip route add dst-address=192.168.10.100/32 gateway=192.168.10.100 interface=bridge \
    distance=1 comment="Local file server"
    
/ip route add dst-address=192.168.10.200/32 gateway=192.168.10.200 interface=bridge \
    distance=1 comment="Local printer"

# 7. Guest network route (if separate)
/ip route add dst-address=192.168.20.0/24 gateway=192.168.10.1 interface=bridge \
    distance=1 comment="Guest network segment"

# 8. Backup routes for VPN traffic via internet
/ip route add dst-address=192.168.1.0/24 gateway=203.0.113.1 interface=ether1 \
    distance=20 comment="HQ via internet backup path"

# 9. Health monitoring setup
/tool netwatch add host=192.168.1.1 timeout=2s interval=30s \
    comment="Monitor HQ connectivity"
    
/tool netwatch add host=8.8.8.8 timeout=2s interval=30s \
    comment="Monitor primary internet"

# 10. IPv6 routes (if dual-stack)
/ipv6 route add dst-address=::/0 gateway=2001:db8::1 interface=ether1 \
    distance=1 comment="IPv6 default route"
    
/ipv6 route add dst-address=2001:db8:1::/64 gateway=2001:db8:tunnel::1 \
    interface=ipsec-hq distance=1 comment="HQ IPv6 network"
```
{% endcode %}

</details>

## Static routing best practices

### Design principles

1. **Keep it simple** - Use static routes only where necessary
2. **Plan for redundancy** - Always have backup paths
3. **Use monitoring** - Implement health checking for critical routes
4. **Document thoroughly** - Use meaningful comments
5. **Test failover** - Verify backup routes work correctly

### Common mistakes to avoid

1. **Routing loops** - Ensure no circular routing paths
2. **Missing return routes** - Verify bidirectional connectivity
3. **Incorrect distances** - Plan administrative distance carefully
4. **No monitoring** - Critical routes should be health-checked
5. **Poor documentation** - Always comment route purposes

### Maintenance procedures

1. **Regular review** - Audit static routes periodically
2. **Change control** - Document all route changes
3. **Testing procedures** - Test before implementing in production
4. **Backup configurations** - Export and save route configs
5. **Monitor performance** - Watch for routing-related issues

