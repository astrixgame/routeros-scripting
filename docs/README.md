---
description: Comprehensive RouterOS documentation covering networking, security, VPNs, and advanced configurations for MikroTik devices running RouterOS v7+.
icon: hand-wave
cover: .gitbook/assets/69481148-network-wallpapers.jpg
coverY: 9
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# RouterOS Configuration Guide

{% hint style="success" %}
**Welcome to the comprehensive RouterOS documentation!** This guide covers everything from basic setup to advanced enterprise configurations for MikroTik devices running RouterOS v7+.
{% endhint %}

This documentation provides practical, tested configurations for RouterOS networking, security, VPNs, and system administration. Each section includes both WinBox GUI instructions and command-line interface (CLI) examples.

***

## What you'll find here

### 🚀 **Getting Started**
- **[Basics](basics/)** - Essential RouterOS setup and configuration procedures
  - Default configuration for secure deployments
  - Connection methods (WinBox, SSH, WebFig, API)
  - System updates and maintenance procedures

### 🔒 **Security & VPNs**
- **[VPNs](vpns/)** - Complete VPN implementation guides
  - OpenVPN servers, clients, and certificate management
  - WireGuard modern VPN setup and configuration
  - Legacy protocols (L2TP/IPsec, PPTP, SSTP) for compatibility
  - ZeroTier software-defined networking

### 🌐 **Networking**
- **[Interfaces](ip-interfaces/interfaces/)** - All interface types and configurations
  - Bridge interfaces for switching and VLANs
  - WiFi setup for both new and legacy wireless packages
  - PPPoE, LTE, VLAN, and tunnel interfaces (VXLAN, GRE, EoIP)

- **[Routing](routing/)** - Comprehensive routing protocols and techniques
  - Static routing and policy-based routing
  - Dynamic protocols (OSPF, BGP, RIP)
  - ECMP, MPLS, VRF, and multicast routing

### 🛡️ **Firewall & Traffic Control**
- **[Firewall](ip-interfaces/firewall/)** - Advanced security and traffic filtering
  - NAT configuration and advanced scenarios
  - Mangle rules for QoS and traffic manipulation
  - Layer-7 protocol filtering and application control

### 📡 **WiFi & Wireless**
- **[WiFi](wifi/)** - Modern wireless networking and management
  - WiFi 6/6E configuration with new RouterOS package
  - Legacy wireless support for older hardware
  - CAP mode for centralized access point management
  - Enterprise roaming with 802.11r/k/v standards

### ⚙️ **Advanced Topics**
- **[Scripting](scripting/)** - RouterOS automation and custom scripts
- **[Advanced](advanced/)** - Enterprise features and complex scenarios
  - Failover and load balancing configurations
  - High availability and redundancy setups

***

## Documentation features

### 🎯 **Comprehensive Coverage**
Every major RouterOS feature is documented with:
- **Step-by-step instructions** for both beginners and experts
- **Real-world examples** tested in production environments
- **Security best practices** for enterprise deployments
- **Troubleshooting guides** for common issues

### 🔧 **Dual Interface Support**
All configurations include:
- **WinBox GUI** - Visual configuration steps
- **Command Line** - CLI commands for automation and scripting
- **Both approaches** side-by-side for learning preferences

### 📱 **Modern RouterOS Focus**
- **RouterOS v7+** - Latest features and syntax throughout
- **Backward compatibility** - Legacy support where needed
- **Hardware coverage** - Both new and older MikroTik devices
- **Performance optimization** - Current best practices

***

## Quick start guide

### 1. **Initial Setup**
Start with the [Basics](basics/) section to:
- Configure secure default settings
- Establish management connectivity
- Set up system updates and monitoring

### 2. **Choose Your Path**
Based on your needs:

**Home/Small Office:**
- Basic [Interface](ip-interfaces/interfaces/) setup
- Simple [Firewall](ip-interfaces/firewall/) rules
- [WiFi](wifi/) access point configuration

**Enterprise/Advanced:**
- [VPN](vpns/) infrastructure deployment
- [Routing](routing/) protocol implementation
- [CAP mode](wifi/cap-mode.md) wireless management
- [Advanced](advanced/) failover and load balancing

**Service Provider:**
- MPLS and VRF configurations
- BGP routing implementations
- High-availability setups

### 3. **Ongoing Management**
- [Scripting](scripting/) for automation
- Monitoring and maintenance procedures
- Security hardening and updates

***

## RouterOS version compatibility

### **Primary Focus: RouterOS v7+**
This documentation primarily targets **RouterOS v7** and later versions, featuring:
- Modern command syntax and structure
- New WiFi package (wifi-qcom) for Qualcomm devices
- Enhanced security features and WPA3 support
- Improved performance and stability

### **Legacy Support**
Where applicable, documentation includes:
- **RouterOS v6** compatibility notes
- **Legacy wireless** package configurations
- **Migration guides** for upgrading from older versions
- **Hardware-specific** considerations

***

## Hardware compatibility

### **Supported Devices**
This guide covers configurations for:

**Modern MikroTik Hardware:**
- hAP series (ax, ac³, ac²)
- CRS series switches
- CCR series routers
- RB series boards (newer models)

**Legacy Hardware:**
- Older RouterBoard models
- Atheros-based wireless devices
- Legacy switch configurations

**Cloud and Virtual:**
- Cloud Hosted Router (CHR)
- Virtual machine deployments
- Container environments

***

## Best practices highlighted

### 🔒 **Security First**
- **Default deny** firewall policies
- **Strong authentication** methods throughout
- **Certificate-based** VPN configurations
- **Regular security** updates and monitoring

### 📈 **Performance Optimization**
- **Hardware offloading** where available
- **Efficient routing** table management
- **QoS implementation** for traffic prioritization
- **Resource monitoring** and capacity planning

### 🏢 **Enterprise Ready**
- **Scalable configurations** for growth
- **High availability** design patterns
- **Centralized management** approaches
- **Standardized deployment** procedures

***

## How to use this documentation

### **Navigation Structure**
Each section is organized for progressive learning:
1. **Overview** - Concepts and theory
2. **Basic Configuration** - Essential setup steps
3. **Advanced Features** - Complex scenarios and optimization
4. **Troubleshooting** - Common issues and solutions
5. **Best Practices** - Production recommendations

### **Code Examples**
All configuration examples include:
- **Complete commands** - Copy-paste ready
- **Explanatory comments** - Understanding each step
- **Variable placeholders** - Customize for your environment
- **Security considerations** - Safe deployment practices

### **Cross-References**
Related topics are linked throughout:
- **Dependencies** clearly marked
- **Related configurations** referenced
- **Alternative approaches** presented
- **Migration paths** documented

***

## Contributing and feedback

### **Community Driven**
This documentation benefits from:
- **Real-world testing** in production environments
- **Community feedback** and improvements
- **Regular updates** for new RouterOS features
- **Practical experience** from network professionals

### **Staying Current**
- **RouterOS updates** incorporated regularly
- **Security advisories** addressed promptly
- **Hardware support** expanded as available
- **Best practices** evolved based on field experience

***

## Getting help

### **Within This Documentation**
- **Search functionality** for quick topic location
- **Comprehensive index** of all covered topics
- **Troubleshooting sections** in each major area
- **Cross-referenced** related configurations

### **Additional Resources**
- **MikroTik Official Documentation** - Primary reference
- **Community Forums** - Peer support and discussion
- **Training Materials** - Official certification paths
- **Professional Services** - Enterprise implementation support

***

<details>

<summary>Quick reference: Common tasks</summary>

**Initial Setup:**
```bash
# Basic security configuration
/ip service disable telnet,ftp,www
/ip service set ssh port=2222
/user set admin password="SecurePassword123!"
```

**WiFi Access Point:**
```bash
# Modern WiFi package
/interface wifi security add name=home-wifi authentication-types=wpa3-psk encryption=ccmp passphrase="WiFiPassword2024!"
/interface wifi add name=wifi-home master-interface=wifi1 ssid="HomeNetwork" security=home-wifi
```

**Basic Firewall:**
```bash
# Input chain protection
/ip firewall filter add chain=input action=accept connection-state=established,related
/ip firewall filter add chain=input action=accept src-address=192.168.88.0/24
/ip firewall filter add chain=input action=drop
```

**NAT Configuration:**
```bash
# Basic masquerade
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

</details>

**Ready to get started?** Begin with the [Basics](basics/) section for your first RouterOS configuration, or jump directly to the topic that interests you most.

**Looking for something specific?** Use the search function or browse the detailed table of contents to find exactly what you need.

---

*This documentation is maintained and updated regularly to reflect the latest RouterOS features and best practices. Last updated: November 2025*
