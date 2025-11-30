---
description: This section covers layer-7 protocol filtering and deep packet inspection for application-aware firewall rules and traffic management.
icon: search
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

# Layer-7

{% hint style="info" %}
Layer-7 filtering enables RouterOS to inspect packet contents and identify applications, protocols, and specific traffic patterns for advanced filtering, QoS, and bandwidth management beyond simple port-based rules.
{% endhint %}

In WinBox you can configure layer-7 protocols in **IP -> Firewall -> Layer7 Protocols**, then use them in filter, mangle, or queue rules.

Layer-7 inspection allows identification of applications that use dynamic ports, encrypted traffic, or protocols that tunnel through standard ports.

***

## Layer-7 fundamentals

### How layer-7 detection works

**Detection methods:**
- **Regular expressions** - Pattern matching against packet payload
- **Connection state tracking** - First few packets of connections analyzed
- **Signature matching** - Known application signatures and behaviors
- **Port-independent** - Identifies protocols regardless of port used

**Processing characteristics:**
- **CPU intensive** - Requires packet content inspection
- **Connection-based** - Only first packets analyzed, then connection marked
- **Memory usage** - Stores connection states and patterns
- **Performance impact** - More complex patterns affect throughput

**Common use cases:**
- **Application blocking** - Block P2P, social media, streaming
- **Bandwidth management** - Limit specific application bandwidth
- **QoS policies** - Prioritize business vs recreational traffic
- **Content filtering** - Block unwanted content categories

***

## Creating layer-7 protocols

### Basic protocol definitions

Create layer-7 protocol patterns for common applications:

{% code overflow="wrap" %}
```bash
# HTTP protocols
/ip firewall layer7-protocol add name=http-web regexp="^(GET|POST|HEAD|PUT|DELETE|OPTIONS|TRACE|CONNECT) .*HTTP/1\\.[01]"

# Social media platforms
/ip firewall layer7-protocol add name=facebook regexp="^.*(facebook|fbcdn|fbsbx).*$"
/ip firewall layer7-protocol add name=twitter regexp="^.*(twitter|twimg).*$"
/ip firewall layer7-protocol add name=instagram regexp="^.*(instagram|cdninstagram).*$"
/ip firewall layer7-protocol add name=tiktok regexp="^.*(tiktok|musical\\.ly).*$"

# Video streaming services
/ip firewall layer7-protocol add name=youtube regexp="^.*(youtube|googlevideo|ytimg).*$"
/ip firewall layer7-protocol add name=netflix regexp="^.*(netflix|nflxvideo).*$"
/ip firewall layer7-protocol add name=twitch regexp="^.*(twitch|twitchcdn).*$"

# File sharing / P2P
/ip firewall layer7-protocol add name=bittorrent regexp="^(\\x13BitTorrent protocol|\\x42\\x54:|\\x47\\x45\\x54\\x2f)"
/ip firewall layer7-protocol add name=edonkey regexp="^[\\xe3\\xc6\\xd4\\xc7\\xe5]"
```
{% endcode %}

### Messaging and communication

{% code overflow="wrap" %}
```bash
# Instant messaging
/ip firewall layer7-protocol add name=telegram regexp="^.*(telegram|t\\.me).*$"
/ip firewall layer7-protocol add name=whatsapp regexp="^.*(whatsapp|wa\\.me).*$" 
/ip firewall layer7-protocol add name=discord regexp="^.*(discord|discordapp).*$"
/ip firewall layer7-protocol add name=skype regexp="^.*(skype|live\\.com).*$"

# VoIP protocols
/ip firewall layer7-protocol add name=sip-voip regexp="^(INVITE|REGISTER|OPTIONS|ACK|BYE) sip:"
/ip firewall layer7-protocol add name=teamspeak regexp="^\\xf4\\xbe\\x03.*$"

# Remote access
/ip firewall layer7-protocol add name=teamviewer regexp="^.*(teamviewer|dyngate).*$"
/ip firewall layer7-protocol add name=anydesk regexp="^.*anydesk.*$"
```
{% endcode %}

### Gaming and entertainment  

{% code overflow="wrap" %}
```bash
# Gaming platforms
/ip firewall layer7-protocol add name=steam regexp="^.*(steam|steamcontent|steamstatic).*$"
/ip firewall layer7-protocol add name=origin regexp="^.*(origin|ea\\.com).*$"
/ip firewall layer7-protocol add name=battlenet regexp="^.*(battle\\.net|blizzard).*$"

# Cloud gaming
/ip firewall layer7-protocol add name=geforce-now regexp="^.*(nvidia|geforce).*$"
/ip firewall layer7-protocol add name=google-stadia regexp="^.*stadia.*$"

# Music streaming
/ip firewall layer7-protocol add name=spotify regexp="^.*(spotify|scdn\\.co).*$"
/ip firewall layer7-protocol add name=apple-music regexp="^.*(apple|itunes).*$"
```
{% endcode %}

***

## Application blocking with layer-7

### Block social media platforms

Prevent access to social media during work hours:

{% code overflow="wrap" %}
```bash
# Block Facebook during business hours
/ip firewall filter add chain=forward action=drop layer7-protocol=facebook \
    time=8h-17h,mon,tue,wed,thu,fri comment="Block Facebook during work hours"

# Block multiple social platforms  
/ip firewall filter add chain=forward action=drop \
    layer7-protocol=facebook,twitter,instagram,tiktok \
    src-address-list=EMPLOYEES comment="Block social media for employees"

# Allow social media for management
/ip firewall filter add chain=forward action=accept \
    layer7-protocol=facebook,twitter,instagram \
    src-address-list=MANAGEMENT comment="Allow social media for management"
```
{% endcode %}

### Block P2P and file sharing

Prevent bandwidth abuse from P2P applications:

{% code overflow="wrap" %}
```bash
# Block BitTorrent completely
/ip firewall filter add chain=forward action=drop layer7-protocol=bittorrent \
    comment="Block BitTorrent traffic"

# Block during business hours only
/ip firewall filter add chain=forward action=drop \
    layer7-protocol=bittorrent,edonkey \
    time=8h-18h,mon,tue,wed,thu,fri comment="Block P2P during business hours"

# Limit P2P to specific users
/ip firewall filter add chain=forward action=accept layer7-protocol=bittorrent \
    src-address-list=P2P-ALLOWED comment="Allow P2P for specific users"

/ip firewall filter add chain=forward action=drop layer7-protocol=bittorrent \
    comment="Block P2P for all others"
```
{% endcode %}

### Block streaming during peak hours

Control bandwidth usage during busy periods:

{% code overflow="wrap" %}
```bash
# Block video streaming during peak hours
/ip firewall filter add chain=forward action=drop \
    layer7-protocol=youtube,netflix,twitch \
    time=9h-12h,14h-17h,mon,tue,wed,thu,fri \
    comment="Block streaming during peak hours"

# Allow streaming for VIP users
/ip firewall filter add chain=forward action=accept \
    layer7-protocol=youtube,netflix \
    src-address-list=VIP-USERS comment="Allow streaming for VIP users"
```
{% endcode %}

***

## Traffic shaping with layer-7

### QoS based on applications

Prioritize or limit specific application traffic:

{% code overflow="wrap" %}
```bash
# Mark VoIP traffic for high priority
/ip firewall mangle add chain=prerouting action=mark-connection \
    layer7-protocol=sip-voip new-connection-mark=voip-conn passthrough=yes \
    comment="Mark VoIP connections"

/ip firewall mangle add chain=prerouting action=mark-packet \
    connection-mark=voip-conn new-packet-mark=voip-priority passthrough=no \
    comment="Mark VoIP packets for priority"

# Mark streaming for bandwidth limiting
/ip firewall mangle add chain=prerouting action=mark-connection \
    layer7-protocol=youtube,netflix new-connection-mark=streaming-conn \
    passthrough=yes comment="Mark streaming connections"

/ip firewall mangle add chain=prerouting action=mark-packet \
    connection-mark=streaming-conn new-packet-mark=streaming-traffic \
    passthrough=no comment="Mark streaming traffic for limiting"

# Mark P2P for lowest priority  
/ip firewall mangle add chain=prerouting action=mark-connection \
    layer7-protocol=bittorrent new-connection-mark=p2p-conn passthrough=yes \
    comment="Mark P2P connections"

/ip firewall mangle add chain=prerouting action=mark-packet \
    connection-mark=p2p-conn new-packet-mark=p2p-traffic passthrough=no \
    comment="Mark P2P traffic for lowest priority"
```
{% endcode %}

### Bandwidth allocation per application

{% code overflow="wrap" %}
```bash
# Create queue tree for application-based bandwidth management
# (This requires the mangle marks created above)

# High priority for VoIP (guarantee 1Mbps, max 2Mbps)
/queue tree add name=voip-queue parent=global packet-mark=voip-priority \
    limit-at=1M max-limit=2M priority=1 comment="VoIP guaranteed bandwidth"

# Medium priority for business apps (guarantee 5Mbps, max 15Mbps)
/queue tree add name=business-queue parent=global packet-mark=business-traffic \
    limit-at=5M max-limit=15M priority=3 comment="Business applications"

# Limited bandwidth for streaming (max 10Mbps per connection)
/queue tree add name=streaming-queue parent=global packet-mark=streaming-traffic \
    max-limit=10M priority=6 comment="Streaming bandwidth limit"

# Lowest priority for P2P (use remaining bandwidth only)
/queue tree add name=p2p-queue parent=global packet-mark=p2p-traffic \
    max-limit=50M priority=8 comment="P2P lowest priority"
```
{% endcode %}

***

## Advanced layer-7 techniques

### Custom protocol detection

Create custom patterns for internal applications:

{% code overflow="wrap" %}
```bash
# Detect custom application by HTTP header
/ip firewall layer7-protocol add name=custom-app \
    regexp="^.*User-Agent: CustomApp/.*$"

# Detect by specific URL patterns  
/ip firewall layer7-protocol add name=internal-api \
    regexp="^GET /api/v[0-9]+/.*$"

# Detect by custom protocol signature
/ip firewall layer7-protocol add name=proprietary-protocol \
    regexp="^\\x4d\\x59\\x41\\x50\\x50.*$"

# Use these patterns in firewall rules
/ip firewall filter add chain=forward action=accept layer7-protocol=custom-app \
    comment="Allow custom application"
```
{% endcode %}

### Multi-pattern protocol detection

Combine multiple patterns for accurate detection:

{% code overflow="wrap" %}
```bash
# More specific YouTube detection  
/ip firewall layer7-protocol add name=youtube-video \
    regexp="^.*(youtube\\.com/watch|googlevideo\\.com).*$"

# Specific Netflix detection
/ip firewall layer7-protocol add name=netflix-streaming \
    regexp="^.*(netflix\\.com|nflxvideo\\.net).*$"

# Gaming-specific protocols
/ip firewall layer7-protocol add name=fortnite \
    regexp="^.*epicgames.*$"

/ip firewall layer7-protocol add name=pubg \
    regexp="^.*(pubg|krafton).*$"
```
{% endcode %}

### Layer-7 with time-based policies

{% code overflow="wrap" %}
```bash
# Allow gaming only after hours
/ip firewall filter add chain=forward action=accept \
    layer7-protocol=steam,origin,battlenet \
    time=18h-8h comment="Allow gaming after work hours"

/ip firewall filter add chain=forward action=drop \
    layer7-protocol=steam,origin,battlenet \
    time=8h-18h,mon,tue,wed,thu,fri comment="Block gaming during work"

# Limit streaming during peak internet usage
/ip firewall filter add chain=forward action=drop \
    layer7-protocol=youtube-video,netflix-streaming \
    time=19h-23h comment="Block streaming during peak hours"
```
{% endcode %}

***

## Layer-7 with address lists

### Dynamic application-based blocking

Combine layer-7 with dynamic address lists:

{% code overflow="wrap" %}
```bash
# Add users to restriction list when using P2P
/ip firewall filter add chain=forward action=add-src-to-address-list \
    layer7-protocol=bittorrent address-list=p2p-users address-list-timeout=1h \
    comment="Track P2P users"

# Limit internet access for tracked P2P users
/ip firewall filter add chain=forward action=drop \
    src-address-list=p2p-users dst-port=80,443 protocol=tcp \
    comment="Limit web access for P2P users"

# Escalating restrictions for repeated violations
/ip firewall filter add chain=forward action=add-src-to-address-list \
    layer7-protocol=facebook src-address-list=social-offenders \
    address-list=repeat-offenders address-list-timeout=24h \
    comment="Track repeat social media users"
```
{% endcode %}

### User-based application policies

{% code overflow="wrap" %}
```bash
# Create user groups with different application access
/ip firewall address-list add list=EXECUTIVES address=192.168.1.10-192.168.1.15
/ip firewall address-list add list=EMPLOYEES address=192.168.1.100-192.168.1.200
/ip firewall address-list add list=GUESTS address=192.168.10.0/24

# Executives - full access
/ip firewall filter add chain=forward action=accept \
    src-address-list=EXECUTIVES comment="Allow all applications for executives"

# Employees - restricted social media
/ip firewall filter add chain=forward action=drop \
    src-address-list=EMPLOYEES layer7-protocol=facebook,twitter,instagram \
    time=8h-17h,mon,tue,wed,thu,fri comment="Block social media for employees"

# Guests - very limited access  
/ip firewall filter add chain=forward action=accept \
    src-address-list=GUESTS protocol=tcp dst-port=80,443 \
    comment="Allow only web browsing for guests"

/ip firewall filter add chain=forward action=drop \
    src-address-list=GUESTS layer7-protocol=bittorrent,steam,youtube \
    comment="Block entertainment apps for guests"
```
{% endcode %}

***

## Performance optimization

### Optimize layer-7 processing

{% code overflow="wrap" %}
```bash
# Use connection marking to avoid re-processing
/ip firewall mangle add chain=prerouting action=mark-connection \
    layer7-protocol=youtube new-connection-mark=youtube-conn passthrough=yes \
    comment="Mark YouTube connections once"

# Use connection marks in subsequent rules instead of layer-7
/ip firewall filter add chain=forward action=drop \
    connection-mark=youtube-conn time=8h-17h,mon,tue,wed,thu,fri \
    comment="Block marked YouTube connections"

# Minimize regex complexity
# Good: simple string match
/ip firewall layer7-protocol add name=simple-match regexp="^.*youtube.*$"

# Avoid: complex nested patterns
# /ip firewall layer7-protocol add name=complex-match regexp="^.*(youtube|googlevideo).*((watch|embed).*)?$"
```
{% endcode %}

### Connection state management

{% code overflow="wrap" %}
```bash
# Only apply layer-7 to new connections
/ip firewall filter add chain=forward action=accept \
    connection-state=established,related comment="Accept established connections"

/ip firewall filter add chain=forward action=drop \
    connection-state=new layer7-protocol=bittorrent \
    comment="Block new P2P connections only"

# Use connection tracking efficiently
/ip firewall connection tracking set enabled=yes
/ip settings set tcp-syncookies=yes
```
{% endcode %}

***

## Monitoring layer-7 traffic

### Monitor detected protocols

{% code overflow="wrap" %}
```bash
# Monitor layer-7 protocol detection
/ip firewall filter print stats where layer7-protocol~"youtube"

# Check active connections by protocol
/ip firewall connection print where layer7-protocol=bittorrent

# Monitor most used protocols
/log print where topics~"firewall" and message~"layer7"

# Real-time protocol monitoring
/tool torch interface=bridge duration=30
```
{% endcode %}

### Debug layer-7 detection

{% code overflow="wrap" %}
```bash
# Enable logging for layer-7 matches
/ip firewall filter add chain=forward action=log log-prefix="L7-MATCH: " \
    layer7-protocol=youtube place-before=0

# Test protocol patterns
/tool fetch url="http://www.youtube.com" mode=http

# Check protocol detection accuracy
/ip firewall connection print detail where layer7-protocol!=no-protocol

# Monitor detection performance
/system resource print
```
{% endcode %}

***

## Troubleshooting layer-7

### Common issues and solutions

{% code overflow="wrap" %}
```bash
# Issue: Protocol not detected
# Solution 1: Check regex pattern
/ip firewall layer7-protocol print where name=youtube

# Solution 2: Test with packet capture
/tool sniffer start interface=bridge filter-stream=yes file-name=l7-debug.pcap

# Solution 3: Verify connection state
/ip firewall connection print where dst-address~"youtube"

# Issue: High CPU usage
# Solution 1: Simplify regex patterns
/ip firewall layer7-protocol set youtube regexp="^.*youtube.*$"

# Solution 2: Use connection marking
/ip firewall mangle add chain=prerouting action=mark-connection \
    layer7-protocol=youtube new-connection-mark=yt-conn passthrough=yes

# Solution 3: Limit layer-7 processing
/ip firewall filter add chain=forward action=accept \
    connection-state=established,related comment="Bypass L7 for established"
```
{% endcode %}

### Performance troubleshooting

{% code overflow="wrap" %}
```bash
# Check layer-7 processing impact
/system resource monitor numbers=0 duration=10

# Monitor connection table usage
/ip firewall connection tracking print

# Check memory usage by layer-7
/system resource print

# Disable layer-7 temporarily for testing
/ip firewall layer7-protocol disable [find]
```
{% endcode %}

***

<details>

<summary>Show complete layer-7 content filtering setup</summary>

{% code overflow="wrap" %}
```bash
# Complete layer-7 content filtering for office environment
# 1. Define layer-7 protocols
/ip firewall layer7-protocol add name=facebook regexp="^.*(facebook|fbcdn).*$"
/ip firewall layer7-protocol add name=youtube regexp="^.*(youtube|googlevideo).*$"
/ip firewall layer7-protocol add name=netflix regexp="^.*(netflix|nflxvideo).*$"
/ip firewall layer7-protocol add name=bittorrent regexp="^(\\x13BitTorrent protocol|\\x42\\x54:)"
/ip firewall layer7-protocol add name=steam regexp="^.*(steam|steamcontent).*$"
/ip firewall layer7-protocol add name=teamviewer regexp="^.*(teamviewer|dyngate).*$"

# 2. Create user groups
/ip firewall address-list add list=MANAGEMENT address=192.168.1.10-192.168.1.20
/ip firewall address-list add list=EMPLOYEES address=192.168.1.100-192.168.1.200
/ip firewall address-list add list=CONTRACTORS address=192.168.2.0/24

# 3. Mark connections for each protocol
/ip firewall mangle add chain=prerouting action=mark-connection layer7-protocol=facebook new-connection-mark=facebook-conn passthrough=yes
/ip firewall mangle add chain=prerouting action=mark-connection layer7-protocol=youtube new-connection-mark=youtube-conn passthrough=yes
/ip firewall mangle add chain=prerouting action=mark-connection layer7-protocol=netflix new-connection-mark=netflix-conn passthrough=yes
/ip firewall mangle add chain=prerouting action=mark-connection layer7-protocol=bittorrent new-connection-mark=p2p-conn passthrough=yes
/ip firewall mangle add chain=prerouting action=mark-connection layer7-protocol=steam new-connection-mark=gaming-conn passthrough=yes

# 4. Management access - allow everything
/ip firewall filter add chain=forward action=accept src-address-list=MANAGEMENT comment="Full access for management"

# 5. Employee restrictions during work hours
/ip firewall filter add chain=forward action=drop src-address-list=EMPLOYEES connection-mark=facebook-conn time=8h-17h,mon,tue,wed,thu,fri comment="Block Facebook for employees"
/ip firewall filter add chain=forward action=drop src-address-list=EMPLOYEES connection-mark=youtube-conn time=8h-17h,mon,tue,wed,thu,fri comment="Block YouTube for employees"
/ip firewall filter add chain=forward action=drop src-address-list=EMPLOYEES connection-mark=gaming-conn time=8h-17h,mon,tue,wed,thu,fri comment="Block gaming for employees"

# 6. Contractor restrictions - more limited access
/ip firewall filter add chain=forward action=drop src-address-list=CONTRACTORS connection-mark=facebook-conn comment="Block social media for contractors"
/ip firewall filter add chain=forward action=drop src-address-list=CONTRACTORS connection-mark=netflix-conn comment="Block streaming for contractors"
/ip firewall filter add chain=forward action=drop src-address-list=CONTRACTORS connection-mark=p2p-conn comment="Block P2P for contractors"
/ip firewall filter add chain=forward action=drop src-address-list=CONTRACTORS connection-mark=gaming-conn comment="Block gaming for contractors"

# 7. Global P2P blocking
/ip firewall filter add chain=forward action=drop connection-mark=p2p-conn comment="Block P2P globally"

# 8. Bandwidth limiting for streaming (when allowed)
/ip firewall mangle add chain=prerouting action=mark-packet connection-mark=youtube-conn new-packet-mark=youtube-traffic passthrough=no
/ip firewall mangle add chain=prerouting action=mark-packet connection-mark=netflix-conn new-packet-mark=netflix-traffic passthrough=no
/queue tree add name=streaming-limit parent=global packet-mark=youtube-traffic,netflix-traffic max-limit=5M comment="Limit streaming bandwidth"

# 9. Monitoring and logging
/ip firewall filter add chain=forward action=log log-prefix="BLOCKED-APP: " connection-mark=facebook-conn,youtube-conn src-address-list=EMPLOYEES time=8h-17h,mon,tue,wed,thu,fri
/system logging add topics=firewall,info action=memory
```
{% endcode %}

</details>

## Layer-7 best practices

### Design recommendations

1. **Start simple** - Use basic patterns before complex regex
2. **Test thoroughly** - Verify detection accuracy with real traffic
3. **Monitor performance** - Watch CPU impact of layer-7 rules
4. **Use connection marking** - Mark once, use many times
5. **Plan for scale** - Consider impact with many concurrent connections

### Security considerations

1. **Encrypted traffic** - Layer-7 cannot inspect encrypted payloads
2. **Evasion techniques** - Applications may bypass detection
3. **Privacy concerns** - Deep packet inspection has privacy implications
4. **False positives** - Overly broad patterns may block legitimate traffic
5. **Regular updates** - Application patterns change over time

### Performance guidelines

1. **Limit scope** - Apply layer-7 only where needed
2. **Optimize patterns** - Use efficient regex patterns
3. **Connection limits** - Monitor connection table usage
4. **CPU monitoring** - Watch for performance degradation
5. **Memory management** - Layer-7 increases memory usage

