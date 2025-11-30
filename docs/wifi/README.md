---
description: This section covers how to configure WiFi interfaces for wireless networking.
icon: wifi
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

# WiFi

{% hint style="info" %}
RouterOS WiFi configuration has evolved significantly. RouterOS v7+ introduces a new WiFi package (wifi-qcom) alongside the legacy wireless package for different hardware platforms.
{% endhint %}

In WinBox you can configure WiFi in **WiFi** (RouterOS v7+) or **Wireless** (legacy), or you can use terminal with commands `/interface wifi` or `/interface wireless`

This documentation covers both the new WiFi package and legacy wireless configuration.

***

## WiFi package overview

### New WiFi package (RouterOS v7+)

**Supported hardware:**
- Qualcomm-based devices (hAP ax series, etc.)
- Uses `/interface wifi` commands
- Modern WiFi 6/6E support
- Better performance and features

**Features:**
- WiFi 6 (802.11ax) support
- WPA3 security
- OFDMA and MU-MIMO
- Better channel management
- Advanced security features

### Legacy wireless package

**Supported hardware:**
- Atheros-based devices (most older RouterBoards)
- Uses `/interface wireless` commands
- WiFi 4/5 support (802.11n/ac)
- Widely deployed and stable

***

## New WiFi package configuration (v7+)

### Enable WiFi interface

{% code overflow="wrap" %}
```bash
# Check available WiFi interfaces
/interface wifi print

# Enable WiFi radio
/interface wifi set wifi1 disabled=no

# Check WiFi capabilities
/interface wifi cap print
```
{% endcode %}

### Create basic access point

{% code overflow="wrap" %}
```bash
# Create WiFi security profile
/interface wifi security add name=home-security authentication-types=wpa2-psk,wpa3-psk encryption=ccmp passphrase="YourSecurePassword123!"

# Create WiFi configuration
/interface wifi add name=wifi-home master-interface=wifi1 ssid="YourNetworkName" security=home-security disabled=no

# Set channel and power
/interface wifi set wifi1 channel=auto power=20
```
{% endcode %}

### Advanced WiFi configuration

{% code overflow="wrap" %}
```bash
# Configure detailed WiFi settings
/interface wifi set wifi1 \
    channel=36 \
    channel-width=20/40/80mhz \
    country=united_states \
    power=20 \
    scan-list=5180-5825 \
    guard-interval=auto

# Configure multiple SSIDs
/interface wifi add name=wifi-guest master-interface=wifi1 ssid="GuestNetwork" security=guest-security disabled=no
/interface wifi add name=wifi-iot master-interface=wifi1 ssid="IoTDevices" security=iot-security disabled=no
```
{% endcode %}

### WiFi 6 specific features

{% code overflow="wrap" %}
```bash
# Enable WiFi 6 features
/interface wifi set wifi1 \
    .ax=yes \
    mu-mimo=yes \
    tx-chains=0,1,2,3 \
    rx-chains=0,1,2,3

# Configure OFDMA
/interface wifi set wifi1 ofdma=yes
```
{% endcode %}

***

## Legacy wireless configuration

### Basic wireless AP setup

{% code overflow="wrap" %}
```bash
# Enable wireless interface
/interface wireless set wlan1 disabled=no

# Configure basic wireless settings
/interface wireless set wlan1 \
    mode=ap-bridge \
    ssid="YourNetwork" \
    band=5ghz-a/n/ac \
    channel-width=20/40/80mhz-ce \
    frequency=auto \
    wireless-protocol=802.11 \
    security-profile=default

# Set transmission power
/interface wireless set wlan1 tx-power=20
```
{% endcode %}

### Create security profile

{% code overflow="wrap" %}
```bash
# Create WPA2 security profile
/interface wireless security-profiles add name=wpa2-profile \
    mode=dynamic-keys \
    authentication-types=wpa2-psk \
    unicast-ciphers=aes-ccm \
    group-ciphers=aes-ccm \
    wpa2-pre-shared-key="YourSecurePassword123!"

# Apply security profile
/interface wireless set wlan1 security-profile=wpa2-profile
```
{% endcode %}

### Advanced wireless settings

{% code overflow="wrap" %}
```bash
# Configure advanced wireless parameters
/interface wireless set wlan1 \
    distance=indoors \
    tx-power-mode=card-rates \
    rate-set=configured \
    supported-rates-a/g=6Mbps,9Mbps,12Mbps,18Mbps,24Mbps,36Mbps,48Mbps,54Mbps \
    basic-rates-a/g=6Mbps,12Mbps,24Mbps \
    max-station-count=50 \
    multicast-helper=full \
    wmm-support=enabled \
    guard-interval=short
```
{% endcode %}

***

## WiFi security configuration

### WPA3 security (new WiFi package)

{% code overflow="wrap" %}
```bash
# Create WPA3 security profile
/interface wifi security add name=wpa3-profile \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="SuperSecurePassword123!" \
    disable-pmkid=yes

# Enhanced security settings
/interface wifi security set wpa3-profile \
    ft=yes \
    ft-over-ds=yes \
    group-key-update=1h
```
{% endcode %}

### Enterprise security (802.1X)

{% code overflow="wrap" %}
```bash
# Configure RADIUS authentication
/radius add service=wireless address=192.168.1.10 secret="radius_secret" timeout=3s

# Create enterprise security profile
/interface wifi security add name=enterprise-security \
    authentication-types=wpa2-eap,wpa3-eap \
    encryption=ccmp \
    eap-methods=eap-tls,peap,eap-ttls \
    tls-mode=verify-certificate
```
{% endcode %}

### Guest network isolation

{% code overflow="wrap" %}
```bash
# Create isolated guest network
/interface wifi security add name=guest-security \
    authentication-types=wpa2-psk \
    encryption=ccmp \
    passphrase="GuestPassword123" \
    group-key-update=30m

/interface wifi add name=wifi-guest \
    master-interface=wifi1 \
    ssid="Guest-Network" \
    security=guest-security \
    multicast-helper=disabled \
    disabled=no
```
{% endcode %}

***

## Multiple SSID configuration

### Create multiple networks

{% code overflow="wrap" %}
```bash
# Main corporate network
/interface wifi security add name=corp-security authentication-types=wpa3-psk encryption=ccmp passphrase="CorpPassword123!"
/interface wifi add name=wifi-corp master-interface=wifi1 ssid="Corporate" security=corp-security

# Guest network with captive portal
/interface wifi security add name=guest-security authentication-types=wpa2-psk encryption=ccmp passphrase="Guest2024"
/interface wifi add name=wifi-guest master-interface=wifi1 ssid="Guest-WiFi" security=guest-security

# IoT network with isolation
/interface wifi security add name=iot-security authentication-types=wpa2-psk encryption=ccmp passphrase="IoTDevices123"
/interface wifi add name=wifi-iot master-interface=wifi1 ssid="IoT-Devices" security=iot-security
```
{% endcode %}

### VLAN assignment per SSID

{% code overflow="wrap" %}
```bash
# Assign VLANs to different SSIDs
/interface wifi set wifi-corp vlan-id=10
/interface wifi set wifi-guest vlan-id=20  
/interface wifi set wifi-iot vlan-id=30

# Create VLAN interfaces
/interface vlan add name=vlan10-corp vlan-id=10 interface=bridge1
/interface vlan add name=vlan20-guest vlan-id=20 interface=bridge1
/interface vlan add name=vlan30-iot vlan-id=30 interface=bridge1

# Bridge WiFi interfaces to VLANs
/interface bridge port add interface=wifi-corp bridge=bridge1 pvid=10
/interface bridge port add interface=wifi-guest bridge=bridge1 pvid=20
/interface bridge port add interface=wifi-iot bridge=bridge1 pvid=30
```
{% endcode %}

***

## WiFi monitoring and optimization

### Monitor WiFi performance

{% code overflow="wrap" %}
```bash
# Check WiFi interface status
/interface wifi print detail

# Monitor connected clients
/interface wifi registration-table print

# Check signal quality
/interface wifi monitor wifi1

# View WiFi statistics
/interface wifi print stats
```
{% endcode %}

### Channel optimization

{% code overflow="wrap" %}
```bash
# Scan for available channels
/interface wifi scan wifi1

# Set optimal channel
/interface wifi set wifi1 channel=36 channel-width=80mhz

# Enable automatic channel selection
/interface wifi set wifi1 channel=auto
```
{% endcode %}

### Client management

{% code overflow="wrap" %}
```bash
# Set client limits
/interface wifi set wifi-guest max-sta-count=20

# Enable band steering
/interface wifi set wifi1 band-steering=yes

# Configure load balancing
/interface wifi set wifi1 load-balancing=yes
```
{% endcode %}

***

## WiFi troubleshooting

### Common issues diagnostics

{% code overflow="wrap" %}
```bash
# Check WiFi interface status
/interface wifi print detail where name=wifi1

# Monitor real-time statistics
/interface wifi monitor wifi1

# Check for interference
/interface wifi scan wifi1 duration=10

# View error counters
/interface print stats where name~"wifi"
```
{% endcode %}

### Client connectivity issues

{% code overflow="wrap" %}
```bash
# Check client registration
/interface wifi registration-table print detail

# Monitor authentication failures
/log print where message~"wifi" and message~"auth"

# Check security configuration
/interface wifi security print detail
```
{% endcode %}

### Performance troubleshooting

{% code overflow="wrap" %}
```bash
# Check channel utilization
/interface wifi scan wifi1

# Monitor traffic
/interface monitor-traffic interface=wifi1

# Check power levels
/interface wifi set wifi1 tx-power=15

# Test different channels
/interface wifi set wifi1 channel=44
```
{% endcode %}

***

## Advanced WiFi features

### WiFi mesh networking

{% code overflow="wrap" %}
```bash
# Configure mesh node
/interface wifi mesh add name=mesh1 \
    mesh-id="corporate-mesh" \
    master-interface=wifi1 \
    security=mesh-security \
    disabled=no

# Create mesh security
/interface wifi security add name=mesh-security \
    authentication-types=wpa2-psk \
    encryption=ccmp \
    passphrase="MeshPassword123"
```
{% endcode %}

### Captive portal integration

{% code overflow="wrap" %}
```bash
# Enable hotspot on guest network
/ip hotspot add name=guest-hotspot interface=wifi-guest address-pool=guest-pool disabled=no

# Create guest IP pool
/ip pool add name=guest-pool ranges=192.168.100.10-192.168.100.100

# Configure hotspot profile
/ip hotspot profile set default login-by=http-pap,cookie,trial trial-uptime-limit=1h
```
{% endcode %}

### Band steering and load balancing

{% code overflow="wrap" %}
```bash
# Enable advanced client management
/interface wifi set wifi1 \
    band-steering=yes \
    load-balancing=yes \
    multicast-enhance=yes

# Configure client timeouts
/interface wifi set wifi1 \
    keepalive-frames=enabled \
    client-isolation=no
```
{% endcode %}

***

<details>

<summary>Show complete WiFi setup with multiple SSIDs</summary>

{% code overflow="wrap" %}
```bash
# 1. Configure WiFi radio (new package)
/interface wifi set wifi1 disabled=no channel=auto power=20 country=united_states

# 2. Create security profiles
/interface wifi security add name=corp-security authentication-types=wpa3-psk encryption=ccmp passphrase="CorpSecure2024!"
/interface wifi security add name=guest-security authentication-types=wpa2-psk encryption=ccmp passphrase="GuestWiFi2024"
/interface wifi security add name=iot-security authentication-types=wpa2-psk encryption=ccmp passphrase="IoTDevices2024"

# 3. Create multiple SSIDs
/interface wifi add name=wifi-corp master-interface=wifi1 ssid="Corporate" security=corp-security disabled=no
/interface wifi add name=wifi-guest master-interface=wifi1 ssid="Guest-Network" security=guest-security disabled=no
/interface wifi add name=wifi-iot master-interface=wifi1 ssid="IoT-Network" security=iot-security disabled=no

# 4. Configure VLAN assignment
/interface wifi set wifi-corp vlan-id=10
/interface wifi set wifi-guest vlan-id=20
/interface wifi set wifi-iot vlan-id=30

# 5. Add to bridge with VLAN support
/interface bridge port add interface=wifi-corp bridge=bridge1 pvid=10
/interface bridge port add interface=wifi-guest bridge=bridge1 pvid=20
/interface bridge port add interface=wifi-iot bridge=bridge1 pvid=30

# 6. Create VLAN interfaces
/interface vlan add name=vlan10-corp vlan-id=10 interface=bridge1
/interface vlan add name=vlan20-guest vlan-id=20 interface=bridge1
/interface vlan add name=vlan30-iot vlan-id=30 interface=bridge1

# 7. Assign IP addresses
/ip address add address=192.168.10.1/24 interface=vlan10-corp comment="Corporate VLAN"
/ip address add address=192.168.20.1/24 interface=vlan20-guest comment="Guest VLAN"
/ip address add address=192.168.30.1/24 interface=vlan30-iot comment="IoT VLAN"
```
{% endcode %}

</details>

## WiFi security best practices

### Security recommendations

1. **Use WPA3** - Enable WPA3-PSK for enhanced security
2. **Strong passwords** - Use complex passphrases (12+ characters)
3. **Regular updates** - Keep firmware updated for security patches
4. **Network segmentation** - Separate guest and corporate traffic
5. **Monitor access** - Regularly check connected clients

### Advanced security features

{% code overflow="wrap" %}
```bash
# Enable additional security features
/interface wifi security set corp-security \
    disable-pmkid=yes \
    ft=yes \
    group-key-update=1h \
    eapol-timeout=30s

# Configure MAC address filtering (if needed)
/interface wifi access-list add interface=wifi-corp mac-address=AA:BB:CC:DD:EE:FF action=accept
/interface wifi set wifi-corp default-authentication=no
```
{% endcode %}

## Performance optimization

### Optimize for high-density environments

{% code overflow="wrap" %}
```bash
# Configure for high client density
/interface wifi set wifi1 \
    multicast-enhance=yes \
    ampdu-priorities=0,1,2,3,4,5,6,7 \
    amsdu-limit=8192 \
    amsdu-threshold=8192

# Enable fair queuing
/queue interface set wifi1 queue=wireless-default
```
{% endcode %}

### Channel and power optimization

{% code overflow="wrap" %}
```bash
# Optimize channel selection
/interface wifi set wifi1 \
    channel=auto \
    scan-list=5180-5825 \
    channel-width=20/40/80mhz

# Dynamic power adjustment
/interface wifi set wifi1 \
    tx-power=auto \
    tx-power-mode=card-rates
```
{% endcode %}

