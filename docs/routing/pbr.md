---
description: Policy-Based Routing (PBR) allows routing decisions based on criteria other than destination address, enabling advanced traffic engineering and multi-WAN scenarios.
icon: flow-arrow
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

# Policy-Based Routing (PBR)

{% hint style="info" %}
Policy-Based Routing enables sophisticated traffic steering based on source addresses, protocols, applications, or other packet characteristics, going beyond traditional destination-based routing.
{% endhint %}

PBR in RouterOS v7+ uses mangle rules to mark packets and routing tables to direct marked traffic, providing flexible traffic engineering capabilities for complex network requirements.

***

## PBR fundamentals

### How PBR works

**Traditional routing:**
- Routes packets based solely on destination IP address
- Uses longest prefix match in single routing table
- Limited flexibility for traffic engineering

**Policy-based routing:**
- Routes packets based on multiple criteria (source, protocol, port, etc.)
- Uses packet marking and multiple routing tables
- Enables sophisticated traffic steering policies

**PBR components in RouterOS:**
- **Mangle rules** - Mark packets based on criteria
- **Routing tables** - Separate routing tables for different policies
- **Routing marks** - Connect mangle marks to routing tables
- **Route rules** - Direct marked traffic to appropriate routing table

### PBR use cases

{% code overflow="wrap" %}
```bash
# Common PBR scenarios:
# 1. Multi-WAN load balancing and failover
# 2. Source-based routing (different subnets via different gateways)
# 3. Application-based routing (VoIP, video streaming priorities)
# 4. Quality of Service (QoS) implementation  
# 5. Security policies (isolate guest networks)
# 6. Bandwidth optimization (cache servers, CDN selection)
# 7. Compliance requirements (data sovereignty, audit trails)
```
{% endcode %}

***

## Basic PBR configuration

### Source-based routing

Route traffic based on source IP address:

{% code overflow="wrap" %}
```bash
# Scenario: Route different subnets through different ISPs
# LAN1 (192.168.1.0/24) -> ISP1 (203.0.113.1)
# LAN2 (192.168.2.0/24) -> ISP2 (203.0.114.1)

# Step 1: Create routing tables
/routing table add name=isp1-table fib comment="Routing table for ISP1"
/routing table add name=isp2-table fib comment="Routing table for ISP2"

# Step 2: Add routes to routing tables
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=isp1-table comment="ISP1 default route"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=isp2-table comment="ISP2 default route"

# Step 3: Create mangle rules to mark traffic
/ip firewall mangle add chain=prerouting src-address=192.168.1.0/24 \
    action=mark-routing new-routing-mark=to-isp1 comment="Mark LAN1 traffic for ISP1"

/ip firewall mangle add chain=prerouting src-address=192.168.2.0/24 \
    action=mark-routing new-routing-mark=to-isp2 comment="Mark LAN2 traffic for ISP2"

# Step 4: Create routing rules to use marked routing tables
/ip route rule add src-address=192.168.1.0/24 action=lookup-only-in-table \
    table=isp1-table comment="LAN1 uses ISP1 table"

/ip route rule add src-address=192.168.2.0/24 action=lookup-only-in-table \
    table=isp2-table comment="LAN2 uses ISP2 table"

# Step 5: Verify configuration
/ip route print where routing-table=isp1-table
/ip route print where routing-table=isp2-table
/ip firewall mangle print where chain=prerouting
```
{% endcode %}

### Protocol-based routing

Route different protocols through different paths:

{% code overflow="wrap" %}
```bash
# Scenario: Route HTTP/HTTPS through ISP1, other traffic through ISP2
# Create routing tables (if not already created)
/routing table add name=web-traffic fib comment="HTTP/HTTPS traffic"
/routing table add name=other-traffic fib comment="Non-web traffic"

# Add routes to tables
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=web-traffic comment="ISP1 for web"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=other-traffic comment="ISP2 for other"

# Mark HTTP traffic (port 80)
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80 \
    action=mark-routing new-routing-mark=web-mark comment="Mark HTTP traffic"

# Mark HTTPS traffic (port 443)  
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=443 \
    action=mark-routing new-routing-mark=web-mark comment="Mark HTTPS traffic"

# Mark other traffic (everything else)
/ip firewall mangle add chain=prerouting action=mark-routing \
    new-routing-mark=other-mark comment="Mark non-web traffic"

# Create routing rules
/ip route rule add routing-mark=web-mark action=lookup-only-in-table \
    table=web-traffic comment="Web traffic routing rule"

/ip route rule add routing-mark=other-mark action=lookup-only-in-table \
    table=other-traffic comment="Other traffic routing rule"
```
{% endcode %}

***

## Advanced PBR scenarios

### Multi-WAN load balancing with PBR

Intelligent load balancing across multiple ISPs:

{% code overflow="wrap" %}
```bash
# Advanced multi-WAN setup with weighted load balancing
# ISP1: 100Mbps, ISP2: 50Mbps (2:1 ratio desired)

# Create routing tables
/routing table add name=isp1 fib comment="ISP1 100Mbps"
/routing table add name=isp2 fib comment="ISP2 50Mbps"

# Add default routes
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=isp1
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=isp2

# Create connection marks for persistent routing
/ip firewall mangle add chain=prerouting in-interface=bridge \
    connection-state=new action=mark-connection \
    new-connection-mark=isp1-conn probability=66.7 comment="66.7% to ISP1"

/ip firewall mangle add chain=prerouting in-interface=bridge \
    connection-state=new connection-mark=!isp1-conn action=mark-connection \
    new-connection-mark=isp2-conn comment="Remaining 33.3% to ISP2"

# Mark routing based on connection marks
/ip firewall mangle add chain=prerouting connection-mark=isp1-conn \
    action=mark-routing new-routing-mark=isp1-mark

/ip firewall mangle add chain=prerouting connection-mark=isp2-conn \
    action=mark-routing new-routing-mark=isp2-mark

# Routing rules
/ip route rule add routing-mark=isp1-mark action=lookup-only-in-table table=isp1
/ip route rule add routing-mark=isp2-mark action=lookup-only-in-table table=isp2

# Failover routes in main table
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 \
    target-scope=30 comment="Primary failover route"
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 \
    target-scope=30 comment="Backup failover route"
```
{% endcode %}

### Application-specific routing

Route specific applications through optimal paths:

{% code overflow="wrap" %}
```bash
# Route VoIP through low-latency ISP, bulk data through high-bandwidth ISP
# ISP1: Low latency for VoIP (203.0.113.1)
# ISP2: High bandwidth for data (203.0.114.1)

# Create application-specific routing tables
/routing table add name=voip-table fib comment="Low latency for VoIP"
/routing table add name=data-table fib comment="High bandwidth for data"

# Add routes
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=voip-table
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=data-table

# Mark VoIP traffic (SIP and RTP)
/ip firewall mangle add chain=prerouting protocol=udp dst-port=5060 \
    action=mark-routing new-routing-mark=voip-mark comment="SIP signaling"

/ip firewall mangle add chain=prerouting protocol=udp dst-port=10000-20000 \
    action=mark-routing new-routing-mark=voip-mark comment="RTP audio"

# Mark bulk data traffic (FTP, HTTP downloads)
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=21 \
    action=mark-routing new-routing-mark=data-mark comment="FTP control"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=20 \
    action=mark-routing new-routing-mark=data-mark comment="FTP data"

# Large downloads (based on connection bytes)
/ip firewall mangle add chain=forward protocol=tcp \
    connection-bytes=50000000-0 action=mark-routing \
    new-routing-mark=data-mark comment="Large downloads via high-bandwidth ISP"

# Routing rules
/ip route rule add routing-mark=voip-mark action=lookup-only-in-table table=voip-table
/ip route rule add routing-mark=data-mark action=lookup-only-in-table table=data-table
```
{% endcode %}

### Geographic or time-based routing

Dynamic routing based on conditions:

{% code overflow="wrap" %}
```bash
# Time-based routing: Business hours vs after hours
# Business hours (8 AM - 6 PM): Use primary ISP
# After hours: Use backup ISP to save costs

# Create time-based mangle rules
/ip firewall mangle add chain=prerouting time=8h-18h,mon,tue,wed,thu,fri \
    action=mark-routing new-routing-mark=business-hours \
    comment="Business hours routing"

/ip firewall mangle add chain=prerouting time=18h-23h59m,mon,tue,wed,thu,fri \
    action=mark-routing new-routing-mark=after-hours \
    comment="After hours routing - weekdays"

/ip firewall mangle add chain=prerouting time=0h-23h59m,sat,sun \
    action=mark-routing new-routing-mark=after-hours \
    comment="After hours routing - weekends"

# Create routing tables
/routing table add name=business-table fib
/routing table add name=afterhours-table fib

# Add routes (business hours uses primary ISP)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=business-table
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=afterhours-table

# Routing rules
/ip route rule add routing-mark=business-hours action=lookup-only-in-table table=business-table
/ip route rule add routing-mark=after-hours action=lookup-only-in-table table=afterhours-table
```
{% endcode %}

***

## PBR with QoS integration

### Priority-based routing

Combine PBR with QoS for optimal traffic handling:

{% code overflow="wrap" %}
```bash
# Integrate PBR with Quality of Service
# High priority traffic: VoIP, video conferencing
# Normal priority: Web browsing, email
# Low priority: File transfers, backups

# Mark packets with both routing and priority
# High priority VoIP traffic
/ip firewall mangle add chain=prerouting protocol=udp dst-port=5060,5061 \
    action=mark-packet new-packet-mark=voip-packet passthrough=yes \
    comment="Mark VoIP packets"
/ip firewall mangle add chain=prerouting packet-mark=voip-packet \
    action=mark-routing new-routing-mark=high-priority \
    comment="Route VoIP via low-latency path"

# Normal priority web traffic  
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80,443 \
    connection-bytes=0-10000000 action=mark-packet \
    new-packet-mark=web-packet passthrough=yes comment="Mark web browsing"
/ip firewall mangle add chain=prerouting packet-mark=web-packet \
    action=mark-routing new-routing-mark=normal-priority

# Low priority bulk transfers
/ip firewall mangle add chain=prerouting protocol=tcp \
    connection-bytes=10000000-0 action=mark-packet \
    new-packet-mark=bulk-packet passthrough=yes comment="Mark bulk transfers"
/ip firewall mangle add chain=prerouting packet-mark=bulk-packet \
    action=mark-routing new-routing-mark=low-priority

# Create routing tables with different characteristics
/routing table add name=priority-high fib comment="Low latency path"
/routing table add name=priority-normal fib comment="Balanced path"
/routing table add name=priority-low fib comment="High bandwidth path"

# Add routes to tables
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=priority-high  # Low latency ISP
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=priority-normal # Balanced ISP
/ip route add dst-address=0.0.0.0/0 gateway=203.0.115.1 routing-table=priority-low   # High bandwidth ISP

# Routing rules
/ip route rule add routing-mark=high-priority action=lookup-only-in-table table=priority-high
/ip route rule add routing-mark=normal-priority action=lookup-only-in-table table=priority-normal  
/ip route rule add routing-mark=low-priority action=lookup-only-in-table table=priority-low
```
{% endcode %}

***

## PBR monitoring and troubleshooting

### Monitoring PBR effectiveness

Track PBR performance and utilization:

{% code overflow="wrap" %}
```bash
# Monitor mangle rule hit counters
/ip firewall mangle print stats where chain=prerouting

# Check routing table utilization
/ip route print stats where routing-table=isp1-table
/ip route print stats where routing-table=isp2-table

# Monitor routing rule effectiveness
/ip route rule print

# Test specific routing paths
/tool traceroute 8.8.8.8 src-address=192.168.1.100  # Should use ISP1
/tool traceroute 8.8.8.8 src-address=192.168.2.100  # Should use ISP2

# Connection tracking for persistent routing
/ip firewall connection print where connection-mark~"isp"

# Script to monitor PBR balance
:local totalISP1 [/ip firewall mangle get [find new-routing-mark="to-isp1"] bytes];
:local totalISP2 [/ip firewall mangle get [find new-routing-mark="to-isp2"] bytes];
:local total ($totalISP1 + $totalISP2);

:if ($total > 0) do={
    :local isp1Percent (($totalISP1 * 100) / $total);
    :local isp2Percent (($totalISP2 * 100) / $total);
    
    /log info ("PBR Distribution - ISP1: " . $isp1Percent . "% ISP2: " . $isp2Percent . "%");
};
```
{% endcode %}

### Troubleshooting PBR issues

Common problems and diagnostic steps:

{% code overflow="wrap" %}
```bash
# Debug PBR routing decisions
# 1. Check if mangle rules are matching traffic
/ip firewall mangle print stats where chain=prerouting

# 2. Verify routing tables have correct routes
/ip route print where routing-table=isp1-table
/ip route print where routing-table=isp2-table

# 3. Check routing rules are properly configured
/ip route rule print

# 4. Test with specific source addresses
/ping 8.8.8.8 src-address=192.168.1.100 count=3
/ping 8.8.8.8 src-address=192.168.2.100 count=3

# 5. Check connection tracking for persistent sessions
/ip firewall connection print where connection-mark~"isp" and protocol=tcp

# 6. Verify NAT rules for each ISP
/ip firewall nat print where chain=srcnat

# 7. Monitor route changes and errors
/log print where topics~"route,firewall"

# 8. Test failover behavior
/ip route disable [find gateway=203.0.113.1 and routing-table=isp1-table]
# Test connectivity, then re-enable
/ip route enable [find gateway=203.0.113.1 and routing-table=isp1-table]

# 9. Check for asymmetric routing issues
/tool traceroute 8.8.8.8 src-address=192.168.1.100
# Compare with return path if possible
```
{% endcode %}

***

## PBR best practices

### Design principles

1. **Keep it simple** - Start with basic scenarios before adding complexity
2. **Document thoroughly** - PBR configurations can become complex quickly
3. **Test extensively** - Verify behavior under various conditions
4. **Plan for failover** - Ensure backup paths exist for all scenarios
5. **Monitor actively** - Track PBR effectiveness and adjust as needed

### Performance considerations

1. **Rule ordering** - Place most frequently matched rules first
2. **Minimize complexity** - Avoid overly complex mangle rule chains
3. **Use connection marking** - Maintain session persistence efficiently
4. **Optimize routing tables** - Keep routing tables focused and minimal
5. **Regular maintenance** - Review and update PBR policies regularly

### Security considerations

1. **Validate source addresses** - Prevent source address spoofing
2. **Secure routing tables** - Protect routing table integrity
3. **Monitor for abuse** - Watch for unusual traffic patterns
4. **Access control** - Limit administrative access to PBR configuration
5. **Audit regularly** - Review PBR policies for security implications

***

## Complete PBR examples

### Enterprise multi-WAN setup

{% code overflow="wrap" %}
```bash
# Complete enterprise PBR configuration
# Scenario: 3 ISPs with different characteristics
# ISP1: Low latency (VoIP, real-time apps)
# ISP2: High bandwidth (downloads, streaming)  
# ISP3: Backup only (failover)

# Create routing tables
/routing table add name=low-latency fib comment="ISP1 - Low latency"
/routing table add name=high-bandwidth fib comment="ISP2 - High bandwidth"
/routing table add name=backup fib comment="ISP3 - Backup"

# Add routes to each table
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 routing-table=low-latency distance=1
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 routing-table=high-bandwidth distance=1
/ip route add dst-address=0.0.0.0/0 gateway=203.0.115.1 routing-table=backup distance=1

# High priority traffic (VoIP, video conferencing)
/ip firewall mangle add chain=prerouting protocol=udp dst-port=5060-5061,10000-20000 \
    action=mark-routing new-routing-mark=realtime comment="VoIP/RTP traffic"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=1720,5060 \
    action=mark-routing new-routing-mark=realtime comment="H.323/SIP"

# Bulk data traffic (downloads, streaming)
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80,443 \
    connection-bytes=10000000-0 action=mark-routing \
    new-routing-mark=bulk-data comment="Large HTTP downloads"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=21,22 \
    action=mark-routing new-routing-mark=bulk-data comment="FTP/SSH file transfers"

# Normal web traffic (load balance between ISP1 and ISP2)
/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80,443 \
    connection-bytes=0-10000000 probability=50 action=mark-routing \
    new-routing-mark=realtime comment="50% small web via ISP1"

/ip firewall mangle add chain=prerouting protocol=tcp dst-port=80,443 \
    connection-bytes=0-10000000 routing-mark=!realtime action=mark-routing \
    new-routing-mark=bulk-data comment="Remaining web via ISP2"

# Routing rules
/ip route rule add routing-mark=realtime action=lookup-only-in-table table=low-latency
/ip route rule add routing-mark=bulk-data action=lookup-only-in-table table=high-bandwidth

# Failover routes in main table (when PBR paths fail)
/ip route add dst-address=0.0.0.0/0 gateway=203.0.113.1 distance=1 target-scope=30
/ip route add dst-address=0.0.0.0/0 gateway=203.0.114.1 distance=2 target-scope=30
/ip route add dst-address=0.0.0.0/0 gateway=203.0.115.1 distance=3 target-scope=30

# Health monitoring for all ISPs
/tool netwatch add host=8.8.8.8 src-address=203.0.113.100 timeout=2s interval=10s \
    comment="ISP1 Health"
/tool netwatch add host=8.8.4.4 src-address=203.0.114.100 timeout=2s interval=10s \
    comment="ISP2 Health"  
/tool netwatch add host=1.1.1.1 src-address=203.0.115.100 timeout=2s interval=10s \
    comment="ISP3 Health"

# NAT for each ISP
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT ISP1"
/ip firewall nat add chain=srcnat out-interface=ether2 action=masquerade comment="NAT ISP2"
/ip firewall nat add chain=srcnat out-interface=ether3 action=masquerade comment="NAT ISP3"
```
{% endcode %}
