---
description: This section covers RouterOS update procedures, version management, and package upgrades for maintaining current and secure installations.
icon: arrow-down-to-square
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

# Checking for updates

{% hint style="info" %}
Regular RouterOS updates provide security patches, bug fixes, new features, and performance improvements. Keeping your RouterOS installation current is essential for security and stability.
{% endhint %}

In WinBox you can check for updates in **System -> Packages**, or you can use terminal with command `/system package update`

RouterOS updates include both the main system and additional packages, with different release channels available for various stability requirements.

***

## Update fundamentals

### RouterOS versioning system

**Version channels:**
- **Stable** - Production-ready releases (recommended)
- **Long-term** - Extended support versions for critical systems
- **Testing** - Beta releases with latest features
- **Development** - Alpha releases for developers

**Version numbering:**
- **Major.minor.patch** format (e.g., 7.16.1)
- **Major versions** - Significant architectural changes
- **Minor versions** - New features and improvements  
- **Patch versions** - Bug fixes and security updates

### Package components

{% code overflow="wrap" %}
```bash
# View installed packages
/system package print

# Common RouterOS packages:
# routeros - Main system package
# wireless - WiFi functionality
# security - IPSec and advanced security
# advanced-tools - Additional tools and features
# gps - GPS functionality
# lcd - LCD display support
```
{% endcode %}

***

## Checking for updates

### Manual update check

Check for available updates using different methods:

{% code overflow="wrap" %}
```bash
# Check for updates via command line
/system package update check-for-updates

# Check update status
/system package update print

# View available updates
/system package update print detail

# Check current version information
/system resource print
/system routerboard print
```
{% endcode %}

### WinBox update process

Using the graphical interface:

1. **Open System -> Packages**
2. **Click "Check For Updates"**
3. **Review available updates**
4. **Click "Download"** to get updates
5. **Click "Reboot"** after download completes

### Web interface updates

Through WebFig/QuickSet:

1. **Navigate to System -> RouterOS**
2. **Click "Check for updates"**  
3. **Download available updates**
4. **Reboot to apply updates**

***

## Download and installation

### Automatic update process

{% code overflow="wrap" %}
```bash
# Download updates automatically
/system package update download

# Check download progress
/system package update print

# Install updates (requires reboot)
/system reboot

# Alternative: Download and install in one command
/system package update install
```
{% endcode %}

### Manual package installation

Install specific packages or versions:

{% code overflow="wrap" %}
```bash
# Upload package file via FTP/SFTP first
/file print where type=".npk"

# Reboot to install uploaded packages
/system reboot

# Remove specific package before reboot
/system package remove wireless

# Check available package space
/system resource print
```
{% endcode %}

### Selective package management

Control which packages are installed:

{% code overflow="wrap" %}
```bash
# Disable automatic package installation
/system package update set channel=stable

# View package dependencies
/system package print detail

# Remove unnecessary packages to save space
/system package remove gps,lcd

# Keep only essential packages for embedded devices
# Typically keep: routeros, security
```
{% endcode %}

***

## Version management

### Channel configuration

Configure update channels for different stability levels:

{% code overflow="wrap" %}
```bash
# Set to stable channel (recommended for production)
/system package update set channel=stable

# Set to long-term support  
/system package update set channel=long-term

# Set to testing channel (for lab environments)
/system package update set channel=testing

# Check current channel
/system package update print
```
{% endcode %}

### Version pinning and rollback

Manage version upgrades and downgrades:

{% code overflow="wrap" %}
```bash
# Check RouterBoard firmware version
/system routerboard print

# Upgrade RouterBoard firmware (if needed)
/system routerboard upgrade

# Note: RouterOS and RouterBoard firmware are separate
# Always upgrade RouterBoard firmware after RouterOS updates

# Backup configuration before major upgrades
/system backup save name=pre-upgrade-backup
/export file=config-backup-v7.15
```
{% endcode %}

***

## Upgrade procedures

### Safe upgrade process

Follow these steps for safe upgrades:

{% code overflow="wrap" %}
```bash
# 1. Backup current configuration
/system backup save name=backup-before-upgrade-$(date +%Y%m%d)
/export file=config-export-$(date +%Y%m%d)

# 2. Check system health
/system health print
/system resource print

# 3. Verify sufficient space
/system resource print

# 4. Check for available updates  
/system package update check-for-updates

# 5. Download updates
/system package update download

# 6. Schedule maintenance window
# Plan for potential downtime during reboot

# 7. Apply updates
/system reboot

# 8. Verify upgrade success
/system resource print
/system package print
```
{% endcode %}

### Batch upgrade process

For multiple devices:

{% code overflow="wrap" %}
```bash
# Create upgrade script for multiple routers
#!/bin/bash
ROUTERS="192.168.1.1 192.168.2.1 192.168.3.1"

for router in $ROUTERS; do
    echo "Upgrading $router..."
    
    # Backup configuration
    ssh admin@$router "/system backup save name=pre-upgrade-backup"
    
    # Check and download updates
    ssh admin@$router "/system package update check-for-updates"
    ssh admin@$router "/system package update download"
    
    # Wait for download completion, then reboot
    ssh admin@$router "/system package update print"
    ssh admin@$router "/system reboot"
    
    echo "Upgrade initiated for $router"
done
```
{% endcode %}

***

## Troubleshooting updates

### Common update issues

{% code overflow="wrap" %}
```bash
# Issue: Update download fails
# Check internet connectivity
/ping 8.8.8.8

# Check DNS resolution
/ping upgrade.mikrotik.com

# Verify firewall allows HTTPS traffic
/ip firewall filter print where protocol=tcp and dst-port=443

# Issue: Insufficient space for updates
/system resource print
/file print where type=file

# Remove old log files and backups
/file remove [find type=file and name~".*\\.backup"]
/log print
```
{% endcode %}

### Recovery procedures

{% code overflow="wrap" %}
```bash
# If upgrade fails and device won't boot:
# 1. Try holding reset button during power-on
# 2. Use Netinstall for complete reinstall
# 3. Restore from backup after reinstall

# Check upgrade logs
/log print where topics~"system"

# Verify package integrity
/system package print detail

# Force package reinstall if corrupted
# Upload packages manually via file transfer
```
{% endcode %}

### Rollback procedures

{% code overflow="wrap" %}
```bash
# Rollback to previous configuration
/system backup load name=backup-before-upgrade

# Downgrade RouterOS (requires manual package installation)
# 1. Download older RouterOS version
# 2. Upload packages via FTP/SCP
# 3. Reboot to install older packages
# Note: Downgrades may not always be supported

# Check configuration compatibility after downgrade
/system resource print
/export show-sensitive
```
{% endcode %}

***

## Automated update management

### Scripted update checks

Create scripts for regular update monitoring:

{% code overflow="wrap" %}
```bash
# Create update notification script
/system script add name=check-updates source={
    /system package update check-for-updates;
    :local updateCount [/system package update get installed-version];
    :if ($updateCount != [/system package update get latest-version]) do={
        /log info "RouterOS updates available";
        /tool e-mail send to="admin@company.com" subject="RouterOS Updates Available" \
            body="Updates available for router $[/system identity get name]";
    }
}

# Schedule weekly update checks
/system scheduler add name=weekly-update-check interval=7d \
    start-time=02:00:00 on-event=check-updates
```
{% endcode %}

### Automatic update installation

{% code overflow="wrap" %}
```bash
# Automatic update script (use with caution)
/system script add name=auto-update source={
    /system backup save name="auto-backup-$[/system clock get date]";
    /system package update check-for-updates;
    :if ([/system package update get installed-version] != [/system package update get latest-version]) do={
        /log info "Starting automatic RouterOS update";
        /system package update download;
        :delay 60s;
        /system package update install;
    } else={
        /log info "RouterOS is up to date";
    }
}

# Schedule monthly automatic updates (maintenance window)
/system scheduler add name=monthly-auto-update interval=30d \
    start-time=02:00:00 on-event=auto-update comment="Automatic monthly updates"
```
{% endcode %}

***

## Version-specific considerations

### Major version upgrades

Special considerations for major version upgrades:

{% code overflow="wrap" %}
```bash
# RouterOS v6 to v7 upgrade considerations:
# 1. Backup configuration (v6 format)
# 2. Check package compatibility
# 3. Review configuration changes needed
# 4. Test in lab environment first
# 5. Plan for extended downtime

# Before v6 to v7 upgrade
/export file=routeros-v6-config
/system backup save name=routeros-v6-backup

# After v7 upgrade, check for configuration issues
/system resource print
/log print where topics~"system,error"
/interface print where disabled=yes
```
{% endcode %}

### Hardware-specific updates

{% code overflow="wrap" %}
```bash
# Check hardware compatibility
/system routerboard print

# RouterBoard firmware updates
/system routerboard print
/system routerboard upgrade

# Special considerations for CCR devices
/system health print
/system cpu print

# Check for hardware-specific packages
/system package print where name~".*ccr.*|.*rb.*"
```
{% endcode %}

***

## Monitoring update status

### Update logging and monitoring

{% code overflow="wrap" %}
```bash
# Enable system logging for updates
/system logging add topics=system,info action=memory
/system logging add topics=system,warning action=disk

# Monitor update process
/log print follow where topics~"system"

# Check update history
/log print where message~".*update.*|.*package.*"

# System resource monitoring during updates
/system resource monitor numbers=0 duration=60
```
{% endcode %}

### Health checks after updates

{% code overflow="wrap" %}
```bash
# Post-update verification checklist
/system resource print              # Check system status
/system health print                # Check hardware health  
/interface print where disabled=yes # Check for disabled interfaces
/ip address print                   # Verify IP configuration
/ip route print where invalid=yes   # Check for invalid routes
/system package print               # Verify all packages loaded

# Network connectivity tests
/ping 8.8.8.8
/tool traceroute 8.8.8.8

# Service status verification
/ip service print where disabled=no
/system identity print
```
{% endcode %}

***

<details>

<summary>Show complete update procedure script</summary>

{% code overflow="wrap" %}
```bash
# Complete RouterOS update procedure script
/system script add name=comprehensive-update source={
    # 1. Pre-update system check
    :log info "Starting RouterOS update procedure";
    
    # Check system resources
    :local freeMemory [/system resource get free-memory];
    :local freeHdd [/system resource get free-hdd-space];
    
    :if ($freeMemory < 10485760) do={
        :log error "Insufficient memory for update";
        :error "Update aborted - insufficient memory";
    }
    
    :if ($freeHdd < 10485760) do={
        :log error "Insufficient disk space for update";
        :error "Update aborted - insufficient disk space";
    }
    
    # 2. Create backup
    :local backupName "pre-update-$[/system clock get date]-$[/system clock get time]";
    /system backup save name=$backupName;
    :log info "Backup created: $backupName";
    
    # Export configuration
    :local exportName "config-export-$[/system clock get date].rsc";
    /export file=$exportName;
    :log info "Configuration exported: $exportName";
    
    # 3. Check for updates
    /system package update check-for-updates;
    :delay 10s;
    
    :local currentVersion [/system package update get installed-version];
    :local latestVersion [/system package update get latest-version];
    
    :if ($currentVersion = $latestVersion) do={
        :log info "RouterOS is already up to date ($currentVersion)";
        :error "No updates available";
    }
    
    :log info "Update available: $currentVersion -> $latestVersion";
    
    # 4. Download updates
    :log info "Downloading RouterOS updates...";
    /system package update download;
    
    # Wait for download completion
    :local downloadStatus;
    :do {
        :delay 30s;
        :set downloadStatus [/system package update get status];
        :log info "Download status: $downloadStatus";
    } while ($downloadStatus = "downloading");
    
    :if ($downloadStatus != "ready to reboot") do={
        :log error "Update download failed: $downloadStatus";
        :error "Update download failed";
    }
    
    # 5. Final checks before reboot
    :log info "Updates downloaded successfully, preparing for reboot...";
    
    # Check RouterBoard firmware
    :local rbCurrent [/system routerboard get current-firmware];
    :local rbUpgrade [/system routerboard get upgrade-firmware];
    
    :if ($rbCurrent != $rbUpgrade) do={
        :log warning "RouterBoard firmware upgrade available: $rbCurrent -> $rbUpgrade";
        /system routerboard upgrade;
        :log info "RouterBoard firmware will be upgraded during reboot";
    }
    
    # 6. Apply updates (reboot)
    :log info "Rebooting to apply RouterOS updates...";
    /system reboot;
}

# Create scheduler to run updates during maintenance window
/system scheduler add name=scheduled-update interval=1w \
    start-time=02:00:00 on-event=comprehensive-update \
    comment="Weekly RouterOS update check and installation"
```
{% endcode %}

</details>

## Update best practices

### Security recommendations

1. **Regular updates** - Apply security patches promptly
2. **Test in lab** - Verify updates in non-production environment first
3. **Backup first** - Always backup before major updates
4. **Monitor advisories** - Subscribe to MikroTik security notifications
5. **Staged rollouts** - Update non-critical devices first

### Operational procedures

1. **Maintenance windows** - Schedule updates during low-traffic periods
2. **Change management** - Document all update procedures
3. **Rollback plans** - Prepare recovery procedures before updates  
4. **Version tracking** - Maintain inventory of device versions
5. **Automated monitoring** - Use scripts to track update status

### Performance considerations

1. **Resource monitoring** - Ensure sufficient memory and storage
2. **Network impact** - Consider bandwidth usage for downloads
3. **Hardware compatibility** - Verify updates work with your hardware
4. **Feature testing** - Test critical features after updates
5. **Performance baseline** - Compare before/after performance metrics
