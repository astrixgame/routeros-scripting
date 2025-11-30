---
description: This section covers how to configure LTE interfaces for cellular internet connectivity.
icon: tower-cell
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

# LTE

{% hint style="info" %}
LTE interfaces provide cellular internet connectivity for primary connections, backup links, or remote site connectivity where traditional broadband is unavailable.
{% endhint %}

In WinBox you can configure LTE in **Interfaces -> LTE**, or you can use terminal with command `/interface lte`

LTE interfaces require compatible hardware with cellular modems and proper SIM card configuration.

***

## LTE hardware requirements

### Supported LTE devices

**RouterBoard models with built-in LTE:**
- **LtAP series** - LtAP mini LTE kit, LtAP LTE6 kit
- **wAP series** - wAP R LTE kit, wAP LTE kit  
- **KNOT** - KNOT LTE kit
- **SXT series** - SXT LTE kit, SXT R LTE6 kit

**External LTE modems:**
- **USB LTE modems** - Various manufacturers supported
- **miniPCIe LTE cards** - For compatible RouterBoard models
- **LTE HAT boards** - For specific RouterBoard series

### SIM card requirements

**SIM card considerations:**
- **Active cellular plan** - Data plan with sufficient bandwidth
- **APN settings** - Access Point Name from carrier
- **Authentication** - Username/password if required
- **PIN code** - SIM card PIN if enabled

***

## Basic LTE configuration

### Enable LTE interface

{% code overflow="wrap" %}
```bash
# Check available LTE interfaces
/interface lte print

# Enable LTE interface
/interface lte set lte1 disabled=no

# Check LTE status
/interface lte monitor lte1
```
{% endcode %}

### Configure APN settings

In WinBox go to **Interfaces -> LTE** and configure:

* **APN** - Access Point Name from carrier
* **Username** - If required by carrier
* **Password** - If required by carrier
* **PIN** - SIM card PIN code
* **Network Mode** - Auto, 3G, LTE, etc.

{% code overflow="wrap" %}
```bash
# Configure basic LTE settings
/interface lte apn set [find default=yes] \
    apn="internet" \
    user="" \
    password="" \
    authentication=none

# Set PIN code if SIM requires it
/interface lte set lte1 pin="1234"

# Enable LTE interface
/interface lte set lte1 disabled=no
```
{% endcode %}

### Carrier-specific APN settings

{% code overflow="wrap" %}
```bash
# Verizon (US)
/interface lte apn set [find default=yes] apn="vzwinternet" authentication=none

# AT&T (US)  
/interface lte apn set [find default=yes] apn="broadband" authentication=none

# T-Mobile (US)
/interface lte apn set [find default=yes] apn="fast.t-mobile.com" authentication=none

# Vodafone (EU)
/interface lte apn set [find default=yes] apn="internet.vodafone.net" user="vodafone" password="vodafone" authentication=pap

# O2 (EU)
/interface lte apn set [find default=yes] apn="internet" user="o2" password="o2" authentication=pap
```
{% endcode %}

***

## Advanced LTE configuration

### Network mode selection

{% code overflow="wrap" %}
```bash
# Set specific network mode
/interface lte set lte1 network-mode=lte

# Allow multiple modes (fallback)
/interface lte set lte1 network-mode=auto

# Force 3G only (for compatibility)
/interface lte set lte1 network-mode=3g

# Check available modes
/interface lte info lte1
```
{% endcode %}

### Band selection

{% code overflow="wrap" %}
```bash
# Lock to specific LTE bands (carrier dependent)
/interface lte set lte1 band=""

# Common LTE bands by region:
# US: 2,4,5,12,13,14,17,25,26,41,66,71
# EU: 1,3,7,8,20,28,32,38,40,42,43
# Asia: 1,3,8,11,18,19,21,28,41,42

# Example: Lock to bands 1,3,7 (common EU bands)
/interface lte set lte1 band="1,3,7"
```
{% endcode %}

### Multiple APN profiles

{% code overflow="wrap" %}
```bash
# Create multiple APN profiles
/interface lte apn add name=internet apn="internet" authentication=none use-network-apn=no
/interface lte apn add name=corporate apn="corp.apn" user="corpuser" password="corppass" authentication=pap use-network-apn=no

# Set default APN
/interface lte set lte1 apn-profiles=internet

# Switch between APNs
/interface lte set lte1 apn-profiles=corporate
```
{% endcode %}

***

## LTE as primary connection

### Configure LTE as main internet

{% code overflow="wrap" %}
```bash
# Enable LTE with routing
/interface lte set lte1 add-default-route=yes use-peer-dns=yes disabled=no

# Assign IP address to LTE interface (if static needed)
/ip address add address=192.168.88.1/24 interface=lte1

# Configure NAT for LAN clients
/ip firewall nat add chain=srcnat out-interface=lte1 action=masquerade comment="LTE NAT"

# Set up DNS
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes
```
{% endcode %}

### LTE bandwidth management

{% code overflow="wrap" %}
```bash
# Create queue for LTE interface
/queue simple add name=lte-total target=lte1 max-limit=50M/10M comment="LTE bandwidth limit"

# Per-client bandwidth control
/queue simple add name=guest-lte target=192.168.1.0/24 parent=lte-total max-limit=20M/5M

# Priority queuing for important traffic
/queue simple add name=voip-priority target=192.168.1.0/24 parent=lte-total max-limit=2M/2M priority=1/1
```
{% endcode %}

***

## LTE as backup connection

### Automatic failover to LTE

{% code overflow="wrap" %}
```bash
# Configure LTE as backup (higher distance)
/interface lte set lte1 add-default-route=yes default-route-distance=2 use-peer-dns=no disabled=no

# Primary connection (lower distance)
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1 distance=1 comment="Primary internet"

# Monitor primary connection and enable LTE backup
/tool netwatch add \
    host=8.8.8.8 \
    interval=30s \
    timeout=5s \
    up-script="/ip route set [find comment=\"Primary internet\"] disabled=no; /interface lte set lte1 disabled=yes; /log info \"Primary restored, LTE disabled\"" \
    down-script="/interface lte set lte1 disabled=no; /ip route set [find comment=\"Primary internet\"] disabled=yes; /log warning \"Primary failed, LTE enabled\""
```
{% endcode %}

### Smart failover with data usage control

{% code overflow="wrap" %}
```bash
# Advanced failover script with data usage awareness
/system script add name=lte-failover source={
    :local primaryGW "192.168.1.1"
    :local testHost "8.8.8.8"
    :local pingCount 3
    :local failureThreshold 2
    :local lteDataLimit 5000000000
    
    # Check primary connectivity
    :local pingResult [/ping $testHost count=$pingCount routing-table=main]
    
    # Check LTE data usage
    :local lteUsage [/interface get lte1 rx-byte]
    
    :if ($pingResult < $failureThreshold) do={
        :if ($lteUsage < $lteDataLimit) do={
            /interface lte set lte1 disabled=no
            /ip route set [find gateway=$primaryGW] disabled=yes
            /log warning "Primary failed, switching to LTE (usage: $lteUsage bytes)"
        } else {
            /log error "Primary failed but LTE data limit reached ($lteDataLimit bytes)"
        }
    } else {
        /interface lte set lte1 disabled=yes  
        /ip route set [find gateway=$primaryGW] disabled=no
        /log info "Primary connection restored, LTE disabled"
    }
}

# Schedule failover check
/system scheduler add name=check-failover interval=1m on-event=lte-failover
```
{% endcode %}

***

## LTE monitoring and diagnostics

### Monitor LTE connection

{% code overflow="wrap" %}
```bash
# Check LTE status and signal
/interface lte monitor lte1

# View detailed LTE information
/interface lte info lte1

# Check data usage
/interface print stats where name=lte1

# Monitor signal strength over time
/interface lte monitor lte1 duration=60
```
{% endcode %}

### LTE signal optimization

{% code overflow="wrap" %}
```bash
# Check current cell information
/interface lte info lte1

# Scan for available cells
/interface lte scan lte1 duration=60

# Lock to specific cell (if better signal)
/interface lte cell-lock lte1 earfcn=1234 physical-cell-id=56

# Remove cell lock
/interface lte cell-lock lte1 earfcn="" physical-cell-id=""
```
{% endcode %}

### Data usage tracking

{% code overflow="wrap" %}
```bash
# Create data usage monitoring script
/system script add name=lte-usage-check source={
    :local lteRx [/interface get lte1 rx-byte]
    :local lteTx [/interface get lte1 tx-byte]
    :local totalUsage ($lteRx + $lteTx)
    :local usageLimit 10000000000
    
    :if ($totalUsage > $usageLimit) do={
        /interface lte set lte1 disabled=yes
        /log error "LTE disabled - data limit exceeded ($totalUsage bytes)"
        /tool e-mail send to="admin@company.com" subject="LTE Data Limit" body="LTE connection disabled due to data usage limit"
    }
    
    /log info "LTE usage: RX=$lteRx TX=$lteTx Total=$totalUsage"
}

# Schedule usage check
/system scheduler add name=check-lte-usage interval=1h on-event=lte-usage-check
```
{% endcode %}

***

## LTE troubleshooting

### Common LTE issues

{% code overflow="wrap" %}
```bash
# Check SIM card status
/interface lte info lte1

# Verify APN settings
/interface lte apn print

# Check signal strength
/interface lte monitor lte1

# Test connectivity
/ping 8.8.8.8 interface=lte1

# Reset LTE modem
/interface lte set lte1 disabled=yes
:delay 5s
/interface lte set lte1 disabled=no
```
{% endcode %}

### Signal quality diagnostics

{% code overflow="wrap" %}
```bash
# Detailed signal analysis
/interface lte info lte1

# Key signal metrics to check:
# RSRP (Reference Signal Received Power): > -100 dBm good
# RSRQ (Reference Signal Received Quality): > -10 dB good  
# SINR (Signal to Interference plus Noise Ratio): > 10 dB good
# CQI (Channel Quality Indicator): > 7 good

# Log signal quality over time
/system script add name=signal-logger source={
    :local rsrp [/interface lte info lte1 once as-value]->"rsrp"
    :local rsrq [/interface lte info lte1 once as-value]->"rsrq"
    :local sinr [/interface lte info lte1 once as-value]->"sinr"
    /log info "LTE Signal: RSRP=$rsrp RSRQ=$rsrq SINR=$sinr"
}

/system scheduler add name=log-signal interval=5m on-event=signal-logger
```
{% endcode %}

***

## LTE security considerations

### Secure LTE configuration

{% code overflow="wrap" %}
```bash
# Disable unnecessary services on LTE
/ip service disable telnet,ftp,www

# Use strong firewall rules for LTE interface
/ip firewall filter add chain=input in-interface=lte1 action=drop comment="Drop all input on LTE"
/ip firewall filter add chain=input in-interface=lte1 connection-state=established,related action=accept place-before=0

# Enable connection tracking
/ip firewall connection tracking set enabled=yes

# Monitor for unusual traffic
/ip firewall filter add chain=forward in-interface=lte1 action=log log-prefix="LTE-IN"
```
{% endcode %}

### VPN over LTE

{% code overflow="wrap" %}
```bash
# Configure VPN client over LTE for security
/interface ovpn-client add name=lte-vpn connect-to=vpn.company.com port=1194 mode=ip protocol=udp user=username password=password add-default-route=no interface=lte1

# Route specific traffic through VPN
/ip route add dst-address=192.168.100.0/24 gateway=lte-vpn

# Force all LTE traffic through VPN
/ip route add dst-address=0.0.0.0/0 gateway=lte-vpn distance=1
```
{% endcode %}

***

## LTE optimization techniques

### Antenna optimization

**External antenna considerations:**
- **Directional antennas** - Point towards cell tower for better signal
- **MIMO antennas** - Use both antenna connections for LTE diversity
- **Antenna placement** - Higher placement typically improves signal
- **Cable quality** - Use low-loss coaxial cables

### Carrier aggregation

{% code overflow="wrap" %}
```bash
# Enable carrier aggregation (if supported)
/interface lte set lte1 carrier-aggregation=yes

# Check aggregation status
/interface lte info lte1

# Monitor aggregated carriers
/interface lte monitor lte1
```
{% endcode %}

***

<details>

<summary>Show complete LTE backup setup</summary>

{% code overflow="wrap" %}
```bash
# 1. Configure LTE interface with APN
/interface lte apn set [find default=yes] apn="internet" authentication=none use-network-apn=no
/interface lte set lte1 pin="" add-default-route=yes default-route-distance=2 use-peer-dns=no disabled=yes

# 2. Create primary internet route
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1 distance=1 comment="Primary Internet"

# 3. Configure NAT for both connections
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="Primary NAT"
/ip firewall nat add chain=srcnat out-interface=lte1 action=masquerade comment="LTE NAT"

# 4. Set up bandwidth limiting for LTE
/queue simple add name=lte-limit target=lte1 max-limit=50M/10M disabled=yes

# 5. Create failover monitoring script
/system script add name=internet-failover source={
    :local primaryHost "8.8.8.8"
    :local backupHost "1.1.1.1"
    :local pingCount 3
    :local threshold 1
    
    # Test primary connection
    :local primaryResult [/ping $primaryHost count=$pingCount routing-table=main]
    
    :if ($primaryResult < $threshold) do={
        # Primary failed, test backup path
        :local backupResult [/ping $backupHost count=$pingCount routing-table=main]
        :if ($backupResult < $threshold) do={
            # Enable LTE backup
            /interface lte set lte1 disabled=no
            /queue simple set lte-limit disabled=no
            /ip route set [find comment="Primary Internet"] disabled=yes
            /log warning "Primary internet failed, LTE backup enabled"
        }
    } else {
        # Primary working, disable LTE
        /interface lte set lte1 disabled=yes
        /queue simple set lte-limit disabled=yes  
        /ip route set [find comment="Primary Internet"] disabled=no
        /log info "Primary internet restored, LTE backup disabled"
    }
}

# 6. Schedule failover check
/system scheduler add name=failover-check interval=2m on-event=internet-failover

# 7. Create data usage monitoring
/system script add name=lte-data-monitor source={
    :local dataLimit 5000000000
    :local currentUsage ([/interface get lte1 rx-byte] + [/interface get lte1 tx-byte])
    
    :if ($currentUsage > $dataLimit) do={
        /interface lte set lte1 disabled=yes
        /log error "LTE disabled - monthly data limit reached"
        /tool e-mail send to="admin@example.com" subject="LTE Data Alert" body="LTE backup has reached monthly data limit and has been disabled"
    }
}

# 8. Schedule data monitoring
/system scheduler add name=data-check interval=1h on-event=lte-data-monitor
```
{% endcode %}

</details>

## Performance optimization

### Optimize LTE performance

{% code overflow="wrap" %}
```bash
# Set optimal network mode for performance
/interface lte set lte1 network-mode=lte

# Enable carrier aggregation
/interface lte set lte1 carrier-aggregation=yes

# Optimize for low latency vs high throughput
/interface lte set lte1 modem-init="AT+QCFG=\"nwscanmode\",3"

# Set appropriate MTU
/interface lte set lte1 mtu=1500
```
{% endcode %}

### Monitor performance metrics

{% code overflow="wrap" %}
```bash
# Monitor latency
/ping 8.8.8.8 interface=lte1 count=100

# Check throughput
/tool bandwidth-test address=speedtest.net interface=lte1

# Monitor signal quality trends
/interface lte monitor lte1 duration=300
```
{% endcode %}

## Best practices

### LTE deployment best practices

1. **Antenna placement** - Use external antennas when possible
2. **Signal monitoring** - Regularly check signal quality metrics  
3. **Data management** - Implement usage monitoring and limits
4. **Redundancy** - Use LTE as backup, not primary when possible
5. **Security** - Always use VPN over cellular connections

### Cost optimization

1. **Usage monitoring** - Track data consumption carefully
2. **Scheduled connectivity** - Enable LTE only when needed
3. **Compression** - Use traffic compression where possible
4. **Priority traffic** - Ensure critical traffic gets priority
5. **Carrier selection** - Choose appropriate data plans

