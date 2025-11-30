---
description: Learn how to handle user input in RouterOS scripts, including interactive prompts, parameter validation, and secure input processing.
icon: keyboard
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

# User Input Handling

{% hint style="info" %}
RouterOS scripts can interact with users through various input methods, from simple parameter passing to interactive prompts. Proper input handling ensures script reliability and security.
{% endhint %}

User input handling in RouterOS enables creation of interactive scripts, configuration wizards, and dynamic automation tools that adapt to user requirements.

***

## Input methods overview

### Script parameter passing

{% code overflow="wrap" %}
```bash
# Script with parameters - accessed via $1, $2, etc.
/system script add name=user-management source={
    # Parameters: $1=username, $2=password, $3=group
    :local userName $1;
    :local userPassword $2;
    :local userGroup $3;
    
    # Validate parameters
    :if ([:len $userName] = 0) do={
        /log error "Username parameter is required";
        :error "Missing username parameter";
    };
    
    :if ([:len $userPassword] = 0) do={
        /log error "Password parameter is required";
        :error "Missing password parameter";
    };
    
    # Set default group if not provided
    :if ([:len $userGroup] = 0) do={
        :set userGroup "read";
    };
    
    # Create user with provided parameters
    /user add name=$userName password=$userPassword group=$userGroup;
    /log info ("User " . $userName . " created with group " . $userGroup);
}

# Execute script with parameters
# /system script run user-management "john" "password123" "full"
```
{% endcode %}

### Environment variable access

{% code overflow="wrap" %}
```bash
# Accessing system and environment information
/system script add name=environment-info source={
    # Get system identity
    :local systemName [/system identity get name];
    :local currentTime [/system clock get time];
    :local currentDate [/system clock get date];
    
    # Get current user context (limited in RouterOS)
    :local routerboardInfo [/system routerboard get];
    :local firmwareVersion ($routerboardInfo->"current-firmware");
    
    # Display environment information
    /log info ("System: " . $systemName);
    /log info ("Current time: " . $currentDate . " " . $currentTime);
    /log info ("Firmware: " . $firmwareVersion);
    
    # Get interface information as "environment"
    :local interfaceCount [/interface print count-only];
    /log info ("Total interfaces: " . $interfaceCount);
    
    # Get network configuration as input context
    :local ipAddresses [/ip address print as-value];
    :foreach addr in=$ipAddresses do={
        :local interface ($addr->"interface");
        :local address ($addr->"address");
        /log info ("Interface " . $interface . ": " . $address);
    };
}
```
{% endcode %}

***

## Interactive user prompts

### Simple yes/no prompts

{% code overflow="wrap" %}
```bash
# Interactive confirmation script
/system script add name=interactive-reboot source={
    # Simulate interactive prompt (RouterOS doesn't have built-in prompts)
    # This would typically be handled by external script or manual execution
    
    # Create a simple confirmation mechanism using global variables
    :global userConfirmation;
    
    # Check if confirmation was provided
    :if ([:typeof $userConfirmation] = "nothing") do={
        /log info "Please set global variable 'userConfirmation' to 'yes' to proceed";
        /log info "Example: :global userConfirmation yes";
        /log info "Then run the script again";
        :error "User confirmation required";
    };
    
    :if ($userConfirmation = "yes") do={
        /log info "User confirmed - proceeding with reboot";
        # Clear the confirmation
        :set userConfirmation;
        /system reboot;
    } else={
        /log info "Reboot cancelled by user";
        :set userConfirmation;
    };
}

# Usage pattern:
# :global userConfirmation yes
# /system script run interactive-reboot
```
{% endcode %}

### Configuration wizard simulation

{% code overflow="wrap" %}
```bash
# Configuration wizard using global variables for input
/system script add name=wifi-setup-wizard source={
    # Input variables (set by user before running script)
    :global wifiSSID;
    :global wifiPassword;
    :global wifiChannel;
    :global wifiCountry;
    
    # Default values
    :if ([:typeof $wifiSSID] = "nothing") do={ :set wifiSSID "MyNetwork"; };
    :if ([:typeof $wifiPassword] = "nothing") do={ :set wifiPassword ""; };
    :if ([:typeof $wifiChannel] = "nothing") do={ :set wifiChannel "auto"; };
    :if ([:typeof $wifiCountry] = "nothing") do={ :set wifiCountry "united_states"; };
    
    /log info "WiFi Setup Wizard Starting...";
    /log info ("SSID: " . $wifiSSID);
    /log info ("Channel: " . $wifiChannel);
    /log info ("Country: " . $wifiCountry);
    
    # Validate input
    :if ([:len $wifiSSID] = 0) do={
        /log error "WiFi SSID cannot be empty";
        :error "Invalid SSID";
    };
    
    :if ([:len $wifiPassword] < 8) do={
        /log error "WiFi password must be at least 8 characters";
        :error "Invalid password";
    };
    
    # Configure WiFi (example for new WiFi package)
    :do {
        # Create security profile
        /interface wifi security add name=wizard-security \
            authentication-types=wpa2-psk,wpa3-psk \
            encryption=ccmp \
            passphrase=$wifiPassword;
            
        # Configure WiFi interface
        /interface wifi set wifi1 \
            disabled=no \
            channel=$wifiChannel \
            country=$wifiCountry;
            
        # Create WiFi network
        /interface wifi add name=wifi-home \
            master-interface=wifi1 \
            ssid=$wifiSSID \
            security=wizard-security \
            disabled=no;
            
        /log info "WiFi configuration completed successfully";
        
        # Clear input variables
        :set wifiSSID;
        :set wifiPassword;
        :set wifiChannel;
        :set wifiCountry;
        
    } on-error={
        /log error "WiFi configuration failed";
    };
}

# Usage example:
# :global wifiSSID "MyHomeNetwork"
# :global wifiPassword "SuperSecure123!"
# :global wifiChannel 36
# :global wifiCountry "united_states"
# /system script run wifi-setup-wizard
```
{% endcode %}

***

## Input validation and sanitization

### Basic input validation

{% code overflow="wrap" %}
```bash
# Comprehensive input validation functions
/system script add name=input-validation-demo source={
    # Input validation functions
    :local validateIP do={
        :local ipAddress $1;
        
        # Basic IP address format validation
        :if ([:len $ipAddress] = 0) do={
            :return false;
        };
        
        # Check for valid IP format (simplified)
        :local dotCount 0;
        :local i 0;
        :while ($i < [:len $ipAddress]) do={
            :local char [:pick $ipAddress $i ($i + 1)];
            :if ($char = ".") do={
                :set dotCount ($dotCount + 1);
            };
            :set i ($i + 1);
        };
        
        # Should have exactly 3 dots for IPv4
        :return ($dotCount = 3);
    };
    
    :local validatePort do={
        :local port $1;
        
        # Convert string to number if needed
        :local portNum $port;
        :if ([:typeof $port] = "str") do={
            :do {
                :set portNum [:tonum $port];
            } on-error={
                :return false;
            };
        };
        
        # Check port range (1-65535)
        :return ($portNum >= 1 && $portNum <= 65535);
    };
    
    :local validateSSID do={
        :local ssid $1;
        
        # SSID validation rules
        :if ([:len $ssid] = 0 || [:len $ssid] > 32) do={
            :return false;
        };
        
        # Check for invalid characters (simplified)
        :if ($ssid ~ "[\x00-\x1F]") do={
            :return false;  # Control characters not allowed
        };
        
        :return true;
    };
    
    :local validatePassword do={
        :local password $1;
        :local minLength $2;
        
        # Set default minimum length
        :if ([:len $minLength] = 0) do={ :set minLength 8; };
        
        # Length check
        :if ([:len $password] < $minLength) do={
            :return false;
        };
        
        # Check for at least one number (simplified)
        :local hasNumber ($password ~ ".*[0-9].*");
        
        # Check for at least one letter (simplified)
        :local hasLetter ($password ~ ".*[A-Za-z].*");
        
        :return ($hasNumber && $hasLetter);
    };
    
    # Test the validation functions
    :local testIP "192.168.1.1";
    :local testPort "8080";
    :local testSSID "MyNetwork";
    :local testPassword "SecurePass123";
    
    :local ipValid [$validateIP $testIP];
    :local portValid [$validatePort $testPort];
    :local ssidValid [$validateSSID $testSSID];
    :local passwordValid [$validatePassword $testPassword 8];
    
    /log info ("IP " . $testIP . " is valid: " . $ipValid);
    /log info ("Port " . $testPort . " is valid: " . $portValid);
    /log info ("SSID '" . $testSSID . "' is valid: " . $ssidValid);
    /log info ("Password is valid: " . $passwordValid);
}
```
{% endcode %}

### Input sanitization

{% code overflow="wrap" %}
```bash
# Input sanitization and cleaning functions
/system script add name=input-sanitization source={
    # String sanitization function
    :local sanitizeString do={
        :local input $1;
        :local allowedChars $2;  # Optional: specify allowed characters
        
        :local cleaned $input;
        
        # Remove dangerous characters for command injection prevention
        :set cleaned [:replace $cleaned ";" ""];
        :set cleaned [:replace $cleaned "|" ""];
        :set cleaned [:replace $cleaned "&" ""];
        :set cleaned [:replace $cleaned "`" ""];
        :set cleaned [:replace $cleaned "\$" ""];
        :set cleaned [:replace $cleaned "'" ""];
        :set cleaned [:replace $cleaned "\"" ""];
        
        # Remove control characters
        :set cleaned [:replace $cleaned "\n" ""];
        :set cleaned [:replace $cleaned "\r" ""];
        :set cleaned [:replace $cleaned "\t" " "];
        
        # Trim whitespace (simplified)
        :while ([:pick $cleaned 0 1] = " ") do={
            :set cleaned [:pick $cleaned 1 [:len $cleaned]];
        };
        :while ([:pick $cleaned ([:len $cleaned] - 1) [:len $cleaned]] = " ") do={
            :set cleaned [:pick $cleaned 0 ([:len $cleaned] - 1)];
        };
        
        :return $cleaned;
    };
    
    # Filename sanitization
    :local sanitizeFilename do={
        :local filename $1;
        :local cleaned $filename;
        
        # Remove invalid filename characters
        :set cleaned [:replace $cleaned "/" "_"];
        :set cleaned [:replace $cleaned "\\" "_"];
        :set cleaned [:replace $cleaned ":" "_"];
        :set cleaned [:replace $cleaned "*" "_"];
        :set cleaned [:replace $cleaned "?" "_"];
        :set cleaned [:replace $cleaned "\"" "_"];
        :set cleaned [:replace $cleaned "<" "_"];
        :set cleaned [:replace $cleaned ">" "_"];
        :set cleaned [:replace $cleaned "|" "_"];
        
        # Limit length
        :if ([:len $cleaned] > 50) do={
            :set cleaned [:pick $cleaned 0 50];
        };
        
        :return $cleaned;
    };
    
    # IP address normalization
    :local normalizeIP do={
        :local ip $1;
        :local normalized $ip;
        
        # Remove whitespace
        :set normalized [:replace $normalized " " ""];
        
        # Validate basic format
        :if (!($normalized ~ "^[0-9.]+$")) do={
            :return "";  # Invalid format
        };
        
        :return $normalized;
    };
    
    # Username sanitization for system accounts
    :local sanitizeUsername do={
        :local username $1;
        :local cleaned $username;
        
        # Convert to lowercase
        :set cleaned [:replace $cleaned "A" "a"];
        :set cleaned [:replace $cleaned "B" "b"];
        :set cleaned [:replace $cleaned "C" "c"];
        # ... (simplified, full implementation would handle all letters)
        
        # Remove invalid characters for usernames
        :set cleaned [:replace $cleaned " " "_"];
        :set cleaned [:replace $cleaned "-" "_"];
        :set cleaned [:replace $cleaned "." "_"];
        
        # Ensure it starts with a letter (add prefix if needed)
        :if ([:pick $cleaned 0 1] ~ "[0-9]") do={
            :set cleaned ("user_" . $cleaned);
        };
        
        # Limit length
        :if ([:len $cleaned] > 20) do={
            :set cleaned [:pick $cleaned 0 20];
        };
        
        :return $cleaned;
    };
    
    # Test sanitization functions
    :local dangerousInput "test; /system reboot & dangerous";
    :local messyFilename "My File/Name: Test*.txt";
    :local malformedIP " 192.168.1.1 ";
    :local weirdUsername "123-John.Doe@Company";
    
    :local cleanString [$sanitizeString $dangerousInput];
    :local cleanFilename [$sanitizeFilename $messyFilename];
    :local cleanIP [$normalizeIP $malformedIP];
    :local cleanUsername [$sanitizeUsername $weirdUsername];
    
    /log info ("Original: '" . $dangerousInput . "'");
    /log info ("Cleaned: '" . $cleanString . "'");
    /log info ("Filename: '" . $messyFilename . "' -> '" . $cleanFilename . "'");
    /log info ("IP: '" . $malformedIP . "' -> '" . $cleanIP . "'");
    /log info ("Username: '" . $weirdUsername . "' -> '" . $cleanUsername . "'");
}
```
{% endcode %}

***

## Parameter processing patterns

### Command-line style arguments

{% code overflow="wrap" %}
```bash
# Command-line argument parsing simulation
/system script add name=arg-parser-demo source={
    # Simulate command line: script-name --host=192.168.1.1 --port=8080 --enable
    # Parameters passed as single string and parsed
    
    :local parseArguments do={
        :local argString $1;
        :local args {};
        
        # Split arguments by spaces (simplified parser)
        :local parts {};
        :local currentArg "";
        :local i 0;
        
        # Simple space-based splitting
        :while ($i < [:len $argString]) do={
            :local char [:pick $argString $i ($i + 1)];
            :if ($char = " ") do={
                :if ([:len $currentArg] > 0) do={
                    :set parts ($parts, $currentArg);
                    :set currentArg "";
                };
            } else={
                :set currentArg ($currentArg . $char);
            };
            :set i ($i + 1);
        };
        
        # Add last argument
        :if ([:len $currentArg] > 0) do={
            :set parts ($parts, $currentArg);
        };
        
        # Process each part
        :foreach part in=$parts do={
            :if ([:pick $part 0 2] = "--") do={
                # Long option
                :local optionPart [:pick $part 2 [:len $part]];
                :local equalPos [:find $optionPart "="];
                
                :if ([:typeof $equalPos] != "nothing") do={
                    # Option with value: --key=value
                    :local key [:pick $optionPart 0 $equalPos];
                    :local value [:pick $optionPart ($equalPos + 1) [:len $optionPart]];
                    :set ($args->$key) $value;
                } else={
                    # Boolean option: --flag
                    :set ($args->$optionPart) true;
                };
            } else={
                :if ([:pick $part 0 1] = "-") do={
                    # Short option (simplified)
                    :local shortOpt [:pick $part 1 [:len $part]];
                    :set ($args->$shortOpt) true;
                };
            };
        };
        
        :return $args;
    };
    
    # Example usage of argument parser
    :local testArgs "--host=192.168.1.1 --port=8080 --verbose -f --enable";
    :local parsedArgs [$parseArguments $testArgs];
    
    # Access parsed arguments
    :local host ($parsedArgs->"host");
    :local port ($parsedArgs->"port");
    :local verbose ($parsedArgs->"verbose");
    :local forceMode ($parsedArgs->"f");
    :local enabled ($parsedArgs->"enable");
    
    /log info ("Host: " . $host);
    /log info ("Port: " . $port);
    /log info ("Verbose: " . $verbose);
    /log info ("Force mode: " . $forceMode);
    /log info ("Enabled: " . $enabled);
}
```
{% endcode %}

### Configuration file input

{% code overflow="wrap" %}
```bash
# Configuration file processing
/system script add name=config-file-processor source={
    # Process configuration from file content
    :local processConfigFile do={
        :local configContent $1;
        :local config {};
        
        # Split content by lines (simplified)
        :local lines {};
        :local currentLine "";
        :local i 0;
        
        :while ($i < [:len $configContent]) do={
            :local char [:pick $configContent $i ($i + 1)];
            :if ($char = "\n") do={
                :if ([:len $currentLine] > 0) do={
                    :set lines ($lines, $currentLine);
                    :set currentLine "";
                };
            } else={
                :set currentLine ($currentLine . $char);
            };
            :set i ($i + 1);
        };
        
        # Add last line
        :if ([:len $currentLine] > 0) do={
            :set lines ($lines, $currentLine);
        };
        
        # Process each line
        :foreach line in=$lines do={
            # Skip comments and empty lines
            :local trimmed $line;
            # Basic trim (remove leading/trailing spaces)
            
            :if ([:len $trimmed] > 0 && [:pick $trimmed 0 1] != "#") do={
                :local equalPos [:find $trimmed "="];
                :if ([:typeof $equalPos] != "nothing") do={
                    :local key [:pick $trimmed 0 $equalPos];
                    :local value [:pick $trimmed ($equalPos + 1) [:len $trimmed]];
                    
                    # Remove quotes from value if present
                    :if ([:pick $value 0 1] = "\"" && [:pick $value ([:len $value] - 1) [:len $value]] = "\"") do={
                        :set value [:pick $value 1 ([:len $value] - 1)];
                    };
                    
                    :set ($config->$key) $value;
                };
            };
        };
        
        :return $config;
    };
    
    # Example configuration content
    :local sampleConfig "# Network Configuration\nhost_name=router1\nip_address=192.168.1.1\nsubnet_mask=255.255.255.0\ngateway=192.168.1.254\ndns_server=\"8.8.8.8\"\nenable_dhcp=true\n# End of config";
    
    :local parsedConfig [$processConfigFile $sampleConfig];
    
    # Use parsed configuration
    :local hostName ($parsedConfig->"host_name");
    :local ipAddress ($parsedConfig->"ip_address");
    :local gateway ($parsedConfig->"gateway");
    :local dnsServer ($parsedConfig->"dns_server");
    :local enableDHCP ($parsedConfig->"enable_dhcp");
    
    /log info ("Configuration loaded:");
    /log info ("Host: " . $hostName);
    /log info ("IP: " . $ipAddress);
    /log info ("Gateway: " . $gateway);
    /log info ("DNS: " . $dnsServer);
    /log info ("DHCP: " . $enableDHCP);
}
```
{% endcode %}

***

## Secure input handling

### Password and sensitive data

{% code overflow="wrap" %}
```bash
# Secure password and sensitive data handling
/system script add name=secure-input-handling source={
    # Secure password validation
    :local validateSecurePassword do={
        :local password $1;
        :local minLength 12;
        :local maxLength 128;
        
        # Length validation
        :if ([:len $password] < $minLength) do={
            :return "Password too short (minimum " . $minLength . " characters)";
        };
        
        :if ([:len $password] > $maxLength) do={
            :return "Password too long (maximum " . $maxLength . " characters)";
        };
        
        # Complexity requirements
        :local hasUpper ($password ~ ".*[A-Z].*");
        :local hasLower ($password ~ ".*[a-z].*");
        :local hasDigit ($password ~ ".*[0-9].*");
        :local hasSpecial ($password ~ ".*[!@#\$%^&*(),.?\":{}|<>].*");
        
        :local missingRequirements {};
        :if (!$hasUpper) do={ :set missingRequirements ($missingRequirements, "uppercase letter"); };
        :if (!$hasLower) do={ :set missingRequirements ($missingRequirements, "lowercase letter"); };
        :if (!$hasDigit) do={ :set missingRequirements ($missingRequirements, "digit"); };
        :if (!$hasSpecial) do={ :set missingRequirements ($missingRequirements, "special character"); };
        
        :if ([:len $missingRequirements] > 0) do={
            :local errorMsg "Password missing: ";
            :foreach req in=$missingRequirements do={
                :set errorMsg ($errorMsg . $req . ", ");
            };
            :return $errorMsg;
        };
        
        # Check for common weak patterns
        :if ($password ~ ".*(123|abc|password|admin|qwerty).*") do={
            :return "Password contains common weak patterns";
        };
        
        :return "valid";
    };
    
    # Secure credential storage (limited in RouterOS)
    :local storeCredentials do={
        :local username $1;
        :local password $2;
        :local description $3;
        
        # Validate inputs
        :if ([:len $username] = 0 || [:len $password] = 0) do={
            /log error "Username and password are required";
            :return false;
        };
        
        # Create user with secure password
        :do {
            /user add name=$username password=$password group=read comment=$description;
            /log info ("User " . $username . " created successfully");
            
            # Clear password variable (security measure)
            :set password "";
            
            :return true;
        } on-error={
            /log error ("Failed to create user: " . $username);
            # Clear password variable even on error
            :set password "";
            :return false;
        };
    };
    
    # API key validation
    :local validateAPIKey do={
        :local apiKey $1;
        
        # API key format validation (example: 32-character hex)
        :if ([:len $apiKey] != 32) do={
            :return false;
        };
        
        # Check if it contains only valid hex characters
        :if (!($apiKey ~ "^[0-9a-fA-F]+\$")) do={
            :return false;
        };
        
        :return true;
    };
    
    # Certificate validation helper
    :local validateCertificateInput do={
        :local certData $1;
        
        # Basic PEM format validation
        :if (!($certData ~ "-----BEGIN.*CERTIFICATE-----")) do={
            :return false;
        };
        
        :if (!($certData ~ "-----END.*CERTIFICATE-----")) do={
            :return false;
        };
        
        # Check for reasonable length
        :if ([:len $certData] < 200 || [:len $certData] > 10000) do={
            :return false;
        };
        
        :return true;
    };
    
    # Test secure input functions
    :local testPassword "SecureP@ssw0rd123";
    :local weakPassword "password123";
    :local testAPIKey "1a2b3c4d5e6f7890abcdef1234567890";
    :local invalidAPIKey "invalid-key";
    
    :local strongResult [$validateSecurePassword $testPassword];
    :local weakResult [$validateSecurePassword $weakPassword];
    :local validAPIResult [$validateAPIKey $testAPIKey];
    :local invalidAPIResult [$validateAPIKey $invalidAPIKey];
    
    /log info ("Strong password validation: " . $strongResult);
    /log info ("Weak password validation: " . $weakResult);
    /log info ("Valid API key: " . $validAPIResult);
    /log info ("Invalid API key: " . $invalidAPIResult);
    
    # Example of secure user creation
    [$storeCredentials "testuser" $testPassword "Test user account"];
}
```
{% endcode %}

***

## Input processing workflows

### Multi-step configuration wizard

{% code overflow="wrap" %}
```bash
# Complete configuration wizard workflow
/system script add name=network-setup-wizard source={
    # Global state management for wizard
    :global wizardState;
    :global wizardStep;
    
    # Initialize wizard if first run
    :if ([:typeof $wizardState] = "nothing") do={
        :set wizardState {};
        :set wizardStep 1;
    };
    
    # Wizard step functions
    :local stepWelcome do={
        /log info "=== Network Configuration Wizard ===";
        /log info "This wizard will help you configure basic network settings.";
        /log info "Step 1: Basic System Information";
        /log info "Please set the following global variables:";
        /log info ":global systemHostname \"your-router-name\"";
        /log info ":global adminEmail \"admin@company.com\"";
        /log info "Then run the script again.";
    };
    
    :local stepBasicInfo do={
        :global systemHostname;
        :global adminEmail;
        
        :if ([:typeof $systemHostname] = "nothing" || [:typeof $adminEmail] = "nothing") do={
            /log error "Missing required information. Please set systemHostname and adminEmail.";
            :return false;
        };
        
        # Validate inputs
        :if ([:len $systemHostname] = 0 || [:len $systemHostname] > 63) do={
            /log error "Invalid hostname (1-63 characters required)";
            :return false;
        };
        
        :if (!($adminEmail ~ ".*@.*\\..*")) do={
            /log error "Invalid email format";
            :return false;
        };
        
        # Store validated data
        :set ($wizardState->"hostname") $systemHostname;
        :set ($wizardState->"email") $adminEmail;
        
        # Apply basic settings
        /system identity set name=$systemHostname;
        
        /log info "Step 1 completed. Basic information configured.";
        /log info "Step 2: Network Configuration";
        /log info "Please set: :global lanNetwork \"192.168.1.0/24\"";
        /log info "Please set: :global wanInterface \"ether1\"";
        
        :set systemHostname;  # Clear sensitive data
        :set adminEmail;
        :set wizardStep 2;
        :return true;
    };
    
    :local stepNetworkConfig do={
        :global lanNetwork;
        :global wanInterface;
        
        :if ([:typeof $lanNetwork] = "nothing" || [:typeof $wanInterface] = "nothing") do={
            /log error "Missing network configuration. Please set lanNetwork and wanInterface.";
            :return false;
        };
        
        # Validate network format
        :if (!($lanNetwork ~ "^[0-9.]+/[0-9]+\$")) do={
            /log error "Invalid network format (use CIDR notation like 192.168.1.0/24)";
            :return false;
        };
        
        # Store network configuration
        :set ($wizardState->"lan-network") $lanNetwork;
        :set ($wizardState->"wan-interface") $wanInterface;
        
        # Configure basic networking
        :do {
            # Extract IP and subnet from CIDR
            :local slashPos [:find $lanNetwork "/"];
            :local networkIP [:pick $lanNetwork 0 $slashPos];
            :local subnetBits [:pick $lanNetwork ($slashPos + 1) [:len $lanNetwork]];
            
            # Create bridge and add LAN interface
            /interface bridge add name=bridge-lan;
            /ip address add address=($networkIP . "/" . $subnetBits) interface=bridge-lan;
            
            /log info "Network configuration applied.";
            /log info "Step 3: Security Configuration";
            /log info "Please set: :global adminPassword \"your-secure-password\"";
            /log info "Please set: :global enableFirewall yes";
            
            :set lanNetwork;  # Clear variables
            :set wanInterface;
            :set wizardStep 3;
            :return true;
            
        } on-error={
            /log error "Failed to configure network settings";
            :return false;
        };
    };
    
    :local stepSecurity do={
        :global adminPassword;
        :global enableFirewall;
        
        :if ([:typeof $adminPassword] = "nothing") do={
            /log error "Missing security configuration. Please set adminPassword.";
            :return false;
        };
        
        # Validate password strength
        :if ([:len $adminPassword] < 8) do={
            /log error "Password too weak (minimum 8 characters)";
            :return false;
        };
        
        # Apply security settings
        :do {
            # Change admin password
            /user set admin password=$adminPassword;
            
            # Basic firewall rules if requested
            :if ($enableFirewall = "yes") do={
                /ip firewall filter add chain=input action=accept connection-state=established,related;
                /ip firewall filter add chain=input action=accept src-address=($wizardState->"lan-network");
                /ip firewall filter add chain=input action=drop;
                /ip firewall nat add chain=srcnat out-interface=($wizardState->"wan-interface") action=masquerade;
            };
            
            /log info "Security configuration completed.";
            /log info "=== Configuration Wizard Complete ===";
            
            # Generate summary
            :local summary "Configuration Summary:\n";
            :set summary ($summary . "Hostname: " . ($wizardState->"hostname") . "\n");
            :set summary ($summary . "LAN Network: " . ($wizardState->"lan-network") . "\n");
            :set summary ($summary . "WAN Interface: " . ($wizardState->"wan-interface") . "\n");
            :set summary ($summary . "Firewall: " . $enableFirewall . "\n");
            
            /log info $summary;
            
            # Clear wizard state
            :set wizardState;
            :set wizardStep;
            :set adminPassword "";  # Clear sensitive data
            :set enableFirewall;
            
            :return true;
            
        } on-error={
            /log error "Failed to apply security settings";
            :set adminPassword "";  # Clear sensitive data even on error
            :return false;
        };
    };
    
    # Execute appropriate step
    :if ($wizardStep = 1) do={
        [$stepWelcome];
        :if ([/log print count-only where message~"systemHostname"] = 0) do={
            [$stepBasicInfo];
        };
    } else={
        :if ($wizardStep = 2) do={
            :if ([$stepBasicInfo]) do={
                [$stepNetworkConfig];
            };
        } else={
            :if ($wizardStep = 3) do={
                [$stepSecurity];
            } else={
                /log error "Invalid wizard step";
                :set wizardState;
                :set wizardStep 1;
            };
        };
    };
}
```
{% endcode %}

***

## Error handling for user input

### Input validation errors

{% code overflow="wrap" %}
```bash
# Comprehensive input error handling
/system script add name=input-error-handling source={
    # Error reporting system
    :local reportError do={
        :local errorType $1;
        :local errorMessage $2;
        :local inputValue $3;
        
        :local timestamp [/system clock get time];
        :local fullMessage ("INPUT ERROR [" . $timestamp . "] " . $errorType . ": " . $errorMessage);
        
        :if ([:len $inputValue] > 0) do={
            :set fullMessage ($fullMessage . " (Input: '" . $inputValue . "')");
        };
        
        /log error $fullMessage;
        :return $fullMessage;
    };
    
    # Safe input processor with comprehensive error handling
    :local processUserInput do={
        :local inputType $1;
        :local inputValue $2;
        :local options $3;  # Optional parameters
        
        # Input type validation
        :if ($inputType = "ip-address") do={
            :if ([:len $inputValue] = 0) do={
                :return [$reportError "VALIDATION" "IP address cannot be empty" $inputValue];
            };
            
            # IP format validation
            :if (!($inputValue ~ "^[0-9.]+\$")) do={
                :return [$reportError "FORMAT" "Invalid IP address format" $inputValue];
            };
            
            # Validate octets
            :local octets {};
            :local currentOctet "";
            :local dotCount 0;
            
            :for i from=0 to=([:len $inputValue] - 1) do={
                :local char [:pick $inputValue $i ($i + 1)];
                :if ($char = ".") do={
                    :set octets ($octets, [:tonum $currentOctet]);
                    :set currentOctet "";
                    :set dotCount ($dotCount + 1);
                } else={
                    :set currentOctet ($currentOctet . $char);
                };
            };
            :set octets ($octets, [:tonum $currentOctet]);
            
            :if ($dotCount != 3 || [:len $octets] != 4) do={
                :return [$reportError "FORMAT" "IP address must have 4 octets" $inputValue];
            };
            
            # Validate octet ranges
            :foreach octet in=$octets do={
                :if ($octet < 0 || $octet > 255) do={
                    :return [$reportError "RANGE" "IP octet out of range (0-255)" $inputValue];
                };
            };
            
            :return "valid";
            
        } else={
            :if ($inputType = "port") do={
                :if ([:len $inputValue] = 0) do={
                    :return [$reportError "VALIDATION" "Port number cannot be empty" $inputValue];
                };
                
                :local portNum;
                :do {
                    :set portNum [:tonum $inputValue];
                } on-error={
                    :return [$reportError "FORMAT" "Port must be a number" $inputValue];
                };
                
                :if ($portNum < 1 || $portNum > 65535) do={
                    :return [$reportError "RANGE" "Port must be between 1-65535" $inputValue];
                };
                
                # Check for reserved ports if specified
                :if ([:typeof $options] != "nothing" && ($options->"check-reserved") = true) do={
                    :local reservedPorts {22; 23; 80; 443; 8291};
                    :foreach reserved in=$reservedPorts do={
                        :if ($portNum = $reserved) do={
                            :return [$reportError "CONFLICT" "Port is reserved for system use" $inputValue];
                        };
                    };
                };
                
                :return "valid";
                
            } else={
                :if ($inputType = "filename") do={
                    :if ([:len $inputValue] = 0) do={
                        :return [$reportError "VALIDATION" "Filename cannot be empty" $inputValue];
                    };
                    
                    :if ([:len $inputValue] > 255) do={
                        :return [$reportError "LENGTH" "Filename too long (max 255 characters)" $inputValue];
                    };
                    
                    # Check for invalid filename characters
                    :if ($inputValue ~ "[<>:\"/\\\\|?*]") do={
                        :return [$reportError "FORMAT" "Filename contains invalid characters" $inputValue];
                    };
                    
                    # Check for reserved names
                    :local reservedNames {"CON"; "PRN"; "AUX"; "NUL"};
                    :foreach reserved in=$reservedNames do={
                        :if ($inputValue = $reserved) do={
                            :return [$reportError "RESERVED" "Filename is a reserved name" $inputValue];
                        };
                    };
                    
                    :return "valid";
                    
                } else={
                    :return [$reportError "SYSTEM" "Unknown input type" $inputType];
                };
            };
        };
    };
    
    # Batch input validation
    :local validateInputBatch do={
        :local inputBatch $1;
        :local errors {};
        :local validInputs {};
        
        :foreach input in=$inputBatch do={
            :local inputType ($input->"type");
            :local inputValue ($input->"value");
            :local inputName ($input->"name");
            :local options ($input->"options");
            
            :local result [$processUserInput $inputType $inputValue $options];
            
            :if ($result = "valid") do={
                :set ($validInputs->$inputName) $inputValue;
                /log info ("Input '" . $inputName . "' validated successfully");
            } else={
                :set ($errors->$inputName) $result;
                /log error ("Input '" . $inputName . "' validation failed: " . $result);
            };
        };
        
        :if ([:len $errors] = 0) do={
            /log info "All inputs validated successfully";
            :return $validInputs;
        } else={
            /log error ("Validation failed for " . [:len $errors] . " inputs");
            :return $errors;
        };
    };
    
    # Test the error handling system
    :local testInputs {
        {name="server-ip"; type="ip-address"; value="192.168.1.300"};  # Invalid IP
        {name="ssh-port"; type="port"; value="22"; options={check-reserved=true}};  # Reserved port
        {name="backup-file"; type="filename"; value="backup/file.rsc"};  # Invalid filename
        {name="wan-ip"; type="ip-address"; value="203.0.113.1"};  # Valid IP
        {name="web-port"; type="port"; value="8080"}  # Valid port
    };
    
    :local results [$validateInputBatch $testInputs];
    
    # Report results
    :if ([:typeof ($results->"server-ip")] != "nothing") do={
        /log info "Validation results processed";
    } else={
        /log info "Some validations failed - check error log";
    };
}
```
{% endcode %}

***

## Best practices for user input

### Security guidelines

1. **Always validate input** - Never trust user input without validation
2. **Sanitize data** - Remove or escape dangerous characters
3. **Use type checking** - Ensure input matches expected data types
4. **Implement length limits** - Prevent buffer overflow scenarios
5. **Clear sensitive data** - Remove passwords and keys from memory after use

### Usability recommendations

1. **Provide clear prompts** - Tell users exactly what input is expected
2. **Give helpful error messages** - Explain what went wrong and how to fix it
3. **Use defaults wisely** - Provide sensible defaults for optional parameters
4. **Validate incrementally** - Check input as it's provided, not just at the end
5. **Confirm destructive actions** - Always confirm before making irreversible changes

### Performance considerations

1. **Cache validation results** - Don't re-validate the same input repeatedly
2. **Batch process inputs** - Validate multiple inputs together when possible
3. **Use efficient validation** - Implement validation logic that scales well
4. **Limit input processing** - Set reasonable limits on input size and complexity
5. **Handle errors gracefully** - Don't let input errors crash the entire script

