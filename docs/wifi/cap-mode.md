---
description: This section covers how to configure Controlled Access Point (CAP) mode for centralized WiFi management using CAPsMAN or WiFi provisioning.
icon: tower-control
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

# CAP Mode

{% hint style="info" %}
CAP (Controlled Access Point) mode allows centralized management of multiple wireless access points from a single controller. RouterOS supports both legacy CAPsMAN and the new WiFi provisioning system for v7+.
{% endhint %}

In WinBox you can configure CAP mode in **CAPsMAN** (legacy) or **WiFi** > **Provisioning** (v7+), or use terminal commands `/caps-man` or `/interface wifi provisioning`

CAP mode is essential for managing multiple access points in enterprise environments, providing centralized configuration, monitoring, and seamless roaming.

***

## CAP mode overview

### Traditional CAPsMAN (legacy)

**Features:**
- Centralized access point management
- Works with legacy wireless package
- Uses `/caps-man` commands
- Supports older RouterBoard models

**Limitations:**
- Legacy wireless package only
- Limited scalability compared to new system
- No WiFi 6 support

### WiFi Provisioning (RouterOS v7+)

**Features:**
- Modern centralized WiFi management
- Works with new WiFi package
- Uses `/interface wifi provisioning` commands
- Supports WiFi 6/6E features
- Better performance and scalability

**Advantages:**
- Cloud-ready architecture
- Enhanced security features
- Better roaming support
- Simplified configuration

***

## Setting up WiFi provisioning controller (v7+)

### Controller configuration

{% code overflow="wrap" %}
```bash
# Enable WiFi provisioning service
/interface wifi provisioning enable

# Configure provisioning settings
/interface wifi provisioning set \
    enabled=yes \
    supported-bands=2ghz-n,5ghz-ac,5ghz-ax \
    country=united_states

# Create provisioning profile
/interface wifi provisioning add \
    master-configuration=controller-config \
    slave-configurations=cap-config \
    action=create-enabled \
    name-format=cap-office
```
{% endcode %}

### Master configuration (Controller)

{% code overflow="wrap" %}
```bash
# Create master WiFi configuration
/interface wifi configuration add \
    name=controller-config \
    mode=controller \
    ssid="Enterprise-WiFi" \
    security.authentication-types=wpa2-psk,wpa3-psk \
    security.encryption=ccmp \
    security.passphrase="EnterprisePassword123!" \
    country=united_states \
    channel=auto \
    tx-power=20

# Configure advanced controller settings
/interface wifi configuration set controller-config \
    multicast-helper=full \
    load-balancing=yes \
    band-steering=yes
```
{% endcode %}

### CAP configuration template

{% code overflow="wrap" %}
```bash
# Create CAP slave configuration
/interface wifi configuration add \
    name=cap-config \
    mode=station \
    ssid="Management-Network" \
    security.authentication-types=wpa2-psk \
    security.passphrase="CapManagement123!" \
    disabled=no

# Configure CAP specific settings
/interface wifi configuration set cap-config \
    manager-addresses=192.168.1.1 \
    manager-certificate-tag=cap-cert
```
{% endcode %}

***

## Legacy CAPsMAN setup

### CAPsMAN controller configuration

{% code overflow="wrap" %}
```bash
# Enable CAPsMAN manager
/caps-man manager set enabled=yes upgrade-policy=suggest-same-version

# Configure manager interface
/caps-man manager interface set [ find default=yes ] interface=bridge1 forbid=no

# Create security profile
/caps-man security add name=enterprise-security \
    authentication-types=wpa2-psk \
    encryption=aes-ccm \
    passphrase="EnterpriseSecure2024!"

# Create channel configuration
/caps-man channel add name=2ghz-channels band=2ghz-b/g/n extension-channel=XX frequency=2412,2437,2462 tx-power=20
/caps-man channel add name=5ghz-channels band=5ghz-a/n/ac extension-channel=XXXX frequency=5180,5200,5220,5240 tx-power=20

# Create datapath configuration  
/caps-man datapath add name=enterprise-datapath bridge=bridge1 client-to-client-forwarding=yes local-forwarding=yes

# Create configuration profile
/caps-man configuration add \
    name=enterprise-ap \
    mode=ap \
    ssid="Enterprise-WiFi" \
    security=enterprise-security \
    channel=2ghz-channels,5ghz-channels \
    datapath=enterprise-datapath \
    country=united_states
```
{% endcode %}

### CAP client setup (legacy)

{% code overflow="wrap" %}
```bash
# Enable CAP mode on access point
/interface wireless cap set enabled=yes interfaces=wlan1,wlan2

# Configure CAP connection
/interface wireless cap set \
    caps-man-addresses=192.168.1.1 \
    certificate=none \
    discovery-interfaces=bridge1

# Set CAP identity
/system identity set name="AP-Office-Floor2"
```
{% endcode %}

***

## Multi-site CAP deployment

### Controller hierarchy

{% code overflow="wrap" %}
```bash
# Regional controller configuration
/interface wifi provisioning add \
    name=region-controller \
    master-configuration=regional-config \
    coverage-area="North America" \
    timezone=America/New_York

# Site-specific configurations
/interface wifi provisioning add \
    name=office-site1 \
    master-configuration=office-config \
    parent-controller=region-controller \
    site-id="NYC-Office-01"

/interface wifi provisioning add \
    name=office-site2 \
    master-configuration=office-config \
    parent-controller=region-controller \
    site-id="NYC-Office-02"
```
{% endcode %}

### VLAN and network segmentation

{% code overflow="wrap" %}
```bash
# Create network profiles for different user groups
/interface wifi configuration add \
    name=corporate-profile \
    mode=ap \
    ssid="Corporate-Net" \
    security.authentication-types=wpa3-psk \
    security.passphrase="CorpNet2024!" \
    datapath.vlan-id=10

/interface wifi configuration add \
    name=guest-profile \
    mode=ap \
    ssid="Guest-WiFi" \
    security.authentication-types=wpa2-psk \
    security.passphrase="GuestAccess2024" \
    datapath.vlan-id=20 \
    datapath.client-isolation=yes

/interface wifi configuration add \
    name=iot-profile \
    mode=ap \
    ssid="IoT-Devices" \
    security.authentication-types=wpa2-psk \
    security.passphrase="IoTSecure2024" \
    datapath.vlan-id=30
```
{% endcode %}

***

## Advanced CAP features

### Seamless roaming configuration

{% code overflow="wrap" %}
```bash
# Enable fast roaming (802.11r)
/interface wifi configuration set corporate-profile \
    security.ft=yes \
    security.ft-over-ds=yes \
    security.ft-preserve-vlanid=yes

# Configure roaming thresholds
/interface wifi configuration set corporate-profile \
    roaming-threshold=-70 \
    roaming-hysteresis=5
```
{% endcode %}

### Load balancing and band steering

{% code overflow="wrap" %}
```bash
# Enable intelligent client distribution
/interface wifi configuration set corporate-profile \
    load-balancing=yes \
    band-steering=yes \
    max-sta-count=50

# Configure client limits per radio
/interface wifi configuration add \
    name=high-density-config \
    mode=ap \
    ssid="HD-Corporate" \
    load-balancing=yes \
    max-sta-count=100 \
    client-idle-timeout=300s
```
{% endcode %}

### Mesh networking with CAPs

{% code overflow="wrap" %}
```bash
# Configure mesh backhaul
/interface wifi configuration add \
    name=mesh-backhaul \
    mode=station-bridge \
    ssid="MeshBackhaul" \
    security.authentication-types=wpa3-psk \
    security.passphrase="MeshSecure2024!" \
    mesh.mesh-id="enterprise-mesh"

# Enable mesh forwarding
/interface wifi configuration set mesh-backhaul \
    mesh.forwarding=yes \
    mesh.root-mode=yes
```
{% endcode %}

***

## CAP monitoring and management

### Monitor CAP status

{% code overflow="wrap" %}
```bash
# Check connected CAPs (WiFi provisioning)
/interface wifi provisioning print

# Monitor CAP performance
/interface wifi provisioning monitor [find name=cap-office]

# Check client distribution
/interface wifi registration-table print where ap~"cap-office"

# Legacy CAPsMAN monitoring
/caps-man remote-cap print detail
/caps-man registration-table print
```
{% endcode %}

### Centralized logging and alerts

{% code overflow="wrap" %}
```bash
# Configure centralized logging
/system logging add topics=wireless,info action=remote remote=192.168.1.100:514

# Create alerts for CAP disconnections
/system script add name=cap-alert source={
    :local capCount [/caps-man remote-cap print count-only where connection-state=enabled];
    :if ($capCount < 5) do={
        /tool e-mail send to="admin@company.com" \
            subject="CAP Alert: Low CAP Count" \
            body="Only $capCount CAPs connected";
    }
}

# Schedule regular CAP monitoring
/system scheduler add name=cap-monitor interval=5m on-event=cap-alert
```
{% endcode %}

### Firmware and configuration updates

{% code overflow="wrap" %}
```bash
# Update all CAPs via provisioning
/interface wifi provisioning upgrade-firmware [find]

# Push configuration updates
/interface wifi provisioning push-configuration [find name~"office"]

# Legacy CAPsMAN updates
/caps-man manager set upgrade-policy=require-same-version
/caps-man remote-cap upgrade [find]
```
{% endcode %}

***

## Troubleshooting CAP deployments

### CAP connectivity issues

{% code overflow="wrap" %}
```bash
# Check CAP discovery
/interface wifi provisioning print detail where connection-state!=connected

# Test manager connectivity
/ping 192.168.1.1 interface=bridge1

# Check certificates (if using secure connection)
/certificate print where name~"cap"

# Legacy CAPsMAN troubleshooting
/caps-man remote-cap print detail where connection-state!=enabled
/log print where topics~"caps"
```
{% endcode %}

### Configuration sync problems

{% code overflow="wrap" %}
```bash
# Force configuration sync
/interface wifi provisioning sync-configuration [find]

# Check configuration differences
/interface wifi provisioning print configuration-status

# Reset CAP to default and reprovision
/interface wifi provisioning reset-configuration [find name=problematic-cap]

# Legacy CAPsMAN sync
/caps-man remote-cap provision [find]
```
{% endcode %}

### Performance troubleshooting

{% code overflow="wrap" %}
```bash
# Monitor controller load
/system resource print
/interface monitor-traffic interface=bridge1

# Check CAP radio utilization
/interface wifi monitor [find where master-interface]

# Analyze client roaming patterns
/log print where topics~"wifi" and message~"roam"
```
{% endcode %}

***

## Security considerations for CAP

### Secure controller communication

{% code overflow="wrap" %}
```bash
# Generate certificates for secure CAP communication
/certificate add name=ca-cert common-name=enterprise-ca
/certificate sign ca-cert ca-crl-host=192.168.1.1

# Create CAP certificate template
/certificate add name=cap-template common-name=cap-device template=yes
/certificate sign cap-template ca=ca-cert

# Configure secure provisioning
/interface wifi provisioning set certificate=ca-cert
```
{% endcode %}

### Access control and isolation

{% code overflow="wrap" %}
```bash
# Implement MAC-based access control
/interface wifi access-list add \
    interface=corporate-profile \
    mac-address=00:11:22:33:44:55 \
    action=accept \
    comment="Approved corporate device"

# Configure client isolation per network
/interface wifi configuration set guest-profile \
    datapath.client-isolation=yes \
    datapath.local-forwarding=no

# Implement time-based access control
/system scheduler add name=guest-disable \
    start-time=18:00:00 \
    on-event="/interface wifi set guest-profile disabled=yes" \
    comment="Disable guest network after hours"
```
{% endcode %}

***

## CAP deployment best practices

### Network design recommendations

1. **Controller placement** - Central location with reliable connectivity
2. **Redundancy** - Deploy backup controllers for high availability
3. **Network segmentation** - Separate management and user traffic
4. **Capacity planning** - Size controller based on CAP count and features
5. **Security** - Use certificates and encrypted communication

### Operational procedures

{% code overflow="wrap" %}
```bash
# Create standardized CAP naming convention
/system identity set name="CAP-[SITE]-[FLOOR]-[ROOM]"

# Implement configuration backup
/system script add name=cap-backup source={
    /interface wifi provisioning export file="cap-config-backup-$[/system clock get date]";
    /system backup save name="controller-backup-$[/system clock get date]";
}

# Schedule regular backups
/system scheduler add name=weekly-backup interval=7d start-time=02:00:00 on-event=cap-backup
```
{% endcode %}

### Performance optimization

{% code overflow="wrap" %}
```bash
# Optimize controller for high CAP count
/interface wifi provisioning set \
    max-cap-count=100 \
    keepalive-timeout=30s \
    discovery-timeout=60s

# Configure efficient channel allocation
/interface wifi configuration set [find mode=ap] \
    channel.frequency=auto \
    channel.width=20/40/80mhz \
    channel.skip-dfs-channels=yes
```
{% endcode %}

***

<details>

<summary>Show complete enterprise CAP deployment</summary>

{% code overflow="wrap" %}
```bash
# 1. Configure controller (CAPsMAN legacy example)
/caps-man manager set enabled=yes upgrade-policy=suggest-same-version
/caps-man manager interface set [ find default=yes ] interface=bridge1

# 2. Create security profiles for different user groups
/caps-man security add name=corporate-security authentication-types=wpa3-psk encryption=aes-ccm passphrase="CorpSecure2024!"
/caps-man security add name=guest-security authentication-types=wpa2-psk encryption=aes-ccm passphrase="GuestWiFi2024"
/caps-man security add name=iot-security authentication-types=wpa2-psk encryption=aes-ccm passphrase="IoTDevices2024"

# 3. Create channel configurations
/caps-man channel add name=2ghz-auto band=2ghz-b/g/n frequency=2412,2437,2462 tx-power=20 extension-channel=XX
/caps-man channel add name=5ghz-auto band=5ghz-a/n/ac frequency=5180,5200,5220,5240,5260,5280 tx-power=20 extension-channel=XXXX

# 4. Create datapath configurations with VLAN support
/caps-man datapath add name=corporate-path bridge=bridge1 vlan-mode=use-tag vlan-id=10 client-to-client-forwarding=yes
/caps-man datapath add name=guest-path bridge=bridge1 vlan-mode=use-tag vlan-id=20 client-to-client-forwarding=no
/caps-man datapath add name=iot-path bridge=bridge1 vlan-mode=use-tag vlan-id=30 client-to-client-forwarding=no

# 5. Create configuration profiles
/caps-man configuration add name=corporate-ap mode=ap ssid="Corporate-WiFi" security=corporate-security \
    channel=2ghz-auto,5ghz-auto datapath=corporate-path country=united_states load-balancing-group=corporate

/caps-man configuration add name=guest-ap mode=ap ssid="Guest-Network" security=guest-security \
    channel=2ghz-auto,5ghz-auto datapath=guest-path country=united_states max-sta-count=20

/caps-man configuration add name=iot-ap mode=ap ssid="IoT-Network" security=iot-security \
    channel=2ghz-auto datapath=iot-path country=united_states max-sta-count=50

# 6. Configure provisioning rules
/caps-man provisioning add action=create-dynamic-enabled master-configuration=corporate-ap \
    slave-configurations=guest-ap,iot-ap name-format=identity

# 7. Enable CAP mode on access points
# Run on each CAP device:
/interface wireless cap set enabled=yes interfaces=wlan1,wlan2 caps-man-addresses=192.168.1.1
/system identity set name="CAP-NYC-Floor1-Room101"

# 8. Configure monitoring and alerts
/system logging add topics=wireless,caps action=memory
/system script add name=cap-monitor source={
    :local activeCaps [/caps-man remote-cap print count-only where connection-state=enabled];
    :if ($activeCaps < 10) do={
        /log warning "Low CAP count: $activeCaps active";
    }
}
/system scheduler add name=cap-check interval=5m on-event=cap-monitor
```
{% endcode %}

</details>

## Migration from CAPsMAN to WiFi provisioning

### Planning the migration

{% code overflow="wrap" %}
```bash
# 1. Export existing CAPsMAN configuration
/caps-man export file=capsman-backup

# 2. Document current setup
/caps-man configuration print detail
/caps-man security print detail  
/caps-man datapath print detail

# 3. Plan new WiFi provisioning structure
# Map CAPsMAN configs to new WiFi configurations
```
{% endcode %}

### Migration process

{% code overflow="wrap" %}
```bash
# 1. Prepare new WiFi provisioning (parallel to existing CAPsMAN)
/interface wifi provisioning enable

# 2. Create equivalent configurations
# Convert CAPsMAN security profiles to WiFi security
# Convert datapaths to WiFi datapath configurations
# Convert configurations to WiFi configurations

# 3. Gradual migration
# Migrate CAPs one by one or site by site
# Test thoroughly before proceeding

# 4. Decommission CAPsMAN
/caps-man manager set enabled=no
```
{% endcode %}

This comprehensive CAP mode documentation covers both legacy CAPsMAN and modern WiFi provisioning systems, providing enterprise-grade centralized wireless management capabilities for RouterOS deployments.