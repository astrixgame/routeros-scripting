---
description: >-
  An older VPN protocol that is considered insecure. It works in a Client-Server
  model but should never be used for secure communications.
icon: triangle-exclamation
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

# PPTP

{% hint style="danger" %}
PPTP is fundamentally insecure and should NEVER be used for any real security needs. This documentation is provided for educational purposes and legacy system understanding only.
{% endhint %}

In WinBox you can configure PPTP in **PPP -> PPTP Server**, or you can use terminal with command `/interface pptp-server`

**Why PPTP should be avoided:**
- Uses weak MS-CHAP v2 authentication (crackable in hours)
- Relies on outdated MPPE encryption (RC4-based)
- Multiple known security vulnerabilities
- No perfect forward secrecy
- Easily blocked by firewalls (uses GRE protocol)

***

## Historical context

PPTP (Point-to-Point Tunneling Protocol) was developed by Microsoft in 1996 as one of the first widely-available VPN protocols. While it was convenient and fast, security researchers quickly identified fundamental flaws that cannot be fixed without completely redesigning the protocol.

**Timeline of PPTP vulnerabilities:**
- **1998** - First major flaws discovered in MS-CHAP authentication
- **2012** - Complete MS-CHAP v2 authentication bypass demonstrated
- **2012** - MPPE encryption proven easily breakable
- **Present** - Considered completely compromised

***

## Legacy PPTP server configuration

{% hint style="warning" %}
This configuration is shown for educational purposes only. Do NOT use in production environments.
{% endhint %}

### Basic server setup

In WinBox go to **PPP -> PPTP Server**:

* **Enabled** - Yes
* **Max MTU** - 1460 (to avoid fragmentation)
* **Max MRU** - 1460
* **Authentication** - mschap2 (still weak!)
* **Default Profile** - Select appropriate profile

{% code overflow="wrap" %}
```bash
# Enable PPTP server (NOT RECOMMENDED)
/interface pptp-server server set enabled=yes max-mtu=1460 max-mru=1460 authentication=mschap2 default-profile=default-encryption
```
{% endcode %}

### Create IP pool and profile

{% code overflow="wrap" %}
```bash
# Create IP pool for PPTP clients
/ip pool add name=pptp-pool ranges=192.168.200.2-192.168.200.50

# Create PPTP profile
/ppp profile add name=pptp-profile local-address=192.168.200.1 remote-address=pptp-pool dns-server=8.8.8.8,1.1.1.1 use-encryption=yes
```
{% endcode %}

### Add users

{% code overflow="wrap" %}
```bash
# Add PPTP users (credentials will be easily compromised!)
/ppp secret add name=testuser password=testpass service=pptp profile=pptp-profile comment="INSECURE - MIGRATE IMMEDIATELY"
```
{% endcode %}

### Firewall rules

PPTP requires specific firewall rules:

{% code overflow="wrap" %}
```bash
# Allow PPTP control connection
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=1723 comment="PPTP Control (INSECURE)"

# Allow GRE protocol for PPTP tunneling
/ip firewall filter add chain=input action=accept protocol=gre comment="PPTP GRE (INSECURE)"

# Forward PPTP client traffic
/ip firewall filter add chain=forward action=accept in-interface=pptp-in comment="PPTP Forward (INSECURE)"
```
{% endcode %}

***

## Security vulnerabilities

### Authentication weaknesses

**MS-CHAP v2 flaws:**
- Password hash can be extracted from network traffic
- Hash can be cracked offline using rainbow tables
- No protection against replay attacks
- Vulnerable to man-in-the-middle attacks

### Encryption problems

**MPPE encryption issues:**
- Based on RC4 stream cipher (cryptographically broken)
- Uses weak 40-bit or 128-bit keys
- Key derivation from password hash is predictable
- No authentication of encrypted data (malleable)

### Protocol design flaws

**Fundamental issues:**
- Uses GRE protocol (problematic with NAT/firewalls)
- No certificate-based authentication
- Single TCP connection for control (easy to DOS)
- No perfect forward secrecy

***

## Migration strategies

### Immediate replacement options

**Migrate to WireGuard (RECOMMENDED):**
{% code overflow="wrap" %}
```bash
# Disable PPTP immediately
/interface pptp-server server set enabled=no

# Enable WireGuard instead
/interface wireguard add name=wg-secure listen-port=13231
/ip address add address=10.13.13.1/24 interface=wg-secure
```
{% endcode %}

**Migrate to OpenVPN (Alternative):**
{% code overflow="wrap" %}
```bash
# Disable PPTP
/interface pptp-server server set enabled=no

# Configure OpenVPN with proper security
/interface ovpn-server server add name=ovpn-secure port=35324 certificate=SERVER auth=sha256 cipher=aes256 tls-version=only-1.2
```
{% endcode %}

### Migration planning

**Phase 1 - Assessment:**
1. Identify all PPTP users and use cases
2. Document current network access requirements
3. Plan new VPN infrastructure (WireGuard/OpenVPN)
4. Prepare user migration documentation

**Phase 2 - Parallel deployment:**
1. Deploy secure VPN alongside PPTP
2. Migrate users gradually to new VPN
3. Monitor and ensure new VPN works properly
4. Provide user training and support

**Phase 3 - PPTP removal:**
1. Set firm deadline for PPTP shutdown
2. Notify all users multiple times
3. Disable PPTP server
4. Remove PPTP firewall rules and users

***

## User notification template

Use this template to notify users about PPTP deprecation:

```
Subject: URGENT: PPTP VPN Service Discontinuation - Action Required

Dear [User],

Our PPTP VPN service will be discontinued on [DATE] due to critical security vulnerabilities. 

IMMEDIATE ACTION REQUIRED:
1. Download new VPN client: [LINK]
2. Install new VPN profile: [INSTRUCTIONS]
3. Test new VPN connection before [DATE]
4. Contact IT support if you need assistance: [CONTACT]

WHY THIS CHANGE IS NECESSARY:
- PPTP has fundamental security flaws that cannot be fixed
- Your current VPN traffic can be easily intercepted and decrypted
- Industry security standards require migration to secure protocols

NEW VPN BENEFITS:
- Military-grade encryption (AES-256)
- Better performance and reliability
- Modern security protocols (WireGuard/OpenVPN)
- Improved mobile device support

Please complete this migration by [DEADLINE]. After this date, PPTP access will be permanently disabled.

Thank you for your cooperation in improving our network security.

IT Security Team
```

***

## Compliance considerations

### Regulatory requirements

**Industries that MUST avoid PPTP:**
- Healthcare (HIPAA compliance)
- Finance (PCI DSS, SOX)
- Government (FISMA, FedRAMP)
- Legal (attorney-client privilege)
- Any industry handling personal data (GDPR, CCPA)

### Audit findings

**Common audit flags:**
- Use of deprecated cryptographic protocols
- Insufficient access control mechanisms
- Lack of perfect forward secrecy
- Weak authentication methods
- Unencrypted credential transmission

***

## Emergency PPTP blocking

If you discover PPTP in use and need to block it immediately:

{% code overflow="wrap" %}
```bash
# Emergency PPTP blocking
/ip firewall filter add chain=input action=drop protocol=tcp dst-port=1723 comment="EMERGENCY: Block PPTP"
/ip firewall filter add chain=input action=drop protocol=gre comment="EMERGENCY: Block PPTP GRE"
/ip firewall filter add chain=forward action=drop in-interface=pptp-in comment="EMERGENCY: Block PPTP traffic"

# Disable PPTP server
/interface pptp-server server set enabled=no

# Log PPTP attempts
/ip firewall filter add chain=input action=log protocol=tcp dst-port=1723 log-prefix="PPTP-BLOCKED"
```
{% endcode %}

***

<details>

<summary>Show PPTP vulnerability demonstration (Educational)</summary>

**MS-CHAP v2 hash extraction:**
```bash
# This is how easily PPTP can be compromised:
# 1. Capture network traffic containing MS-CHAP v2 handshake
# 2. Extract challenge/response from packets
# 3. Use tools like hashcat to crack password hash
# 4. Full password recovery typically takes hours, not years

# Example hashcat command for MS-CHAP v2:
# hashcat -m 5600 -a 0 mschapv2.hash wordlist.txt

# This demonstrates why PPTP cannot be secured!
```

</details>

## Security testing

### Penetration testing PPTP

**Common attack vectors:**
1. **Traffic interception** - Capture authentication handshake
2. **Hash cracking** - Offline password attacks against MS-CHAP v2
3. **Man-in-the-middle** - Impersonate PPTP server
4. **GRE manipulation** - Inject malicious traffic into tunnel

**Testing tools:**
- Wireshark (traffic capture)
- Hashcat (password cracking)
- Asleap (MS-CHAP attack tool)
- Chapcrack (MS-CHAP v2 cracker)

### Vulnerability scanning

{% code overflow="wrap" %}
```bash
# Nmap scan for PPTP services
nmap -sS -p 1723 target_ip

# Check for GRE protocol support
nmap -sO -p 47 target_ip

# Vulnerability assessment
nmap --script pptp-version target_ip
```
{% endcode %}

***

## Alternatives comparison

### WireGuard vs PPTP

| Feature | PPTP | WireGuard |
|---------|------|-----------|
| **Security** | ❌ Fundamentally broken | ✅ Modern cryptography |
| **Performance** | ⚠️ Fast but insecure | ✅ Faster and secure |
| **Setup complexity** | ✅ Simple | ✅ Simple |
| **Mobile support** | ⚠️ Basic | ✅ Excellent |
| **NAT traversal** | ❌ Problematic | ✅ Seamless |
| **Compliance** | ❌ Fails audits | ✅ Passes audits |

### OpenVPN vs PPTP

| Feature | PPTP | OpenVPN |
|---------|------|---------|
| **Security** | ❌ Broken | ✅ Secure |
| **Flexibility** | ❌ Limited | ✅ Highly configurable |
| **Client support** | ✅ Built-in OS support | ✅ Wide client availability |
| **Firewall traversal** | ❌ GRE issues | ✅ TCP/UDP options |
| **Certificate support** | ❌ None | ✅ Full PKI support |

***

## Final recommendations

### For administrators

1. **Audit immediately** - Scan for any PPTP usage in your network
2. **Plan migration** - Create timeline for moving to secure VPN
3. **Educate users** - Explain security risks of continuing PPTP use
4. **Block PPTP** - Prevent new PPTP connections at firewall level
5. **Monitor compliance** - Ensure no shadow IT PPTP usage

### For users

1. **Stop using PPTP immediately** - Even for "low-risk" activities
2. **Migrate to WireGuard or OpenVPN** - Contact IT for secure alternatives  
3. **Update devices** - Remove PPTP profiles from all devices
4. **Report PPTP requirements** - If forced to use PPTP, escalate security concerns

{% hint style="info" %}
Remember: There is NO secure way to configure PPTP. Any use of PPTP represents a critical security vulnerability. The protocol should be considered completely compromised.
{% endhint %}
