---
description: Virtual Routing and Forwarding (VRF) enables network virtualization by creating multiple isolated routing domains within a single router.
icon: layers-3
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

# Virtual Routing and Forwarding (VRF)

{% hint style="info" %}
VRF technology allows a single RouterOS device to maintain multiple separate routing tables, enabling network virtualization, multi-tenancy, and traffic isolation for complex networking scenarios.
{% endhint %}

VRF in RouterOS v7+ provides complete routing table isolation, supporting MPLS VPNs, multi-tenant networks, and advanced traffic engineering with per-VRF routing protocols and policies.

***

## VRF fundamentals

### How VRF works

**Core concepts:**
- **Virtual routing instances** - Separate routing tables per VRF
- **Interface assignment** - Interfaces belong to specific VRF instances
- **Route isolation** - Routes in one VRF are invisible to others
- **Routing protocols** - Each VRF can run independent routing protocols
- **Route targets** - Control route import/export between VRFs (MPLS VPNs)

**VRF benefits:**
- **Network segmentation** - Complete traffic isolation between tenants
- **Simplified management** - Single device supporting multiple customers
- **Reduced hardware costs** - Virtualization instead of physical separation
- **Flexible routing policies** - Different routing policies per VRF
- **Service provider enablement** - Foundation for MPLS L3VPNs

### VRF vs traditional routing

{% code overflow="wrap" %}
```bash
# Traditional routing (single global table)
/ip route add dst-address=192.168.1.0/24 gateway=10.0.0.1
# All interfaces share the same routing table

# VRF routing (multiple isolated tables)
/ip route vrf add interfaces=ether2 routing-mark=customer-a
/ip route add dst-address=192.168.1.0/24 gateway=10.0.0.1 routing-table=customer-a
# Interface ether2 uses isolated routing table "customer-a"

# Route isolation verification
/ip route print  # Shows global routing table
/ip route print where routing-table=customer-a  # Shows VRF-specific routes
```
{% endcode %}

***

## Basic VRF configuration

### Simple VRF setup

Create basic VRF instances for customer isolation:

{% code overflow="wrap" %}
```bash
# Create VRF for Customer A
/ip route vrf add interfaces=ether2 routing-mark=customer-a \
    comment="Customer A isolated routing"

# Create VRF for Customer B  
/ip route vrf add interfaces=ether3 routing-mark=customer-b \
    comment="Customer B isolated routing"

# Assign IP addresses to VRF interfaces
/ip address add address=192.168.10.1/24 interface=ether2 comment="Customer A network"
/ip address add address=192.168.20.1/24 interface=ether3 comment="Customer B network"

# Add routes to specific VRF tables
/ip route add dst-address=10.10.0.0/16 gateway=192.168.10.100 \
    routing-table=customer-a comment="Customer A internal networks"

/ip route add dst-address=10.20.0.0/16 gateway=192.168.20.100 \
    routing-table=customer-b comment="Customer B internal networks"

# Verify VRF configuration
/ip route vrf print
/ip route print where routing-table=customer-a
/ip route print where routing-table=customer-b
```
{% endcode %}

### Multi-interface VRF

VRF spanning multiple interfaces:

{% code overflow="wrap" %}
```bash
# Create VRF with multiple interfaces (branch offices)
/ip route vrf add interfaces=ether2,ether4,ether5 routing-mark=enterprise-customer \
    comment="Enterprise customer with multiple sites"

# Configure IP addresses for all VRF interfaces
/ip address add address=192.168.100.1/24 interface=ether2 comment="HQ connection"
/ip address add address=192.168.101.1/24 interface=ether4 comment="Branch A connection"  
/ip address add address=192.168.102.1/24 interface=ether5 comment="Branch B connection"

# Add routes for inter-site connectivity
/ip route add dst-address=10.100.0.0/16 gateway=192.168.100.100 \
    routing-table=enterprise-customer comment="HQ networks"

/ip route add dst-address=10.101.0.0/16 gateway=192.168.101.100 \
    routing-table=enterprise-customer comment="Branch A networks"

/ip route add dst-address=10.102.0.0/16 gateway=192.168.102.100 \
    routing-table=enterprise-customer comment="Branch B networks"

# Verify inter-VRF connectivity (within same VRF)
/ping 192.168.101.100 routing-table=enterprise-customer
```
{% endcode %}

***

## Advanced VRF scenarios

### VRF with dynamic routing

Running routing protocols within VRF instances:

{% code overflow="wrap" %}
```bash
# OSPF within VRF for dynamic routing
/ip route vrf add interfaces=ether2,ether3 routing-mark=ospf-customer \
    comment="Customer with OSPF routing"

# Create OSPF instance for the VRF
/routing ospf instance add name=customer-ospf router-id=10.1.1.1 \
    vrf=ospf-customer disabled=no comment="Customer OSPF instance"

# Configure OSPF area for the VRF
/routing ospf area add name=customer-backbone area-id=0.0.0.0 \
    instance=customer-ospf comment="Customer backbone area"

# Add VRF interfaces to OSPF
/routing ospf interface-template add area=customer-backbone \
    interfaces=ether2,ether3 type=broadcast disabled=no \
    comment="Customer OSPF interfaces"

# Verify OSPF within VRF
/routing ospf neighbor print where instance=customer-ospf
/ip route print where routing-table=ospf-customer and ospf=yes

# BGP within VRF (for larger customers)
/routing bgp template add name=customer-bgp as=65100 \
    router-id=10.1.1.1 vrf=ospf-customer disabled=no

/routing bgp connection add template=customer-bgp \
    remote.address=192.168.10.100 remote.as=65200 disabled=no \
    comment="Customer BGP peer"
```
{% endcode %}

### Inter-VRF route leaking

Controlled communication between VRF instances:

{% code overflow="wrap" %}
```bash
# Scenario: Shared services VRF needs access from customer VRFs
# Create shared services VRF
/ip route vrf add interfaces=ether4 routing-mark=shared-services \
    comment="Shared services (DNS, NTP, etc.)"

/ip address add address=172.16.1.1/24 interface=ether4 comment="Shared services network"

# Add routes for shared services
/ip route add dst-address=172.16.10.0/24 gateway=172.16.1.100 \
    routing-table=shared-services comment="DNS servers"

/ip route add dst-address=172.16.20.0/24 gateway=172.16.1.200 \
    routing-table=shared-services comment="NTP servers"

# Route leaking: Allow customer VRFs to access shared services
# Add shared service routes to customer VRF routing tables
/ip route add dst-address=172.16.10.0/24 gateway=172.16.1.100 \
    routing-table=customer-a comment="DNS access from Customer A"

/ip route add dst-address=172.16.20.0/24 gateway=172.16.1.200 \
    routing-table=customer-a comment="NTP access from Customer A"

# Selective route leaking with routing filters
/routing filter rule add chain=vrf-leak-in src-address=192.168.10.0/24 \
    action=accept comment="Allow Customer A to shared services"

/routing filter rule add chain=vrf-leak-in action=discard \
    comment="Block other VRF access"
```
{% endcode %}

### VRF with MPLS integration

Integrate VRF with MPLS for service provider scenarios:

{% code overflow="wrap" %}
```bash
# MPLS VPN configuration with VRF
# Create VRF with route targets for MPLS VPN
/ip route vrf add interfaces=ether5 routing-mark=mpls-customer \
    route-distinguisher=65001:100 \
    import-route-targets=65001:100,65001:999 \
    export-route-targets=65001:100 \
    comment="MPLS VPN Customer"

# Configure customer interface
/ip address add address=192.168.150.1/30 interface=ether5 \
    comment="MPLS customer CE connection"

# Customer routes (will be advertised via MP-BGP)
/ip route add dst-address=10.150.0.0/16 gateway=192.168.150.2 \
    routing-table=mpls-customer comment="Customer networks"

# BGP configuration for VPNv4 route advertisement
/routing bgp template add name=mpls-pe as=65001 router-id=1.1.1.1 \
    disabled=no comment="PE router BGP"

/routing bgp connection add template=mpls-pe remote.address=2.2.2.2 \
    remote.as=65001 address-families=vpnv4 disabled=no \
    comment="iBGP to remote PE"

# Route targets 65001:999 allows access to shared services
# Import routes from shared services VPN for customer access

# Verify MPLS VPN operation
/ip route vrf print detail
/routing bgp advertisements print where peer=2.2.2.2
/ip route print where routing-table=mpls-customer
```
{% endcode %}

***

## VRF monitoring and management

### VRF status monitoring

Track VRF performance and connectivity:

{% code overflow="wrap" %}
```bash
# Monitor VRF instances and their interfaces
/ip route vrf print detail

# Check routes per VRF
/ip route print where routing-table=customer-a
/ip route print where routing-table=customer-b

# Monitor VRF interface status
/interface print where master-port="" and disabled=no

# Test connectivity within specific VRF
/ping 192.168.10.100 routing-table=customer-a count=5
/tool traceroute 10.10.1.1 routing-table=customer-a

# Monitor routing protocol status per VRF
/routing ospf neighbor print where instance=customer-ospf
/routing bgp session print where vrf=customer-bgp

# VRF traffic statistics
/interface monitor-traffic ether2,ether3 duration=30
```
{% endcode %}

### Troubleshooting VRF issues

Common VRF problems and diagnostic steps:

{% code overflow="wrap" %}
```bash
# 1. Verify VRF configuration
/ip route vrf print detail

# 2. Check interface assignments
/interface print where master-port=""  # Should show VRF assignments

# 3. Verify routing table contents
/ip route print where routing-table=customer-a

# 4. Test VRF connectivity
/ping 192.168.10.1 routing-table=customer-a

# 5. Check for route leaking issues
/ip route print where routing-table=customer-a and dst-address~"172.16"

# 6. Verify MPLS VPN functionality (if applicable)
/routing bgp advertisements print where peer=2.2.2.2
/ip route print where routing-table=mpls-customer and bgp=yes

# 7. Monitor for VRF-related errors
/log print where topics~"routing,bgp,ospf"

# VRF diagnostic script
:local vrfName "customer-a";
:local testDestination "192.168.10.100";

/log info ("Diagnosing VRF: " . $vrfName);

# Check VRF exists
:local vrfExists [/ip route vrf find routing-mark=$vrfName];
:if ([:len $vrfExists] > 0) do={
    /log info ("VRF " . $vrfName . " is configured");
    
    # Check routes in VRF
    :local routeCount [/ip route print count-only where routing-table=$vrfName];
    /log info ("VRF " . $vrfName . " has " . $routeCount . " routes");
    
    # Test connectivity
    :do {
        /ping $testDestination routing-table=$vrfName count=3;
        /log info ("Connectivity test to " . $testDestination . " successful");
    } on-error={
        /log error ("Connectivity test to " . $testDestination . " failed");
    };
} else={
    /log error ("VRF " . $vrfName . " not found");
};
```
{% endcode %}

***

## VRF security and isolation

### Traffic isolation verification

Ensure proper VRF isolation:

{% code overflow="wrap" %}
```bash
# Test VRF isolation (should fail)
# Try to access Customer B from Customer A VRF
/ping 192.168.20.100 routing-table=customer-a  # Should fail

# Test shared services access (should work if configured)
/ping 172.16.1.100 routing-table=customer-a  # Should work if route leaking configured

# Verify routing table separation
/ip route print where routing-table=customer-a
/ip route print where routing-table=customer-b
# Routes should be completely separate

# Check for accidental route leaking
/ip route print where routing-table=customer-a and dst-address~"192.168.20"
# Should return no results (Customer B networks shouldn't be in Customer A VRF)
```
{% endcode %}

### VRF security policies

Implement security controls for VRF environments:

{% code overflow="wrap" %}
```bash
# Firewall rules for VRF security
# Block inter-VRF communication at firewall level
/ip firewall filter add chain=forward \
    in-interface-list=customer-a-interfaces \
    out-interface-list=customer-b-interfaces \
    action=drop comment="Block Customer A to Customer B"

/ip firewall filter add chain=forward \
    in-interface-list=customer-b-interfaces \
    out-interface-list=customer-a-interfaces \
    action=drop comment="Block Customer B to Customer A"

# Create interface lists for VRF management
/interface list add name=customer-a-interfaces comment="Customer A VRF interfaces"
/interface list add name=customer-b-interfaces comment="Customer B VRF interfaces"

# Add interfaces to appropriate lists
/interface list member add list=customer-a-interfaces interface=ether2
/interface list member add list=customer-b-interfaces interface=ether3

# Allow controlled access to shared services
/ip firewall filter add chain=forward \
    in-interface-list=customer-a-interfaces \
    dst-address=172.16.0.0/16 protocol=tcp dst-port=53,123 \
    action=accept comment="Allow Customer A to shared services"

# Log VRF violations for security monitoring
/ip firewall filter add chain=forward \
    in-interface-list=customer-a-interfaces \
    out-interface-list=customer-b-interfaces \
    action=log log-prefix="VRF-VIOLATION-A-to-B"

# NAT policies per VRF (if needed for internet access)
/ip firewall nat add chain=srcnat \
    out-interface=wan-interface \
    src-address=192.168.10.0/24 \
    action=masquerade comment="Customer A NAT"

/ip firewall nat add chain=srcnat \
    out-interface=wan-interface \
    src-address=192.168.20.0/24 \
    action=masquerade comment="Customer B NAT"
```
{% endcode %}

***

## VRF best practices

### Design considerations

1. **Plan VRF hierarchy** - Design logical VRF structure for scalability
2. **Interface assignment** - Carefully plan interface-to-VRF mappings
3. **Route target strategy** - Plan RT scheme for MPLS VPN environments
4. **Shared services design** - Plan controlled access to common resources
5. **Security policies** - Implement proper isolation and access controls

### Performance optimization

1. **Hardware capabilities** - Verify VRF support in hardware
2. **Routing protocol tuning** - Optimize protocols per VRF requirements
3. **Memory planning** - Account for multiple routing tables
4. **Interface optimization** - Tune VRF interfaces for performance
5. **Monitoring overhead** - Plan for increased monitoring complexity

### Operational guidelines

1. **Documentation** - Maintain clear VRF documentation and diagrams
2. **Change management** - Implement careful VRF change procedures
3. **Testing procedures** - Develop VRF-specific testing protocols
4. **Backup strategies** - Include VRF configuration in backups
5. **Training** - Ensure staff understands VRF concepts and operations

***

## Complete VRF examples

### Service provider multi-tenant setup

{% code overflow="wrap" %}
```bash
# Complete service provider VRF configuration
# Supporting multiple enterprise customers with different requirements

# Customer 1: Finance company with strict isolation
/ip route vrf add interfaces=ether2 routing-mark=finance-corp \
    route-distinguisher=65001:100 \
    import-route-targets=65001:100,65001:999 \
    export-route-targets=65001:100 \
    comment="Finance Corp - High security"

/ip address add address=10.100.1.1/30 interface=ether2 comment="Finance CE link"

# Static routing for Finance customer
/ip route add dst-address=172.16.0.0/12 gateway=10.100.1.2 \
    routing-table=finance-corp comment="Finance internal networks"

# Customer 2: Manufacturing with OSPF
/ip route vrf add interfaces=ether3,ether4 routing-mark=manufacturing-inc \
    route-distinguisher=65001:200 \
    import-route-targets=65001:200,65001:999 \
    export-route-targets=65001:200 \
    comment="Manufacturing Inc - Multi-site OSPF"

/ip address add address=10.200.1.1/30 interface=ether3 comment="Manufacturing HQ"
/ip address add address=10.200.2.1/30 interface=ether4 comment="Manufacturing Plant"

# OSPF for Manufacturing customer
/routing ospf instance add name=manufacturing-ospf router-id=10.1.1.1 \
    vrf=manufacturing-inc disabled=no

/routing ospf area add name=manufacturing-area area-id=0.0.0.0 \
    instance=manufacturing-ospf

/routing ospf interface-template add area=manufacturing-area \
    interfaces=ether3,ether4 type=ptp disabled=no

# Customer 3: Retail chain with BGP
/ip route vrf add interfaces=ether5 routing-mark=retail-chain \
    route-distinguisher=65001:300 \
    import-route-targets=65001:300,65001:999 \
    export-route-targets=65001:300 \
    comment="Retail Chain - BGP connectivity"

/ip address add address=10.300.1.1/30 interface=ether5 comment="Retail HQ"

# BGP for Retail customer
/routing bgp template add name=retail-bgp as=65001 router-id=10.1.1.1 \
    vrf=retail-chain disabled=no

/routing bgp connection add template=retail-bgp \
    remote.address=10.300.1.2 remote.as=65300 disabled=no \
    comment="Retail customer BGP"

# Shared services VRF (DNS, NTP, monitoring)
/ip route vrf add interfaces=ether6 routing-mark=shared-services \
    route-distinguisher=65001:999 \
    import-route-targets=65001:999 \
    export-route-targets=65001:999 \
    comment="Shared services for all customers"

/ip address add address=172.31.1.1/24 interface=ether6 comment="Shared services"

# Shared services routes
/ip route add dst-address=172.31.10.0/24 gateway=172.31.1.10 \
    routing-table=shared-services comment="DNS servers"

/ip route add dst-address=172.31.20.0/24 gateway=172.31.1.20 \
    routing-table=shared-services comment="NTP servers"

# MPLS configuration for inter-site connectivity
/mpls interface add interface=ether1 disabled=no comment="MPLS core"
/mpls ldp set enabled=yes lsr-id=1.1.1.1
/mpls ldp interface add interface=ether1 disabled=no

# BGP for VPNv4 route distribution
/routing bgp template add name=pe-bgp as=65001 router-id=1.1.1.1 disabled=no

/routing bgp connection add template=pe-bgp remote.address=2.2.2.2 \
    remote.as=65001 address-families=vpnv4 disabled=no comment="PE-to-PE iBGP"

# QoS per customer VRF
/queue tree add name=finance-queue parent=ether2 max-limit=100M \
    guaranteed-rate=50M comment="Finance guaranteed bandwidth"

/queue tree add name=manufacturing-queue parent=ether3 max-limit=200M \
    comment="Manufacturing high bandwidth"

/queue tree add name=retail-queue parent=ether5 max-limit=50M \
    comment="Retail standard service"

# Security policies
/ip firewall filter add chain=forward \
    in-interface=ether2 out-interface=ether3,ether4,ether5 \
    action=drop comment="Block Finance to other customers"

/ip firewall filter add chain=forward \
    in-interface=ether3,ether4 out-interface=ether2,ether5 \
    action=drop comment="Block Manufacturing to other customers"

/ip firewall filter add chain=forward \
    in-interface=ether5 out-interface=ether2,ether3,ether4 \
    action=drop comment="Block Retail to other customers"

# Allow all customers to access shared services
/ip firewall filter add chain=forward \
    in-interface=ether2,ether3,ether4,ether5 out-interface=ether6 \
    dst-address=172.31.0.0/16 action=accept comment="Allow shared services access"

# Monitoring and health checks
/tool netwatch add host=10.100.1.2 timeout=2s interval=10s \
    comment="Finance customer health"

/tool netwatch add host=10.200.1.2 timeout=2s interval=10s \
    comment="Manufacturing HQ health"

/tool netwatch add host=10.300.1.2 timeout=2s interval=10s \
    comment="Retail customer health"

# Logging for troubleshooting
/system logging add topics=bgp,ospf,mpls action=memory

# Verification commands
/ip route vrf print
/ip route print where routing-table=finance-corp
/ip route print where routing-table=manufacturing-inc  
/ip route print where routing-table=retail-chain
/routing ospf neighbor print where instance=manufacturing-ospf
/routing bgp session print
/mpls ldp neighbor print
```
{% endcode %}

