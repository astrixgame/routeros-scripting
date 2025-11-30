---
description: This section covers how to configure PPPoE interfaces for internet connections.
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

# PPPoE

{% hint style="info" %}
PPPoE (Point-to-Point Protocol over Ethernet) is commonly used by ISPs for broadband connections. It provides authentication and session management over Ethernet.
{% endhint %}

In WinBox you can configure PPPoE in **Interfaces -> PPPoE Client/Server**, or you can use terminal with commands `/interface pppoe-client` and `/interface pppoe-server`

PPPoE can be used both as a client (connecting to ISP) and as a server (providing internet service).

***

## PPPoE Client configuration

### Basic PPPoE client setup

In WinBox go to **Interfaces -> PPPoE Client** and click <kbd>**+**</kbd>:

* **Name** - Interface name (e.g., pppoe-out1)
* **Interface** - Physical interface connected to ISP (ether1)
* **User** - Username provided by ISP
* **Password** - Password provided by ISP
* **Add Default Route** - Yes (for internet connection)
* **Use Peer DNS** - Yes (use ISP's DNS servers)

{% code overflow="wrap" %}
```bash
# Create basic PPPoE client
/interface pppoe-client add \
    name=pppoe-out1 \
    interface=ether1 \
    user="your_username" \
    password="your_password" \
    add-default-route=yes \
    use-peer-dns=yes \
    disabled=no \
    comment="Main ISP connection"
```
{% endcode %}

### Advanced PPPoE client settings

{% code overflow="wrap" %}
```bash
# Configure advanced PPPoE client options
/interface pppoe-client set pppoe-out1 \
    max-mtu=1500 \
    max-mru=1500 \
    mrru=1600 \
    service-name="" \
    ac-name="" \
    keepalive-timeout=60 \
    dial-on-demand=no \
    use-peer-dns=yes \
    add-default-route=yes \
    default-route-distance=1
```
{% endcode %}

### PPPoE with VLAN (common ISP setup)

Many ISPs require VLAN tagging for PPPoE:

{% code overflow="wrap" %}
```bash
# Create VLAN interface first
/interface vlan add name=vlan-isp vlan-id=35 interface=ether1 comment="ISP VLAN"

# Create PPPoE client on VLAN interface
/interface pppoe-client add \
    name=pppoe-out1 \
    interface=vlan-isp \
    user="your_username" \
    password="your_password" \
    add-default-route=yes \
    use-peer-dns=yes \
    disabled=no \
    comment="PPPoE over VLAN 35"
```
{% endcode %}

***

## PPPoE Server configuration

### Basic PPPoE server setup

{% code overflow="wrap" %}
```bash
# Enable PPPoE server
/interface pppoe-server server set enabled=yes default-profile=default-encryption

# Create IP pool for PPPoE clients
/ip pool add name=pppoe-pool ranges=192.168.100.2-192.168.100.50

# Create PPPoE profile
/ppp profile add name=pppoe-profile \
    local-address=192.168.100.1 \
    remote-address=pppoe-pool \
    dns-server=8.8.8.8,8.8.4.4 \
    use-encryption=yes \
    comment="PPPoE client profile"

# Update server to use custom profile
/interface pppoe-server server set default-profile=pppoe-profile
```
{% endcode %}

### Add PPPoE users

{% code overflow="wrap" %}
```bash
# Add PPPoE users
/ppp secret add name=user1 password=pass1 service=pppoe profile=pppoe-profile comment="PPPoE User 1"
/ppp secret add name=user2 password=pass2 service=pppoe profile=pppoe-profile comment="PPPoE User 2"

# User with specific IP address
/ppp secret add name=admin password=adminpass service=pppoe profile=pppoe-profile remote-address=192.168.100.10 comment="Admin user"

# User with rate limiting
/ppp secret add name=limited password=limitpass service=pppoe profile=pppoe-profile rate-limit=10M/5M comment="Limited user"
```
{% endcode %}

### PPPoE server on specific interface

{% code overflow="wrap" %}
```bash
# Create PPPoE server instance on specific interface
/interface pppoe-server add \
    name=pppoe-server-eth2 \
    interface=ether2 \
    service-name="Internet Access" \
    max-mtu=1480 \
    max-mru=1480 \
    authentication=pap,chap,mschap1,mschap2 \
    keepalive-timeout=10 \
    one-session-per-host=yes \
    max-sessions=50 \
    disabled=no
```
{% endcode %}

***

## ISP-specific configurations

### Common ISP PPPoE setups

**Generic ISP with VLAN:**
{% code overflow="wrap" %}
```bash
# Many European ISPs use this setup
/interface vlan add name=isp-vlan vlan-id=35 interface=ether1
/interface pppoe-client add name=pppoe-isp interface=isp-vlan user="username" password="password" add-default-route=yes use-peer-dns=yes
```
{% endcode %}

**ISP with specific service name:**
{% code overflow="wrap" %}
```bash
# Some ISPs require specific service name
/interface pppoe-client add \
    name=pppoe-isp \
    interface=ether1 \
    user="username" \
    password="password" \
    service-name="internet" \
    add-default-route=yes \
    use-peer-dns=yes
```
{% endcode %}

**Dual-stack PPPoE (IPv4 + IPv6):**
{% code overflow="wrap" %}
```bash
# Configure for IPv6 support
/interface pppoe-client add \
    name=pppoe-isp \
    interface=ether1 \
    user="username" \
    password="password" \
    add-default-route=yes \
    use-peer-dns=yes \
    allow-fast-path=no

# Enable IPv6 on PPPoE interface
/ipv6 settings set forward=yes
/ipv6 dhcp-client add interface=pppoe-isp add-default-route=yes request=prefix
```
{% endcode %}

***

## PPPoE monitoring and troubleshooting

### Monitor PPPoE connection

{% code overflow="wrap" %}
```bash
# Check PPPoE client status
/interface pppoe-client print detail

# Monitor PPPoE connection
/interface pppoe-client monitor pppoe-out1

# Check connection statistics
/interface print stats where name=pppoe-out1

# View active PPP sessions
/ppp active print where service=pppoe
```
{% endcode %}

### PPPoE troubleshooting

{% code overflow="wrap" %}
```bash
# Check PPPoE logs
/log print where topics~"pppoe"

# Test physical connectivity
/ping 8.8.8.8 interface=pppoe-out1

# Check routing
/ip route print where gateway~"pppoe"

# Monitor traffic
/interface monitor-traffic interface=pppoe-out1
```
{% endcode %}

### Common PPPoE issues

{% code overflow="wrap" %}
```bash
# MTU/MSS issues - adjust MTU
/interface pppoe-client set pppoe-out1 max-mtu=1492 max-mru=1492

# Enable MSS clamping for TCP
/ip firewall mangle add chain=forward action=change-mss new-mss=1360 protocol=tcp tcp-flags=syn passthrough=yes out-interface=pppoe-out1

# Authentication failures - check credentials
/interface pppoe-client set pppoe-out1 user="correct_username" password="correct_password"

# VLAN requirements - check with ISP
/interface vlan add name=isp-vlan vlan-id=VLAN_ID interface=ether1
/interface pppoe-client set pppoe-out1 interface=isp-vlan
```
{% endcode %}

***

## PPPoE failover and load balancing

### Dual PPPoE connections

{% code overflow="wrap" %}
```bash
# Primary PPPoE connection
/interface pppoe-client add name=pppoe-primary interface=ether1 user="user1" password="pass1" add-default-route=yes default-route-distance=1

# Backup PPPoE connection  
/interface pppoe-client add name=pppoe-backup interface=ether2 user="user2" password="pass2" add-default-route=yes default-route-distance=2

# Monitor primary connection
/tool netwatch add host=8.8.8.8 interval=30s up-script="/ip route set [find gateway~\"pppoe-primary\"] distance=1" down-script="/ip route set [find gateway~\"pppoe-primary\"] distance=10"
```
{% endcode %}

### PPPoE load balancing

{% code overflow="wrap" %}
```bash
# Create two PPPoE connections
/interface pppoe-client add name=pppoe-wan1 interface=ether1 user="user1" password="pass1" add-default-route=no
/interface pppoe-client add name=pppoe-wan2 interface=ether2 user="user2" password="pass2" add-default-route=no

# Configure load balancing routes
/ip route add dst-address=0.0.0.0/0 gateway=pppoe-wan1,pppoe-wan2 distance=1 check-gateway=ping

# Configure connection marking for session persistence
/ip firewall mangle add chain=prerouting connection-state=new nth=2,1 action=mark-connection new-connection-mark=wan1_conn
/ip firewall mangle add chain=prerouting connection-mark=wan1_conn action=mark-routing new-routing-mark=to_wan1
/ip firewall mangle add chain=prerouting connection-state=new action=mark-connection new-connection-mark=wan2_conn
/ip firewall mangle add chain=prerouting connection-mark=wan2_conn action=mark-routing new-routing-mark=to_wan2

# Add routes for marked connections
/ip route add dst-address=0.0.0.0/0 gateway=pppoe-wan1 routing-mark=to_wan1
/ip route add dst-address=0.0.0.0/0 gateway=pppoe-wan2 routing-mark=to_wan2
```
{% endcode %}

***

## PPPoE security considerations

### Secure PPPoE server

{% code overflow="wrap" %}
```bash
# Limit concurrent sessions per user
/ppp profile set pppoe-profile session-timeout=8h idle-timeout=10m only-one=yes

# Enable stronger authentication
/interface pppoe-server server set authentication=mschap2

# Rate limiting per user
/ppp profile set pppoe-profile rate-limit=20M/10M

# Monitor failed authentication attempts
/log print where message~"authentication failed" and topics~"ppp"
```
{% endcode %}

### Client security

{% code overflow="wrap" %}
```bash
# Use secure authentication methods
/interface pppoe-client set pppoe-out1 allow-fast-path=no

# Monitor connection for security
/tool netwatch add host=GATEWAY_IP interval=60s comment="Monitor PPPoE gateway"

# Log PPPoE events
/system logging add topics=ppp,info action=memory
```
{% endcode %}

***

## Advanced PPPoE features

### PPPoE with VPN integration

{% code overflow="wrap" %}
```bash
# Create PPPoE connection for VPN server
/interface pppoe-client add name=pppoe-vpn interface=ether3 user="vpn_user" password="vpn_pass" add-default-route=no

# Assign specific routing for VPN traffic
/ip route add dst-address=VPN_NETWORK/24 gateway=pppoe-vpn

# Configure firewall for VPN over PPPoE
/ip firewall nat add chain=srcnat out-interface=pppoe-vpn action=masquerade
```
{% endcode %}

### PPPoE bandwidth monitoring

{% code overflow="wrap" %}
```bash
# Monitor PPPoE bandwidth usage
/tool graphing interface add interface=pppoe-out1 allow-address=0.0.0.0/0

# Set up SNMP monitoring
/snmp set enabled=yes contact="admin@company.com" location="Main Office"

# Create bandwidth alerts
/system script add name=bandwidth-alert source={
    :local usage [/interface get pppoe-out1 tx-byte]
    :if ($usage > 1000000000) do={
        /tool e-mail send to="admin@company.com" subject="High bandwidth usage" body="PPPoE usage exceeded 1GB"
    }
}
/system scheduler add name=check-bandwidth interval=1h on-event=bandwidth-alert
```
{% endcode %}

***

<details>

<summary>Show complete PPPoE client setup with failover</summary>

{% code overflow="wrap" %}
```bash
# 1. Create VLAN for ISP (if required)
/interface vlan add name=vlan-isp vlan-id=35 interface=ether1 comment="ISP VLAN"

# 2. Create primary PPPoE connection
/interface pppoe-client add \
    name=pppoe-primary \
    interface=vlan-isp \
    user="primary_user" \
    password="primary_password" \
    add-default-route=yes \
    use-peer-dns=yes \
    default-route-distance=1 \
    max-mtu=1492 \
    max-mru=1492 \
    disabled=no \
    comment="Primary ISP connection"

# 3. Create backup PPPoE connection (different ISP)
/interface pppoe-client add \
    name=pppoe-backup \
    interface=ether2 \
    user="backup_user" \
    password="backup_password" \
    add-default-route=yes \
    use-peer-dns=no \
    default-route-distance=2 \
    max-mtu=1492 \
    max-mru=1492 \
    disabled=no \
    comment="Backup ISP connection"

# 4. Configure MSS clamping
/ip firewall mangle add chain=forward action=change-mss new-mss=1360 protocol=tcp tcp-flags=syn out-interface=pppoe-primary
/ip firewall mangle add chain=forward action=change-mss new-mss=1360 protocol=tcp tcp-flags=syn out-interface=pppoe-backup

# 5. Set up monitoring and failover
/tool netwatch add \
    host=8.8.8.8 \
    interval=30s \
    up-script="/ip route set [find gateway~\"pppoe-primary\"] distance=1; /log info \"Primary PPPoE restored\"" \
    down-script="/ip route set [find gateway~\"pppoe-primary\"] distance=10; /log error \"Primary PPPoE failed, using backup\""

# 6. Configure NAT
/ip firewall nat add chain=srcnat out-interface=pppoe-primary action=masquerade comment="NAT for primary"
/ip firewall nat add chain=srcnat out-interface=pppoe-backup action=masquerade comment="NAT for backup"

# 7. Set backup DNS
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes
```
{% endcode %}

</details>

## PPPoE performance optimization

### Optimize for performance

{% code overflow="wrap" %}
```bash
# Enable fast path (if supported)
/interface pppoe-client set pppoe-out1 allow-fast-path=yes

# Optimize MTU/MRU
/interface pppoe-client set pppoe-out1 max-mtu=1492 max-mru=1492 mrru=1600

# Enable multilink if supported by ISP
/interface pppoe-client set pppoe-out1 mrru=1600

# Optimize keepalive
/interface pppoe-client set pppoe-out1 keepalive-timeout=60
```
{% endcode %}

### Monitor performance

{% code overflow="wrap" %}
```bash
# Monitor connection quality
/interface monitor-traffic interface=pppoe-out1 duration=60

# Check error rates
/interface print stats where name=pppoe-out1

# Test latency and packet loss
/ping 8.8.8.8 interface=pppoe-out1 count=100

# Monitor CPU usage
/system resource print
```
{% endcode %}

## Best practices

### PPPoE client best practices

1. **Use appropriate MTU** - Set to 1492 or lower to avoid fragmentation
2. **Enable MSS clamping** - Prevent TCP MSS issues
3. **Monitor connection** - Use netwatch for failover
4. **Backup DNS** - Don't rely solely on peer DNS
5. **Log events** - Monitor PPPoE connection events

### PPPoE server best practices

1. **Limit sessions** - Prevent resource exhaustion
2. **Use rate limiting** - Control bandwidth per user
3. **Strong authentication** - Use MSCHAPv2 minimum
4. **Monitor usage** - Track bandwidth and session count
5. **Regular maintenance** - Clean up inactive sessions

### Security recommendations

1. **Change default credentials** - Use strong passwords
2. **Enable logging** - Monitor authentication attempts  
3. **Use VLANs** - Isolate PPPoE traffic
4. **Implement monitoring** - Watch for unusual activity
5. **Regular updates** - Keep firmware current

