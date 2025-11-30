---
description: This section covers RouterOS scripting language, automation techniques, and advanced script development for system administration and network management.
icon: code
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

# RouterOS Scripting

{% hint style="info" %}
RouterOS includes a powerful scripting language that allows automation of configuration tasks, monitoring, and system administration. Scripts can be executed manually, scheduled, or triggered by events.
{% endhint %}

In WinBox you can manage scripts in **System -> Scripts**, or you can use terminal with commands `/system script`

RouterOS scripting enables automation of complex tasks, monitoring, and dynamic configuration changes based on system conditions.

***

## Scripting overview

### What is RouterOS scripting?

**RouterOS Script Language:**
- Custom scripting language designed for network device automation
- Event-driven execution capabilities
- Integration with all RouterOS features and commands
- Support for variables, loops, conditions, and functions

**Key capabilities:**
- **System automation** - Automated configuration and maintenance tasks
- **Monitoring and alerting** - Proactive system monitoring and notifications
- **Dynamic configuration** - Adaptive configurations based on conditions
- **Integration** - API integration and external system communication

### When to use scripts

**Common use cases:**
- **Automated backups** - Regular configuration and system backups
- **Health monitoring** - System resource and service monitoring
- **Failover automation** - Automatic failover and recovery procedures
- **User management** - Dynamic user provisioning and access control
- **Traffic analysis** - Network traffic monitoring and reporting
- **Security automation** - Automated threat response and mitigation

***

## Script fundamentals

### Script creation and management

{% code overflow="wrap" %}
```bash
# Create a new script
/system script add name=backup-script source={
    # Script content goes here
    /log info "Starting backup procedure";
}

# List all scripts
/system script print

# Edit existing script
/system script set backup-script source={
    # Updated script content
    /log info "Updated backup procedure";
}

# Execute script manually
/system script run backup-script

# Remove script
/system script remove backup-script
```
{% endcode %}

### Basic script structure

{% code overflow="wrap" %}
```bash
# Basic script template
/system script add name=example-script source={
    # Comments start with #
    
    # Variable declaration
    :local variableName "value";
    :global globalVariable "global value";
    
    # Conditional logic
    :if (condition) do={
        # Commands to execute
        /log info "Condition met";
    } else={
        /log warning "Condition not met";
    };
    
    # Loop structure
    :foreach item in=[/interface print as-value] do={
        /log info ("Interface: " . ($item->"name"));
    };
    
    # Function call
    :local result [:len $variableName];
    
    # Error handling
    :do {
        # Risky operation
        /system reboot;
    } on-error={
        /log error "Reboot failed";
    };
}
```
{% endcode %}

***

## Variables and data types

### Variable declaration and scope

{% code overflow="wrap" %}
```bash
# Local variables (function/script scope)
:local localVar "local value";
:local number 42;
:local boolean true;

# Global variables (system-wide scope)
:global globalVar "global value";
:global counter 0;

# Variable naming rules
:local validName "correct";          # Letters, numbers, underscore
:local valid_name_2 "also correct";  # Underscore and numbers allowed
# :local 2invalid "wrong";           # Cannot start with number
# :local my-var "wrong";             # Hyphens not allowed
```
{% endcode %}

### Data types and operations

{% code overflow="wrap" %}
```bash
# String operations
:local firstName "John";
:local lastName "Doe";
:local fullName ($firstName . " " . $lastName);  # Concatenation
:local nameLength [:len $fullName];               # Length function

# Numeric operations
:local num1 10;
:local num2 5;
:local sum ($num1 + $num2);          # Addition
:local difference ($num1 - $num2);   # Subtraction
:local product ($num1 * $num2);      # Multiplication
:local quotient ($num1 / $num2);     # Division

# Boolean operations
:local isActive true;
:local isEnabled false;
:local bothTrue ($isActive && $isEnabled);  # Logical AND
:local eitherTrue ($isActive || $isEnabled); # Logical OR
:local notActive (!$isActive);              # Logical NOT

# Array operations
:local myArray {1; 2; 3; "text"; true};
:local arrayLength [:len $myArray];
:local firstItem [:pick $myArray 0];         # Get first element
:local subArray [:pick $myArray 1 3];       # Get elements 1-2
```
{% endcode %}

***

## Control structures

### Conditional statements

{% code overflow="wrap" %}
```bash
# Basic if-else structure
:local temperature 25;

:if ($temperature > 30) do={
    /log warning "High temperature detected";
} else={
    :if ($temperature < 10) do={
        /log warning "Low temperature detected";
    } else={
        /log info "Temperature normal";
    };
};

# Multiple conditions
:local cpuLoad 75;
:local memoryUsage 60;

:if ($cpuLoad > 80 && $memoryUsage > 70) do={
    /log error "System overloaded";
} else={
    :if ($cpuLoad > 70 || $memoryUsage > 80) do={
        /log warning "High system load";
    } else={
        /log info "System load normal";
    };
};

# String comparisons
:local userRole "admin";

:if ($userRole = "admin") do={
    /log info "Administrator access granted";
} else={
    :if ($userRole = "user") do={
        /log info "User access granted";
    } else={
        /log warning "Unknown user role";
    };
};
```
{% endcode %}

### Loop structures

{% code overflow="wrap" %}
```bash
# For-each loop with arrays
:local numbers {1; 2; 3; 4; 5};
:foreach number in=$numbers do={
    /log info ("Number: " . $number);
};

# For-each loop with command results
:foreach interface in=[/interface print as-value] do={
    :local ifName ($interface->"name");
    :local ifType ($interface->"type");
    /log info ("Interface $ifName is of type $ifType");
};

# While loop simulation using recursion
:local counter 0;
:local maxCount 5;

:while ($counter < $maxCount) do={
    /log info ("Counter: " . $counter);
    :set counter ($counter + 1);
};

# Nested loops
:local outerArray {"A"; "B"; "C"};
:local innerArray {1; 2; 3};

:foreach outer in=$outerArray do={
    :foreach inner in=$innerArray do={
        /log info ("Combination: " . $outer . $inner);
    };
};
```
{% endcode %}

***

## Functions and built-in commands

### Built-in functions

{% code overflow="wrap" %}
```bash
# String functions
:local text "Hello World";
:local length [:len $text];                    # Get string length
:local substring [:pick $text 0 5];           # Extract substring "Hello"
:local position [:find $text "World"];        # Find substring position
:local uppercase [:tostr $text];              # Convert to string
:local replaced [:replace $text "World" "RouterOS"];  # Replace text

# Conversion functions
:local numberStr "42";
:local number [:tonum $numberStr];            # String to number
:local stringNum [:tostr $number];            # Number to string
:local ipAddress [:toip "192.168.1.1"];      # String to IP
:local timeValue [:totime "1d2h3m4s"];       # String to time

# Array functions  
:local myArray {1; 2; 3; 4; 5};
:local arrayLen [:len $myArray];              # Array length
:local firstThree [:pick $myArray 0 3];      # Get first 3 elements
:local lastTwo [:pick $myArray 3 5];         # Get last 2 elements

# Mathematical functions
:local angle 45;
:local radians ($angle * 3.14159 / 180);     # Degrees to radians
:local result [:sin $radians];                # Sine function
:local logValue [:log 100];                   # Logarithm
:local powerResult [:pow 2 8];                # Power (2^8)
```
{% endcode %}

### Custom functions

{% code overflow="wrap" %}
```bash
# Function definition and usage
:local calculateCircleArea do={
    # Function parameters passed as arguments
    :local radius $1;
    :local pi 3.14159;
    :local area ($pi * $radius * $radius);
    :return $area;
};

# Call the function
:local circleRadius 5;
:local area [$calculateCircleArea $circleRadius];
/log info ("Circle area: " . $area);

# Function with multiple parameters
:local formatUserInfo do={
    :local name $1;
    :local role $2;
    :local active $3;
    
    :local status "inactive";
    :if ($active) do={ :set status "active"; };
    
    :local info ("User: " . $name . ", Role: " . $role . ", Status: " . $status);
    :return $info;
};

# Use the function
:local userInfo [$formatUserInfo "john" "admin" true];
/log info $userInfo;

# Recursive function example
:local factorial do={
    :local n $1;
    :if ($n <= 1) do={
        :return 1;
    } else={
        :local prev [$factorial ($n - 1)];
        :return ($n * $prev);
    };
};

:local result [$factorial 5];  # Result: 120
```
{% endcode %}

***

## Error handling and debugging

### Error handling techniques

{% code overflow="wrap" %}
```bash
# Basic error handling
:do {
    # Potentially problematic operation
    /interface disable ether1;
    /log info "Interface disabled successfully";
} on-error={
    /log error "Failed to disable interface";
};

# Advanced error handling with error capture
:local errorOccurred false;
:local errorMessage "";

:do {
    # Risky operation
    /ip address add address=192.168.1.1/24 interface=ether1;
} on-error={
    :set errorOccurred true;
    :set errorMessage "Failed to add IP address";
};

:if ($errorOccurred) do={
    /log error $errorMessage;
    # Implement fallback procedure
    /log info "Implementing fallback configuration";
} else={
    /log info "IP address configured successfully";
};

# Nested error handling
:do {
    # Outer operation
    :do {
        # Inner operation that might fail
        /system reboot;
    } on-error={
        /log warning "Reboot failed, trying alternative method";
        /system shutdown;
    };
} on-error={
    /log error "System shutdown/reboot failed";
};
```
{% endcode %}

### Debugging techniques

{% code overflow="wrap" %}
```bash
# Debugging with logging
:local debugMode true;

:local debugLog do={
    :local message $1;
    :if ($debugMode) do={
        /log info ("DEBUG: " . $message);
    };
};

# Usage in script
[$debugLog "Starting configuration update"];
:local interfaces [/interface print as-value];
[$debugLog ("Found " . [:len $interfaces] . " interfaces")];

# Variable inspection
:local complexVar {name="test"; value=42; active=true};
:foreach key,value in=$complexVar do={
    /log info ("$key = $value");
};

# Performance timing
:local startTime [:timestamp];
# ... perform operations ...
:local endTime [:timestamp];
:local duration ($endTime - $startTime);
/log info ("Operation completed in " . $duration . " seconds");

# Conditional debugging
:local verboseMode true;
:if ($verboseMode) do={
    :foreach interface in=[/interface print as-value] do={
        /log info ("Interface: " . ($interface->"name") . 
                  ", Type: " . ($interface->"type") . 
                  ", Running: " . ($interface->"running"));
    };
};
```
{% endcode %}

***

## Practical script examples

### System monitoring script

{% code overflow="wrap" %}
```bash
# Comprehensive system monitoring
/system script add name=system-monitor source={
    :local alertThreshold 80;
    :local emailRecipient "admin@company.com";
    
    # Check CPU usage
    :local cpuLoad [/system resource get cpu-load];
    :if ($cpuLoad > $alertThreshold) do={
        :local message ("High CPU usage: " . $cpuLoad . "%");
        /log warning $message;
        /tool e-mail send to=$emailRecipient subject="CPU Alert" body=$message;
    };
    
    # Check memory usage
    :local totalMemory [/system resource get total-memory];
    :local freeMemory [/system resource get free-memory];
    :local memoryUsage ((($totalMemory - $freeMemory) * 100) / $totalMemory);
    
    :if ($memoryUsage > $alertThreshold) do={
        :local message ("High memory usage: " . $memoryUsage . "%");
        /log warning $message;
        /tool e-mail send to=$emailRecipient subject="Memory Alert" body=$message;
    };
    
    # Check disk usage
    :local totalHdd [/system resource get total-hdd-space];
    :local freeHdd [/system resource get free-hdd-space];
    :local diskUsage ((($totalHdd - $freeHdd) * 100) / $totalHdd);
    
    :if ($diskUsage > $alertThreshold) do={
        :local message ("High disk usage: " . $diskUsage . "%");
        /log warning $message;
        /tool e-mail send to=$emailRecipient subject="Disk Alert" body=$message;
    };
    
    # Check interface status
    :foreach interface in=[/interface find where disabled=no] do={
        :local ifName [/interface get $interface name];
        :local isRunning [/interface get $interface running];
        
        :if (!$isRunning) do={
            :local message ("Interface down: " . $ifName);
            /log error $message;
            /tool e-mail send to=$emailRecipient subject="Interface Alert" body=$message;
        };
    };
    
    /log info "System monitoring completed";
}
```
{% endcode %}

### Automated backup script

{% code overflow="wrap" %}
```bash
# Automated backup with rotation
/system script add name=automated-backup source={
    # Configuration
    :local backupPath "/backup";
    :local maxBackups 7;
    :local emailTo "admin@company.com";
    :local ftpServer "backup.company.com";
    :local ftpUser "backup-user";
    :local ftpPassword "backup-password";
    
    # Generate timestamp
    :local currentDate [/system clock get date];
    :local currentTime [/system clock get time];
    :local timestamp ($currentDate . "-" . $currentTime);
    :set timestamp [:replace $timestamp "/" "-"];
    :set timestamp [:replace $timestamp ":" "-"];
    :set timestamp [:replace $timestamp " " "-"];
    
    # Create backup files
    :local backupName ("backup-" . $timestamp);
    :local configName ("config-" . $timestamp);
    
    :do {
        # Create system backup
        /system backup save name=($backupPath . "/" . $backupName);
        /log info ("System backup created: " . $backupName);
        
        # Export configuration
        /export file=($backupPath . "/" . $configName);
        /log info ("Configuration exported: " . $configName);
        
        # Upload to FTP server
        /tool fetch address=$ftpServer src-path=($backupPath . "/" . $backupName . ".backup") \
            user=$ftpUser password=$ftpPassword dst-path=("/" . $backupName . ".backup") \
            mode=ftp upload=yes;
            
        /tool fetch address=$ftpServer src-path=($backupPath . "/" . $configName . ".rsc") \
            user=$ftpUser password=$ftpPassword dst-path=("/" . $configName . ".rsc") \
            mode=ftp upload=yes;
        
        /log info "Backup files uploaded to FTP server";
        
        # Clean up old backups
        :local allFiles [/file find where type="file"];
        :local backupFiles {};
        
        :foreach file in=$allFiles do={
            :local fileName [/file get $file name];
            :if ([:find $fileName "backup-"] = 0 || [:find $fileName "config-"] = 0) do={
                :set backupFiles ($backupFiles, $file);
            };
        };
        
        # Sort and remove oldest if exceeding limit
        :if ([:len $backupFiles] > ($maxBackups * 2)) do={
            :local filesToRemove ([:len $backupFiles] - ($maxBackups * 2));
            :for i from=0 to=($filesToRemove - 1) do={
                :local fileToRemove [:pick $backupFiles $i];
                :local fileName [/file get $fileToRemove name];
                /file remove $fileToRemove;
                /log info ("Old backup removed: " . $fileName);
            };
        };
        
        # Send success notification
        :local message ("Backup completed successfully at " . [/system clock get time]);
        /tool e-mail send to=$emailTo subject="Backup Success" body=$message;
        
    } on-error={
        :local errorMsg "Backup process failed";
        /log error $errorMsg;
        /tool e-mail send to=$emailTo subject="Backup Failed" body=$errorMsg;
    };
}

# Schedule the backup script (daily at 2 AM)
/system scheduler add name=daily-backup interval=1d start-time=02:00:00 on-event=automated-backup
```
{% endcode %}

### Dynamic DNS update script

{% code overflow="wrap" %}
```bash
# Dynamic DNS update script
/system script add name=ddns-update source={
    # Configuration
    :local ddnsHost "home.example.com";
    :local ddnsUser "username";
    :local ddnsPassword "password";
    :local ddnsService "dyndns.org";  # or other DDNS provider
    :local wanInterface "ether1";
    
    # Get current public IP
    :local currentIP "";
    :do {
        # Try multiple methods to get public IP
        :set currentIP [/tool fetch url="http://ipv4.icanhazip.com" output=user as-value]->"data";
        :set currentIP [:pick $currentIP 0 ([:len $currentIP] - 1)];  # Remove newline
    } on-error={
        :do {
            :set currentIP [/tool fetch url="http://checkip.amazonaws.com" output=user as-value]->"data";
            :set currentIP [:pick $currentIP 0 ([:len $currentIP] - 1)];
        } on-error={
            # Fallback: get interface IP
            :set currentIP [/ip address get [find interface=$wanInterface] address];
            :set currentIP [:pick $currentIP 0 [:find $currentIP "/"]];
        };
    };
    
    # Check if IP is valid
    :if ([:len $currentIP] > 0 && [:find $currentIP "."] > 0) do={
        # Get stored IP from global variable
        :global lastKnownIP;
        :if ([:typeof $lastKnownIP] = "nothing") do={ :set lastKnownIP ""; };
        
        # Update only if IP changed
        :if ($currentIP != $lastKnownIP) do={
            /log info ("Public IP changed from " . $lastKnownIP . " to " . $currentIP);
            
            # Construct update URL based on provider
            :local updateURL "";
            :if ($ddnsService = "dyndns.org") do={
                :set updateURL ("http://members.dyndns.org/nic/update?hostname=" . $ddnsHost . "&myip=" . $currentIP);
            } else={
                :if ($ddnsService = "no-ip.com") do={
                    :set updateURL ("http://dynupdate.no-ip.com/nic/update?hostname=" . $ddnsHost . "&myip=" . $currentIP);
                };
            };
            
            # Perform update
            :if ([:len $updateURL] > 0) do={
                :do {
                    :local result [/tool fetch url=$updateURL user=$ddnsUser password=$ddnsPassword output=user as-value];
                    :local response ($result->"data");
                    
                    :if ([:find $response "good"] >= 0 || [:find $response "nochg"] >= 0) do={
                        :set lastKnownIP $currentIP;
                        /log info ("DDNS update successful: " . $ddnsHost . " -> " . $currentIP);
                        
                        # Optional: Send notification email
                        # /tool e-mail send to="admin@company.com" subject="DDNS Updated" \
                        #     body=("Dynamic DNS updated: " . $ddnsHost . " -> " . $currentIP);
                    } else={
                        /log error ("DDNS update failed: " . $response);
                    };
                } on-error={
                    /log error "Failed to contact DDNS provider";
                };
            } else={
                /log error ("Unsupported DDNS service: " . $ddnsService);
            };
        } else={
            /log debug ("Public IP unchanged: " . $currentIP);
        };
    } else={
        /log error "Could not determine public IP address";
    };
}

# Schedule DDNS update every 10 minutes
/system scheduler add name=ddns-updater interval=10m on-event=ddns-update
```
{% endcode %}

***

## Scheduler integration

### Script scheduling

{% code overflow="wrap" %}
```bash
# Schedule script execution
/system scheduler add name=hourly-check interval=1h start-time=00:00:00 on-event=system-monitor
/system scheduler add name=daily-backup interval=1d start-time=02:00:00 on-event=automated-backup
/system scheduler add name=weekly-maintenance interval=7d start-time=03:00:00 on-event=maintenance-script

# One-time scheduled execution
/system scheduler add name=maintenance-window start-time=23:00:00 interval=0 on-event={
    /log info "Starting maintenance window";
    # Maintenance tasks here
    /log info "Maintenance window completed";
}

# Conditional scheduling based on day of week
/system scheduler add name=weekend-task interval=1d on-event={
    :local dayOfWeek [/system clock get date];
    # Simple day check (this is basic - you might want more sophisticated date handling)
    :if ([:find $dayOfWeek "sat"] >= 0 || [:find $dayOfWeek "sun"] >= 0) do={
        /log info "Weekend task executed";
        # Weekend-specific tasks
    };
}
```
{% endcode %}

### Event-driven scripts

{% code overflow="wrap" %}
```bash
# Interface state change monitoring
/system script add name=interface-monitor source={
    :foreach interface in=[/interface find] do={
        :local ifName [/interface get $interface name];
        :local wasRunning [/interface get $interface running];
        
        # Store previous state in global variable
        :global interfaceStates;
        :if ([:typeof $interfaceStates] = "nothing") do={ 
            :set interfaceStates {}; 
        };
        
        # Check for state change
        :local prevState ($interfaceStates->$ifName);
        :if ([:typeof $prevState] = "nothing") do={ :set prevState $wasRunning; };
        
        :if ($wasRunning != $prevState) do={
            :if ($wasRunning) do={
                /log info ("Interface " . $ifName . " came up");
                # Interface up actions
            } else={
                /log warning ("Interface " . $ifName . " went down");
                # Interface down actions - maybe failover?
            };
        };
        
        # Update stored state
        :set ($interfaceStates->$ifName) $wasRunning;
    };
}

# Schedule frequent interface monitoring
/system scheduler add name=interface-check interval=30s on-event=interface-monitor
```
{% endcode %}

***

## Advanced scripting techniques

### Working with APIs and external data

{% code overflow="wrap" %}
```bash
# API integration script
/system script add name=api-integration source={
    # Configuration
    :local apiEndpoint "https://api.example.com/status";
    :local apiKey "your-api-key-here";
    :local timeout 10;
    
    :do {
        # Fetch data from external API
        :local result [/tool fetch url=$apiEndpoint http-method=get \
            http-header-field="Authorization: Bearer $apiKey" \
            output=user as-value];
            
        :local responseData ($result->"data");
        :local httpStatus ($result->"status");
        
        :if ($httpStatus = "finished") do={
            /log info "API response received";
            
            # Parse JSON-like response (basic parsing)
            # RouterOS doesn't have built-in JSON parser, so this is simplified
            :if ([:find $responseData "\"status\":\"ok\""] >= 0) do={
                /log info "External service status: OK";
                # Enable certain features or routes
            } else={
                /log warning "External service status: Not OK";
                # Disable features or implement fallback
            };
        } else={
            /log error ("API request failed with status: " . $httpStatus);
        };
        
    } on-error={
        /log error "Failed to contact external API";
        # Implement offline behavior
    };
}
```
{% endcode %}

### Configuration templates and mass deployment

{% code overflow="wrap" %}
```bash
# Configuration template script
/system script add name=deploy-config-template source={
    # Template variables
    :local siteId "SITE001";
    :local mgmtVlan 100;
    :local userVlan 200;
    :local guestVlan 300;
    :local wanInterface "ether1";
    :local lanBridge "bridge1";
    
    /log info ("Deploying configuration template for site: " . $siteId);
    
    # Create VLANs
    :do {
        /interface vlan add name=("vlan" . $mgmtVlan . "-mgmt") vlan-id=$mgmtVlan interface=$lanBridge;
        /interface vlan add name=("vlan" . $userVlan . "-users") vlan-id=$userVlan interface=$lanBridge;
        /interface vlan add name=("vlan" . $guestVlan . "-guest") vlan-id=$guestVlan interface=$lanBridge;
        /log info "VLANs created successfully";
    } on-error={
        /log error "Failed to create VLANs";
    };
    
    # Configure IP addresses
    :do {
        /ip address add address=("192.168." . $mgmtVlan . ".1/24") interface=("vlan" . $mgmtVlan . "-mgmt");
        /ip address add address=("192.168." . $userVlan . ".1/24") interface=("vlan" . $userVlan . "-users");
        /ip address add address=("192.168." . $guestVlan . ".1/24") interface=("vlan" . $guestVlan . "-guest");
        /log info "IP addresses configured";
    } on-error={
        /log error "Failed to configure IP addresses";
    };
    
    # Create DHCP servers
    :local dhcpPools {"mgmt"; "users"; "guest"};
    :local vlans {$mgmtVlan; $userVlan; $guestVlan};
    
    :for i from=0 to=([:len $dhcpPools] - 1) do={
        :local poolName [:pick $dhcpPools $i];
        :local vlanId [:pick $vlans $i];
        :local network ("192.168." . $vlanId . ".0/24");
        :local poolRange ("192.168." . $vlanId . ".10-192.168." . $vlanId . ".100");
        
        :do {
            /ip pool add name=($poolName . "-pool") ranges=$poolRange;
            /ip dhcp-server add name=($poolName . "-dhcp") interface=("vlan" . $vlanId . "-" . $poolName) \
                address-pool=($poolName . "-pool") disabled=no;
            /ip dhcp-server network add address=$network gateway=("192.168." . $vlanId . ".1") \
                dns-server=("192.168." . $vlanId . ".1");
        } on-error={
            /log error ("Failed to configure DHCP for " . $poolName);
        };
    };
    
    /log info ("Configuration template deployment completed for " . $siteId);
}
```
{% endcode %}

***

## Script security and best practices

### Security considerations

{% code overflow="wrap" %}
```bash
# Secure script practices
/system script add name=secure-script-example source={
    # 1. Input validation
    :local userInput $1;
    :if ([:len $userInput] = 0 || [:len $userInput] > 50) do={
        /log error "Invalid input length";
        :error "Script terminated due to invalid input";
    };
    
    # 2. Sanitize input (remove dangerous characters)
    :local sanitizedInput $userInput;
    :set sanitizedInput [:replace $sanitizedInput ";" ""];
    :set sanitizedInput [:replace $sanitizedInput "|" ""];
    :set sanitizedInput [:replace $sanitizedInput "&" ""];
    
    # 3. Use local variables to prevent global namespace pollution
    :local tempVar "safe local variable";
    
    # 4. Implement proper error handling
    :do {
        # Risky operation
        /log info ("Processing: " . $sanitizedInput);
    } on-error={
        /log error "Operation failed safely";
        # Don't expose sensitive error details
    };
    
    # 5. Clear sensitive variables
    :set sanitizedInput "";
    :set tempVar "";
}

# Script with permission checking
/system script add name=admin-only-script source={
    # Check if current user has admin privileges
    :local currentUser [/system identity get name];  # This is simplified
    
    # In practice, you'd check group membership or permissions
    :local hasAdminAccess true;  # Implement proper permission check
    
    :if (!$hasAdminAccess) do={
        /log warning ("Unauthorized script execution attempt by: " . $currentUser);
        :error "Insufficient permissions";
    };
    
    /log info ("Admin script executed by: " . $currentUser);
    # Admin-level operations here
}
```
{% endcode %}

### Performance optimization

{% code overflow="wrap" %}
```bash
# Optimized script performance
/system script add name=performance-optimized source={
    # 1. Minimize command execution in loops
    :local allInterfaces [/interface print as-value];  # Execute once, store result
    
    :foreach interface in=$allInterfaces do={
        # Process stored data instead of repeated commands
        :local ifName ($interface->"name");
        :local ifType ($interface->"type");
        # Process without additional commands
    };
    
    # 2. Use efficient data structures
    :local interfaceMap {};  # Associative array for fast lookups
    :foreach interface in=$allInterfaces do={
        :local ifName ($interface->"name");
        :set ($interfaceMap->$ifName) $interface;
    };
    
    # 3. Batch operations when possible
    :local commandBatch "";
    :foreach interface in=$allInterfaces do={
        :if (($interface->"type") = "ether") do={
            # Build batch command instead of executing immediately
            :set commandBatch ($commandBatch . "/interface set " . ($interface->"name") . " comment=\"Ethernet\"; ");
        };
    };
    
    # Execute batch if not empty
    :if ([:len $commandBatch] > 0) do={
        # In practice, you'd execute the batch
        /log info ("Would execute batch: " . [:pick $commandBatch 0 100] . "...");
    };
    
    # 4. Early exit conditions
    :local targetInterface "ether1";
    :local found false;
    
    :foreach interface in=$allInterfaces do={
        :if (($interface->"name") = $targetInterface) do={
            :set found true;
            /log info ("Found target interface: " . $targetInterface);
            # Break equivalent - set condition to exit loop early
            :set allInterfaces {};  # Clear array to exit loop
        };
    };
}
```
{% endcode %}

***

<details>

<summary>Show complete automated network monitoring script</summary>

{% code overflow="wrap" %}
```bash
# Comprehensive network monitoring and alerting script
/system script add name=network-monitor-pro source={
    # Configuration section
    :local alertEmail "admin@company.com";
    :local snmpCommunity "monitoring";
    :local alertThresholds {
        cpu=80;
        memory=85;
        disk=90;
        temperature=70;
        interfaceErrors=100;
    };
    
    # Initialize global monitoring state
    :global monitoringState;
    :if ([:typeof $monitoringState] = "nothing") do={
        :set monitoringState {
            lastAlerts={};
            counters={};
            baselines={};
        };
    };
    
    # Helper function for alert throttling
    :local shouldAlert do={
        :local alertKey $1;
        :local currentTime [:timestamp];
        :local cooldownPeriod 3600;  # 1 hour cooldown
        
        :local lastAlert (($monitoringState->"lastAlerts")->$alertKey);
        :if ([:typeof $lastAlert] = "nothing") do={ :set lastAlert 0; };
        
        :if (($currentTime - $lastAlert) > $cooldownPeriod) do={
            :set (($monitoringState->"lastAlerts")->$alertKey) $currentTime;
            :return true;
        };
        :return false;
    };
    
    # System resource monitoring
    :local sysResource [/system resource get];
    :local cpuLoad ($sysResource->"cpu-load");
    :local memoryUsage ((($sysResource->"total-memory") - ($sysResource->"free-memory")) * 100 / ($sysResource->"total-memory"));
    :local diskUsage ((($sysResource->"total-hdd-space") - ($sysResource->"free-hdd-space")) * 100 / ($sysResource->"total-hdd-space"));
    
    # CPU monitoring
    :if ($cpuLoad > ($alertThresholds->"cpu")) do={
        :if ([$shouldAlert "cpu-high"]) do={
            :local message ("Critical: CPU usage at " . $cpuLoad . "%");
            /log error $message;
            /tool e-mail send to=$alertEmail subject="CPU Alert" body=$message;
        };
    };
    
    # Memory monitoring  
    :if ($memoryUsage > ($alertThresholds->"memory")) do={
        :if ([$shouldAlert "memory-high"]) do={
            :local message ("Critical: Memory usage at " . $memoryUsage . "%");
            /log error $message;
            /tool e-mail send to=$alertEmail subject="Memory Alert" body=$message;
        };
    };
    
    # Disk monitoring
    :if ($diskUsage > ($alertThresholds->"disk")) do={
        :if ([$shouldAlert "disk-high"]) do={
            :local message ("Critical: Disk usage at " . $diskUsage . "%");
            /log error $message;
            /tool e-mail send to=$alertEmail subject="Disk Alert" body=$message;
        };
    };
    
    # Temperature monitoring (if available)
    :do {
        :local temperature [/system health get temperature];
        :if ($temperature > ($alertThresholds->"temperature")) do={
            :if ([$shouldAlert "temperature-high"]) do={
                :local message ("Warning: High temperature detected: " . $temperature . "°C");
                /log warning $message;
                /tool e-mail send to=$alertEmail subject="Temperature Alert" body=$message;
            };
        };
    } on-error={
        # Temperature monitoring not available on this device
    };
    
    # Interface monitoring
    :foreach interface in=[/interface find where disabled=no] do={
        :local ifName [/interface get $interface name];
        :local ifStats [/interface get $interface];
        :local isRunning ($ifStats->"running");
        :local rxErrors ($ifStats->"rx-error");
        :local txErrors ($ifStats->"tx-error");
        
        # Interface state monitoring
        :local prevState (($monitoringState->"counters")->($ifName . "-running"));
        :if ([:typeof $prevState] = "nothing") do={ :set prevState $isRunning; };
        
        :if ($isRunning != $prevState) do={
            :if ($isRunning) do={
                :local message ("Interface " . $ifName . " is now UP");
                /log info $message;
                /tool e-mail send to=$alertEmail subject="Interface Recovery" body=$message;
            } else={
                :local message ("Interface " . $ifName . " is DOWN");
                /log error $message;
                /tool e-mail send to=$alertEmail subject="Interface Down" body=$message;
            };
        };
        
        # Error rate monitoring
        :local totalErrors ($rxErrors + $txErrors);
        :local prevErrors (($monitoringState->"counters")->($ifName . "-errors"));
        :if ([:typeof $prevErrors] = "nothing") do={ :set prevErrors $totalErrors; };
        
        :local errorDelta ($totalErrors - $prevErrors);
        :if ($errorDelta > ($alertThresholds->"interfaceErrors")) do={
            :if ([$shouldAlert ($ifName . "-errors")]) do={
                :local message ("High error rate on " . $ifName . ": " . $errorDelta . " errors");
                /log warning $message;
                /tool e-mail send to=$alertEmail subject="Interface Errors" body=$message;
            };
        };
        
        # Update counters
        :set (($monitoringState->"counters")->($ifName . "-running")) $isRunning;
        :set (($monitoringState->"counters")->($ifName . "-errors")) $totalErrors;
    };
    
    # Service monitoring
    :local criticalServices {"ssh"; "winbox"; "www-ssl"};
    :foreach serviceName in=$criticalServices do={
        :local serviceRunning false;
        :do {
            :local serviceInfo [/ip service get [find name=$serviceName]];
            :set serviceRunning (!($serviceInfo->"disabled"));
        } on-error={
            # Service doesn't exist or other error
        };
        
        :local prevServiceState (($monitoringState->"counters")->($serviceName . "-running"));
        :if ([:typeof $prevServiceState] = "nothing") do={ :set prevServiceState $serviceRunning; };
        
        :if ($serviceRunning != $prevServiceState && !$serviceRunning) do={
            :if ([$shouldAlert ($serviceName . "-down")]) do={
                :local message ("Critical service down: " . $serviceName);
                /log error $message;
                /tool e-mail send to=$alertEmail subject="Service Alert" body=$message;
            };
        };
        
        :set (($monitoringState->"counters")->($serviceName . "-running")) $serviceRunning;
    };
    
    # Connectivity monitoring (ping test)
    :local pingTargets {"8.8.8.8"; "1.1.1.1"; "208.67.222.222"};
    :local successfulPings 0;
    
    :foreach target in=$pingTargets do={
        :do {
            /tool ping address=$target count=3 timeout=3s;
            :set successfulPings ($successfulPings + 1);
        } on-error={
            # Ping failed
        };
    };
    
    :if ($successfulPings = 0) do={
        :if ([$shouldAlert "internet-connectivity"]) do={
            :local message "Internet connectivity lost - all ping targets failed";
            /log error $message;
            /tool e-mail send to=$alertEmail subject="Connectivity Alert" body=$message;
        };
    };
    
    # Generate summary report
    :local timestamp [/system clock get time];
    :local reportSummary ("Monitoring Report - " . $timestamp . "\n");
    :set reportSummary ($reportSummary . "CPU: " . $cpuLoad . "% | ");
    :set reportSummary ($reportSummary . "Memory: " . $memoryUsage . "% | ");
    :set reportSummary ($reportSummary . "Disk: " . $diskUsage . "% | ");
    :set reportSummary ($reportSummary . "Connectivity: " . $successfulPings . "/" . [:len $pingTargets]);
    
    /log info $reportSummary;
    
    # Optional: Store metrics for trending (simplified)
    :set (($monitoringState->"baselines")->"last-cpu") $cpuLoad;
    :set (($monitoringState->"baselines")->"last-memory") $memoryUsage;
    :set (($monitoringState->"baselines")->"last-disk") $diskUsage;
}

# Schedule comprehensive monitoring every 5 minutes
/system scheduler add name=network-monitor interval=5m on-event=network-monitor-pro
```
{% endcode %}

</details>

## Scripting best practices

### Code organization and maintainability

1. **Use meaningful variable names** - `$interfaceName` instead of `$if`
2. **Comment your code** - Explain complex logic and business rules
3. **Modular functions** - Break complex scripts into reusable functions
4. **Error handling** - Always implement appropriate error handling
5. **Input validation** - Validate and sanitize all inputs

### Performance considerations

1. **Minimize command calls** - Store results and reuse them
2. **Efficient loops** - Use appropriate loop structures
3. **Early exits** - Implement conditions to exit early when possible
4. **Resource management** - Clean up variables and resources
5. **Batch operations** - Group similar operations when possible

### Security guidelines

1. **Input sanitization** - Always validate and clean user inputs
2. **Permission checks** - Verify user permissions before sensitive operations
3. **Secure communications** - Use encrypted channels for external communications
4. **Logging** - Log important operations and security events
5. **Secret management** - Handle passwords and keys securely