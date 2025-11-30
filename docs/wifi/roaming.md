---
description: This section covers WiFi roaming methods and optimization techniques for seamless client mobility between access points.
icon: arrows-rotate
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

# WiFi Roaming

{% hint style="info" %}
WiFi roaming allows clients to move seamlessly between access points without losing connectivity. RouterOS supports multiple roaming standards and optimization techniques for smooth handovers.
{% endhint %}

In WinBox you can configure roaming in **WiFi** (RouterOS v7+) or **Wireless** (legacy), or use terminal commands `/interface wifi` or `/interface wireless`

This documentation covers roaming implementation for both new WiFi package and legacy wireless systems.

***

## Understanding WiFi roaming

### Roaming fundamentals

**Why roaming matters:**
- Maintains connectivity during client movement
- Reduces connection interruptions in enterprise environments
- Improves user experience in large coverage areas
- Essential for mobile devices and real-time applications

**Roaming process:**
1. **Signal degradation** - Current AP signal weakens
2. **Scanning** - Client searches for better access points
3. **Authentication** - Client authenticates with new AP
4. **Association** - Client associates with new AP
5. **Data transfer** - Normal communication resumes

**Roaming standards:**
- **802.11r (Fast BSS Transition)** - Fast roaming with pre-authentication
- **802.11k (Radio Resource Management)** - Neighbor discovery and reporting
- **802.11v (BSS Transition Management)** - Assisted roaming decisions
- **802.11w (Protected Management Frames)** - Secure management frame protection

***

## Fast roaming with 802.11r

### Fast BSS Transition (FT) overview

**Key benefits:**
- Reduces roaming time from 500ms+ to under 50ms
- Pre-authenticates with target access points
- Maintains security context during handover
- Supports both over-the-air (FT-Air) and over-DS (FT-DS)

**FT methods:**
- **FT over Air** - Direct client-to-AP communication
- **FT over DS** - Communication through distribution system
- **Mixed mode** - Supports both methods simultaneously

### 802.11r configuration (new WiFi package)

{% code overflow="wrap" %}
```bash
# Enable 802.11r on new WiFi package
/interface wifi security add name=fast-roaming-security \
    authentication-types=wpa2-psk,wpa3-psk \
    encryption=ccmp \
    passphrase="FastRoamNetwork2024!" \
    ft=yes \
    ft-over-ds=yes \
    ft-preserve-vlanid=yes

# Configure mobility domain identifier
/interface wifi security set fast-roaming-security \
    ft-mobility-domain=a1b2 \
    ft-r0-key-lifetime=10000 \
    ft-r1-key-holder=00:11:22:33:44:55

# Apply to WiFi interface
/interface wifi add name=wifi-roaming \
    master-interface=wifi1 \
    ssid="Enterprise-Roaming" \
    security=fast-roaming-security \
    disabled=no
```
{% endcode %}

### 802.11r configuration (legacy wireless)

{% code overflow="wrap" %}
```bash
# Create FT security profile for legacy wireless
/interface wireless security-profiles add name=ft-security \
    mode=dynamic-keys \
    authentication-types=wpa2-psk \
    unicast-ciphers=aes-ccm \
    group-ciphers=aes-ccm \
    wpa2-pre-shared-key="RoamingNetwork2024!" \
    ft=yes \
    ft-over-ds=yes

# Configure mobility domain and keys
/interface wireless security-profiles set ft-security \
    ft-mobility-domain=a1b2 \
    ft-r0-key-lifetime=10000 \
    ft-r1-key-holder=00:11:22:33:44:55

# Apply to wireless interface
/interface wireless set wlan1 \
    security-profile=ft-security \
    ssid="Legacy-Roaming" \
    mode=ap-bridge
```
{% endcode %}

### Advanced FT configuration

{% code overflow="wrap" %}
```bash
# Fine-tune FT parameters
/interface wifi security set fast-roaming-security \
    ft-r0-key-lifetime=20000 \
    ft-r1-key-holder=00:11:22:33:44:55 \
    ft-reassociation-timeout=20s

# Configure multiple mobility domains for different areas
/interface wifi security add name=building-a-roaming \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="BuildingA2024!" \
    ft=yes \
    ft-mobility-domain=aa01

/interface wifi security add name=building-b-roaming \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="BuildingB2024!" \
    ft=yes \
    ft-mobility-domain=bb01
```
{% endcode %}

***

## 802.11k Radio Resource Management

### RRM capabilities

**Key features:**
- Neighbor discovery and reporting
- Channel load reporting
- Transmit power control information
- Client steering recommendations
- Interference detection and reporting

**Benefits:**
- Intelligent AP selection
- Reduced scanning time
- Better roaming decisions
- Optimized network performance

### 802.11k configuration (new WiFi package)

{% code overflow="wrap" %}
```bash
# Enable 802.11k on new WiFi package
/interface wifi set wifi1 \
    rrm-neighbor-report=yes \
    rrm-beacon-report=yes \
    rrm-link-measurement=yes

# Configure neighbor reports manually
/interface wifi neighbor add \
    interface=wifi-roaming \
    bssid=00:11:22:33:44:55 \
    ssid="Enterprise-Roaming" \
    channel=36 \
    operating-class=115

# Add multiple neighbors for comprehensive coverage
/interface wifi neighbor add \
    interface=wifi-roaming \
    bssid=00:11:22:33:44:66 \
    ssid="Enterprise-Roaming" \
    channel=44 \
    operating-class=115
```
{% endcode %}

### 802.11k configuration (legacy wireless)

{% code overflow="wrap" %}
```bash
# Enable 802.11k on legacy wireless
/interface wireless set wlan1 \
    wmm-support=enabled \
    rrm=yes

# Configure RRM parameters
/interface wireless set wlan1 \
    rrm-neighbor-report=yes \
    rrm-beacon-report=yes
```
{% endcode %}

***

## 802.11v BSS Transition Management

### BSS Transition features

**Capabilities:**
- Assisted client roaming decisions
- Load balancing between access points
- Network optimization recommendations
- Disassociation imminent notifications
- Preferred candidate list management

**Use cases:**
- High-density environments
- Load distribution optimization
- Network maintenance scenarios
- Emergency evacuations

### 802.11v configuration (new WiFi package)

{% code overflow="wrap" %}
```bash
# Enable 802.11v on new WiFi package
/interface wifi set wifi-roaming \
    bss-transition=yes \
    bss-transition-candidate-list=yes \
    load-balancing=yes

# Configure roaming thresholds
/interface wifi set wifi-roaming \
    roaming-threshold=-70 \
    roaming-hysteresis=5 \
    client-idle-timeout=300s

# Set up candidate AP preferences
/interface wifi candidate add \
    interface=wifi-roaming \
    bssid=00:11:22:33:44:66 \
    preference=255 \
    reason=load-balancing
```
{% endcode %}

### 802.11v configuration (legacy wireless)

{% code overflow="wrap" %}
```bash
# Enable BSS transition on legacy wireless
/interface wireless set wlan1 \
    bss-transition=yes \
    load-balancing-group=office-aps

# Configure load balancing parameters
/interface wireless set wlan1 \
    max-station-count=30 \
    client-idle-timeout=600s
```
{% endcode %}

***

## Advanced roaming optimization

### Client steering and load balancing

{% code overflow="wrap" %}
```bash
# Intelligent client distribution
/interface wifi set wifi1 \
    band-steering=yes \
    load-balancing=yes \
    max-sta-count=50 \
    client-bridge-clone-mac=no

# Configure roaming parameters
/interface wifi set wifi-roaming \
    roaming-threshold=-75 \
    roaming-hysteresis=3 \
    signal-range=-30..-95 \
    keepalive-frames=enabled

# Force client roaming when signal is poor
/interface wifi set wifi-roaming \
    disconnect-threshold=-85 \
    reconnect-timeout=30s
```
{% endcode %}

### Band steering optimization

{% code overflow="wrap" %}
```bash
# Configure intelligent band steering
/interface wifi set wifi1 \
    band-steering=yes \
    band-steering-threshold=-65 \
    preferred-band=5ghz

# Set different thresholds for 2.4GHz and 5GHz
/interface wifi set wifi-2ghz \
    max-sta-count=15 \
    signal-threshold=-80

/interface wifi set wifi-5ghz \
    max-sta-count=30 \
    signal-threshold=-75
```
{% endcode %}

***

## Enterprise roaming with RADIUS

### Centralized authentication for seamless roaming

{% code overflow="wrap" %}
```bash
# Configure RADIUS for enterprise roaming
/radius add service=wireless \
    address=192.168.1.10 \
    secret="radius_roaming_secret" \
    timeout=3s \
    accounting=yes

# Create enterprise roaming security profile
/interface wifi security add name=enterprise-roaming \
    authentication-types=wpa2-eap,wpa3-eap \
    encryption=ccmp \
    eap-methods=eap-tls,peap \
    tls-mode=verify-certificate \
    ft=yes \
    ft-over-ds=yes \
    ft-mobility-domain=e1f2

# Configure PMK caching for faster roaming
/interface wifi security set enterprise-roaming \
    eapol-timeout=30s \
    group-key-update=3600s \
    disable-pmkid=no
```
{% endcode %}

### RADIUS attribute handling for roaming

{% code overflow="wrap" %}
```bash
# Configure RADIUS accounting for roaming tracking
/radius set [find] \
    accounting=yes \
    interim-update=300s

# Enable session tracking
/interface wifi set wifi-roaming \
    accounting=yes \
    interim-accounting=yes

# Configure VLAN assignment via RADIUS
# RADIUS attributes for dynamic VLAN assignment:
# Tunnel-Type = VLAN (13)
# Tunnel-Medium-Type = IEEE-802 (6)
# Tunnel-Private-Group-ID = VLAN-ID
```
{% endcode %}

***

## Mesh roaming configuration

### Seamless roaming in mesh networks

{% code overflow="wrap" %}
```bash
# Configure mesh with roaming support
/interface wifi mesh add name=roaming-mesh \
    mesh-id="enterprise-roaming-mesh" \
    master-interface=wifi1 \
    security=mesh-roaming-security \
    disabled=no

# Mesh security with fast roaming
/interface wifi security add name=mesh-roaming-security \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="MeshRoaming2024!" \
    ft=yes \
    ft-over-ds=yes

# Enable mesh path optimization
/interface wifi mesh set roaming-mesh \
    path-cost-method=airtime \
    path-switching-threshold=5 \
    mesh-retry-timeout=100ms
```
{% endcode %}

### Mesh backhaul optimization

{% code overflow="wrap" %}
```bash
# Optimize mesh for roaming performance
/interface wifi mesh set roaming-mesh \
    forwarding=yes \
    root-mode=yes \
    mesh-portal=yes

# Configure mesh roaming thresholds
/interface wifi set wifi1 \
    mesh-fwding-table-timeout=300s \
    mesh-max-retries=4
```
{% endcode %}

***

## Roaming monitoring and troubleshooting

### Monitor roaming performance

{% code overflow="wrap" %}
```bash
# Monitor client roaming events
/log print follow where topics~"wireless" and message~"roam"

# Check FT key exchange
/log print where message~"ft" and message~"auth"

# View client association history
/interface wifi registration-table print detail where signal-strength<-70

# Check neighbor reports
/interface wifi neighbor print detail

# Monitor roaming statistics
/interface wifi monitor wifi1 duration=60
```
{% endcode %}

### Roaming performance metrics

{% code overflow="wrap" %}
```bash
# Create roaming monitoring script
/system script add name=roaming-monitor source={
    :local roamingClients [/interface wifi registration-table print count-only where uptime<30s];
    :local totalClients [/interface wifi registration-table print count-only];
    :local roamingRate ($roamingClients * 100 / $totalClients);
    
    :if ($roamingRate > 20) do={
        /log warning "High roaming rate detected: $roamingRate%";
    };
    
    /log info "Roaming statistics: $roamingClients/$totalClients clients ($roamingRate%)";
}

# Schedule roaming monitoring
/system scheduler add name=roaming-stats \
    interval=5m \
    on-event=roaming-monitor
```
{% endcode %}

***

## Common roaming issues and solutions

### Sticky client problem

{% code overflow="wrap" %}
```bash
# Force disconnection of weak clients
/interface wifi access-list add \
    interface=wifi-roaming \
    signal-range=..-80 \
    action=reject \
    comment="Force roaming for weak signals"

# Enable client timeout for poor connections
/interface wifi set wifi-roaming \
    client-idle-timeout=180s \
    keepalive-frames=enabled \
    disconnect-threshold=-85

# Implement aggressive roaming thresholds
/interface wifi set wifi-roaming \
    roaming-threshold=-72 \
    roaming-hysteresis=2
```
{% endcode %}

### Roaming authentication failures

{% code overflow="wrap" %}
```bash
# Check FT configuration consistency across APs
/interface wifi security print detail where ft=yes

# Verify mobility domain matching
/interface wifi security print where ft-mobility-domain

# Monitor authentication logs
/log print where message~"auth" and message~"fail"

# Reset client associations if needed
/interface wifi registration-table remove [find mac-address=aa:bb:cc:dd:ee:ff]

# Check key lifetime settings
/interface wifi security print detail where ft-r0-key-lifetime
```
{% endcode %}

### Performance optimization issues

{% code overflow="wrap" %}
```bash
# Analyze roaming delays
/interface wifi monitor wifi1 duration=30

# Check for interference affecting roaming
/interface wifi scan wifi1 duration=10

# Monitor channel utilization
/interface wifi print stats where name~"wifi"

# Check AP load distribution
/interface wifi registration-table print detail
```
{% endcode %}

***

## Load balancing and client distribution

### Optimize client distribution across access points

{% code overflow="wrap" %}
```bash
# Configure intelligent load balancing
/interface wifi set wifi1 \
    load-balancing=yes \
    max-sta-count=25 \
    client-count-limit=20

# Band steering for dual-band optimization
/interface wifi set wifi1 \
    band-steering=yes \
    band-steering-threshold=-65 \
    preferred-band=5ghz

# Time-based load balancing
/system scheduler add name=peak-hours-limit \
    start-time=09:00:00 \
    stop-time=17:00:00 \
    on-event="/interface wifi set wifi-roaming max-sta-count=30" \
    interval=1d

/system scheduler add name=off-hours-limit \
    start-time=17:00:01 \
    stop-time=08:59:59 \
    on-event="/interface wifi set wifi-roaming max-sta-count=50" \
    interval=1d
```
{% endcode %}

### Dynamic load balancing algorithms

{% code overflow="wrap" %}
```bash
# Implement dynamic load balancing script
/system script add name=dynamic-load-balance source={
    :local threshold 25;
    
    :foreach ap in=[/interface wifi print as-value where master-interface] do={
        :local clientCount [/interface wifi registration-table print count-only where ap-tx-ssid=($ap->"ssid")];
        
        :if ($clientCount > $threshold) do={
            # Reduce new client acceptance
            /interface wifi set ($ap->"name") max-sta-count=($clientCount + 5);
            /log info ("Load balancing: Limiting " . ($ap->"name") . " to " . ($clientCount + 5) . " clients");
        } else={
            # Allow normal operation
            /interface wifi set ($ap->"name") max-sta-count=50;
        };
    };
}

# Schedule dynamic load balancing
/system scheduler add name=dynamic-balance \
    interval=2m \
    on-event=dynamic-load-balance
```
{% endcode %}

***

## Site survey for optimal roaming

### Plan access point placement for seamless roaming

{% code overflow="wrap" %}
```bash
# Perform comprehensive site survey
/interface wifi scan wifi1 duration=30

# Analyze coverage overlap for roaming
/interface wifi registration-table print detail where signal-strength>-60

# Optimize channel planning for roaming
/interface wifi set wifi1 \
    channel=36 \
    channel-width=80mhz \
    scan-list=5180-5320,5500-5700

# Set appropriate power levels for coverage overlap
/interface wifi set wifi1 \
    tx-power=15 \
    tx-power-mode=card-rates

# Monitor interference between APs
/interface wifi scan wifi1 where ssid="Enterprise-Roaming"
```
{% endcode %}

### Coverage optimization for roaming

{% code overflow="wrap" %}
```bash
# Create site survey script
/system script add name=site-survey source={
    :local surveyResults "";
    
    :foreach channel in={36;40;44;48} do={
        /interface wifi set wifi1 channel=$channel;
        :delay 5s;
        
        :local scanResults [/interface wifi scan wifi1 duration=10];
        :set surveyResults ($surveyResults . "Channel " . $channel . ": " . [/interface wifi scan wifi1 print count-only] . " APs\n");
    };
    
    /log info $surveyResults;
}

# Generate heat map data
/system script add name=coverage-analysis source={
    :local coverage "";
    
    :foreach client in=[/interface wifi registration-table find] do={
        :local signal [/interface wifi registration-table get $client signal-strength];
        :local mac [/interface wifi registration-table get $client mac-address];
        
        :set coverage ($coverage . $mac . ": " . $signal . "dBm\n");
    };
    
    /log info ("Coverage Analysis:\n" . $coverage);
}
```
{% endcode %}

***

## Roaming with VLANs and network segmentation

### Maintain VLAN assignment during roaming

{% code overflow="wrap" %}
```bash
# Configure VLAN-aware roaming
/interface wifi security set enterprise-roaming \
    ft-preserve-vlanid=yes \
    vlan-mode=use-tag

# Dynamic VLAN assignment via RADIUS
/radius add service=wireless \
    address=192.168.1.10 \
    secret="radius_vlan_secret" \
    accounting=yes

# Configure VLAN attributes in RADIUS responses
# Tunnel-Type = VLAN (13)
# Tunnel-Medium-Type = IEEE-802 (6)  
# Tunnel-Private-Group-ID = VLAN-ID

# Monitor VLAN preservation during roaming
/log print where message~"vlan" and message~"roam"
```
{% endcode %}

### Multi-tenant roaming with VLAN isolation

{% code overflow="wrap" %}
```bash
# Configure tenant-specific roaming domains
/interface wifi security add name=tenant-a-roaming \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="TenantA2024!" \
    ft=yes \
    ft-mobility-domain=aa01 \
    vlan-id=100

/interface wifi security add name=tenant-b-roaming \
    authentication-types=wpa3-psk \
    encryption=ccmp \
    passphrase="TenantB2024!" \
    ft=yes \
    ft-mobility-domain=bb01 \
    vlan-id=200

# Create tenant-specific WiFi interfaces
/interface wifi add name=wifi-tenant-a \
    master-interface=wifi1 \
    ssid="Tenant-A-Network" \
    security=tenant-a-roaming \
    vlan-id=100

/interface wifi add name=wifi-tenant-b \
    master-interface=wifi1 \
    ssid="Tenant-B-Network" \
    security=tenant-b-roaming \
    vlan-id=200
```
{% endcode %}

***

<details>

<summary>Show complete enterprise roaming deployment</summary>

{% code overflow="wrap" %}
```bash
# 1. Configure RADIUS server for centralized authentication
/radius add service=wireless address=192.168.1.10 secret="enterprise_radius_secret" timeout=3s accounting=yes

# 2. Create enterprise roaming security profile
/interface wifi security add name=enterprise-ft-roaming \
    authentication-types=wpa2-eap,wpa3-eap \
    encryption=ccmp \
    eap-methods=eap-tls,peap \
    tls-mode=verify-certificate \
    ft=yes \
    ft-over-ds=yes \
    ft-mobility-domain=ent1 \
    ft-r0-key-lifetime=20000 \
    ft-preserve-vlanid=yes

# 3. Create WiFi interface with roaming
/interface wifi add name=wifi-enterprise \
    master-interface=wifi1 \
    ssid="Enterprise-Roaming" \
    security=enterprise-ft-roaming \
    disabled=no

# 4. Enable 802.11k and 802.11v
/interface wifi set wifi1 \
    rrm-neighbor-report=yes \
    rrm-beacon-report=yes \
    rrm-link-measurement=yes

/interface wifi set wifi-enterprise \
    bss-transition=yes \
    bss-transition-candidate-list=yes \
    load-balancing=yes \
    roaming-threshold=-70 \
    roaming-hysteresis=5

# 5. Configure band steering and load balancing
/interface wifi set wifi1 \
    band-steering=yes \
    load-balancing=yes \
    max-sta-count=40 \
    client-idle-timeout=300s

# 6. Add neighbor APs for 802.11k
/interface wifi neighbor add interface=wifi-enterprise bssid=00:11:22:33:44:55 ssid="Enterprise-Roaming" channel=36 operating-class=115
/interface wifi neighbor add interface=wifi-enterprise bssid=00:11:22:33:44:66 ssid="Enterprise-Roaming" channel=44 operating-class=115

# 7. Configure monitoring and logging
/system logging add topics=wireless,info action=memory
/system logging add topics=wireless,warning action=disk

# 8. Create roaming performance monitoring script
/system script add name=roaming-performance source={
    :local totalClients [/interface wifi registration-table print count-only];
    :local weakClients [/interface wifi registration-table print count-only where signal-strength<-75];
    :local roamingRate ($weakClients * 100 / $totalClients);
    
    /log info ("Roaming Performance: " . $totalClients . " total clients, " . $weakClients . " weak signals (" . $roamingRate . "%)");
    
    :if ($roamingRate > 30) do={
        /log warning "High percentage of weak signal clients detected";
    };
}

# 9. Schedule performance monitoring
/system scheduler add name=roaming-monitor interval=5m on-event=roaming-performance

# 10. Configure client steering for poor connections
/interface wifi access-list add interface=wifi-enterprise signal-range=..-85 action=reject comment="Force roaming threshold"
```
{% endcode %}

</details>

## Roaming best practices

### Design recommendations

1. **Overlap planning** - Ensure 15-20% signal overlap between APs
2. **Channel planning** - Use non-overlapping channels (1, 6, 11 for 2.4GHz)
3. **Power optimization** - Adjust power for proper coverage without interference
4. **Mobility domains** - Use consistent domains across roaming areas
5. **Security consistency** - Maintain identical security settings across APs

### Performance optimization

1. **Thresholds** - Set appropriate roaming thresholds (-70 to -75 dBm)
2. **Timeouts** - Configure reasonable client idle timeouts
3. **Load balancing** - Implement intelligent client distribution
4. **Monitoring** - Regular performance analysis and optimization
5. **Testing** - Validate roaming performance with real devices

### Troubleshooting guidelines

1. **Signal analysis** - Monitor signal strength and coverage gaps
2. **Authentication logs** - Check for FT key exchange issues
3. **Client behavior** - Analyze sticky client problems
4. **Performance metrics** - Track roaming success rates and delays
5. **Network optimization** - Continuous improvement based on monitoring data