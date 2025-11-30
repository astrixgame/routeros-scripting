---
description: Complete RouterOS scripting language syntax reference covering variables, operators, control structures, and advanced language features.
icon: brackets-curly
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

# RouterOS Script Syntax

{% hint style="info" %}
RouterOS script language is a powerful scripting environment designed specifically for network device automation. Understanding the syntax is essential for creating effective automation scripts.
{% endhint %}

This comprehensive syntax reference covers all aspects of RouterOS scripting language, from basic variables to advanced control structures.

***

## Script structure and basics

### Script execution context

{% code overflow="wrap" %}
```bash
# Scripts can be executed in multiple ways:

# 1. Direct execution in terminal
{
    /log info "Direct execution";
    :local message "Hello World";
    /log info $message;
}

# 2. Stored script execution
/system script add name=my-script source={
    /log info "Stored script execution";
}
/system script run my-script

# 3. Scheduler execution
/system scheduler add name=scheduled-task interval=1h on-event={
    /log info "Scheduled execution";
}

# 4. Event-triggered execution (inline)
/ip firewall filter add chain=input action=log log-prefix="BLOCKED" \
    src-address-list=blocked-ips
```
{% endcode %}

### Comments and documentation

{% code overflow="wrap" %}
```bash
# Single-line comment - starts with hash symbol

# Multi-line documentation
# This script performs system monitoring
# Author: Network Administrator
# Version: 1.0
# Last modified: 2024-11-30

{
    # Inline comments within code blocks
    :local systemName [/system identity get name];  # Get router identity
    
    # Comments can appear anywhere on a line after code
    /log info $systemName;  # Log the system name
    
    # Block comments for complex explanations
    # The following section handles error conditions
    # It implements a retry mechanism with exponential backoff
    # Maximum retry attempts: 3
    :local retryCount 0;
}
```
{% endcode %}

***

## Variables and data types

### Variable declaration and naming

{% code overflow="wrap" %}
```bash
# Variable naming rules and examples
{
    # Valid variable names
    :local validName "correct";
    :local userName "john_doe";
    :local counter1 42;
    :local isActive true;
    :local _privateVar "underscore prefix";
    :local CamelCase "mixed case";
    
    # Global variables
    :global globalCounter 0;
    :global systemConfig "production";
    
    # Invalid names (commented out to prevent errors)
    # :local 123invalid "cannot start with number";
    # :local my-variable "hyphens not allowed";
    # :local class "class is reserved word";
    # :local if "if is reserved word";
}
```
{% endcode %}

### Data type examples

{% code overflow="wrap" %}
```bash
# Comprehensive data type examples
{
    # String data type
    :local stringVar "Hello World";
    :local emptyString "";
    :local stringWithQuotes "He said \"Hello\"";
    :local multiLineString "Line 1\nLine 2\nLine 3";
    :local pathString "/system/identity";
    
    # Numeric data type
    :local integerVar 42;
    :local negativeNum -15;
    :local largeNumber 1000000;
    :local hexNumber 0xFF;    # Hexadecimal
    :local octalNumber 0777;  # Octal
    
    # Boolean data type
    :local boolTrue true;
    :local boolFalse false;
    :local yesValue yes;      # Equivalent to true
    :local noValue no;        # Equivalent to false
    
    # IP address data type
    :local ipv4Address 192.168.1.1;
    :local ipv6Address 2001:db8::1;
    :local ipWithMask 192.168.1.0/24;
    :local ipRange 192.168.1.10-192.168.1.20;
    
    # Time data type
    :local timeValue 1d2h3m4s;
    :local seconds 30s;
    :local minutes 15m;
    :local hours 2h;
    :local days 7d;
    :local weeks 2w;
    
    # Array data type
    :local simpleArray {1; 2; 3; 4; 5};
    :local mixedArray {42; "text"; true; 192.168.1.1};
    :local emptyArray {};
    :local stringArray {"apple"; "banana"; "cherry"};
    
    # Associative array (dictionary-like)
    :local configDict {
        hostname="router1";
        ip=192.168.1.1;
        enabled=true;
        vlan=100
    };
    
    # Nested arrays
    :local nestedArray {
        {name="user1"; role="admin"};
        {name="user2"; role="user"};
        {name="user3"; role="guest"}
    };
    
    # Nothing type (null/undefined equivalent)
    :local undefinedVar;  # Type is "nothing"
    :local nullValue [:nothing];
}
```
{% endcode %}

***

## Operators and expressions

### Arithmetic operators

{% code overflow="wrap" %}
```bash
# Mathematical operations
{
    :local a 10;
    :local b 3;
    
    # Basic arithmetic
    :local sum ($a + $b);           # Addition: 13
    :local difference ($a - $b);    # Subtraction: 7  
    :local product ($a * $b);       # Multiplication: 30
    :local quotient ($a / $b);      # Division: 3 (integer division)
    :local remainder ($a % $b);     # Modulo: 1
    
    # Order of operations (PEMDAS/BODMAS)
    :local complex (($a + $b) * 2 - $a / 2);  # Result: 21
    :local withParens ((($a + $b) * 2) - ($a / 2));  # Explicit grouping
    
    # Working with negative numbers
    :local negative (-$a);          # Unary minus: -10
    :local subtractNegative ($a - (-$b));  # Double negative: 13
    
    /log info ("Sum: " . $sum . ", Product: " . $product);
}
```
{% endcode %}

### Comparison operators

{% code overflow="wrap" %}
```bash
# Comparison operations
{
    :local x 10;
    :local y 20;
    :local str1 "apple";
    :local str2 "banana";
    
    # Numeric comparisons
    :local isEqual ($x = $y);         # false (equality)
    :local notEqual ($x != $y);       # true (inequality)  
    :local lessThan ($x < $y);        # true
    :local lessEqual ($x <= $y);      # true
    :local greaterThan ($x > $y);     # false
    :local greaterEqual ($x >= $y);   # false
    
    # String comparisons
    :local strEqual ($str1 = $str2);     # false
    :local strNotEqual ($str1 != $str2); # true
    :local strLess ($str1 < $str2);      # true (alphabetical order)
    
    # Pattern matching with ~ operator
    :local matchResult ($str1 ~ "app.*");     # true (regex-like matching)
    :local noMatch ($str1 ~ "ban.*");         # false
    
    # IP address comparisons
    :local ip1 192.168.1.1;
    :local ip2 192.168.1.100;
    :local network 192.168.1.0/24;
    
    :local ipEqual ($ip1 = ip2);              # false
    :local inNetwork ($ip1 in $network);      # true (IP in subnet)
    
    /log info ("x < y: " . $lessThan . ", str1 < str2: " . $strLess);
}
```
{% endcode %}

### Logical operators

{% code overflow="wrap" %}
```bash
# Boolean logic operations
{
    :local a true;
    :local b false;
    :local c true;
    
    # Basic logical operators
    :local andResult ($a && $b);      # false (logical AND)
    :local orResult ($a || $b);       # true (logical OR)
    :local notResult (!$a);           # false (logical NOT)
    
    # Complex logical expressions
    :local complexAnd ($a && $b && $c);       # false
    :local complexOr ($a || $b || $c);        # true
    :local mixedLogic (($a && $c) || $b);     # true
    :local deMorgan (!($a && $b));            # true (!false = true)
    
    # Short-circuit evaluation
    :local shortCircuit ($b && $a);   # false (b is false, so $a not evaluated)
    
    # Combining with comparisons
    :local num 15;
    :local inRange ($num > 10 && $num < 20);  # true
    :local outOfRange ($num < 5 || $num > 25); # false
    
    /log info ("AND result: " . $andResult . ", OR result: " . $orResult);
}
```
{% endcode %}

### String operators

{% code overflow="wrap" %}
```bash
# String manipulation and operations
{
    :local firstName "John";
    :local lastName "Doe";
    :local separator " ";
    
    # String concatenation with . operator
    :local fullName ($firstName . $separator . $lastName);  # "John Doe"
    :local greeting ("Hello, " . $fullName . "!");         # "Hello, John Doe!"
    
    # Multiple concatenations
    :local address ($firstName . " " . $lastName . " lives in " . "New York");
    
    # Concatenating different types (automatic conversion)
    :local age 25;
    :local description ($firstName . " is " . $age . " years old");
    
    # Empty string handling
    :local empty "";
    :local notEmpty ("Value: " . $empty . "END");  # "Value: END"
    
    # Special characters in strings
    :local withQuotes ("He said \"Hello\"");
    :local withNewlines ("Line 1\nLine 2\nLine 3");
    :local withTabs ("Column1\tColumn2\tColumn3");
    
    # String comparison with concatenation
    :local prefix "Hello";
    :local suffix "World";
    :local combined ($prefix . " " . $suffix);
    :local matches ($combined = "Hello World");  # true
    
    /log info ("Full name: " . $fullName);
    /log info ("Description: " . $description);
}
```
{% endcode %}

***

## Control structures

### Conditional statements

{% code overflow="wrap" %}
```bash
# Comprehensive if-else examples
{
    :local temperature 25;
    :local humidity 60;
    :local userRole "admin";
    
    # Basic if statement
    :if ($temperature > 30) do={
        /log warning "High temperature detected";
    };
    
    # If-else statement
    :if ($temperature > 30) do={
        /log warning "Temperature too high";
    } else={
        /log info "Temperature normal";
    };
    
    # Nested if-else (else-if simulation)
    :if ($temperature > 35) do={
        /log error "Critical temperature";
    } else={
        :if ($temperature > 30) do={
            /log warning "High temperature";
        } else={
            :if ($temperature < 10) do={
                /log warning "Low temperature";
            } else={
                /log info "Normal temperature";
            };
        };
    };
    
    # Multiple conditions with logical operators
    :if ($temperature > 25 && $humidity > 70) do={
        /log info "Hot and humid conditions";
    } else={
        :if ($temperature > 25 || $humidity > 70) do={
            /log info "Either hot or humid";
        } else={
            /log info "Comfortable conditions";
        };
    };
    
    # String-based conditions
    :if ($userRole = "admin") do={
        /log info "Administrator access granted";
    } else={
        :if ($userRole = "user") do={
            /log info "User access granted";
        } else={
            :if ($userRole = "guest") do={
                /log info "Guest access granted";
            } else={
                /log error "Unknown user role";
            };
        };
    };
    
    # Complex conditional with multiple variables
    :local systemLoad 75;
    :local memoryUsage 80;
    :local diskSpace 90;
    
    :if ($systemLoad > 80 && $memoryUsage > 85 && $diskSpace > 95) do={
        /log error "System critical - all resources high";
    } else={
        :if ($systemLoad > 70 || $memoryUsage > 75 || $diskSpace > 85) do={
            /log warning "System under stress";
        } else={
            /log info "System resources normal";
        };
    };
}
```
{% endcode %}

### Loop structures

{% code overflow="wrap" %}
```bash
# Various loop implementations
{
    # Basic foreach with array
    :local numbers {1; 2; 3; 4; 5};
    :foreach number in=$numbers do={
        :local squared ($number * $number);
        /log info ("Number: " . $number . ", Squared: " . $squared);
    };
    
    # Foreach with string array
    :local fruits {"apple"; "banana"; "cherry"; "date"};
    :foreach fruit in=$fruits do={
        :local message ("Processing fruit: " . $fruit);
        /log info $message;
    };
    
    # Foreach with command results
    :foreach interface in=[/interface print as-value] do={
        :local ifName ($interface->"name");
        :local ifType ($interface->"type");
        :local isRunning ($interface->"running");
        
        :if ($isRunning) do={
            /log info ("Interface " . $ifName . " (" . $ifType . ") is UP");
        } else={
            /log warning ("Interface " . $ifName . " (" . $ifType . ") is DOWN");
        };
    };
    
    # Nested foreach loops
    :local departments {"IT"; "Sales"; "Marketing"};
    :local priorities {"High"; "Medium"; "Low"};
    
    :foreach dept in=$departments do={
        :foreach priority in=$priorities do={
            :local task ($dept . " - " . $priority . " Priority");
            /log info ("Task: " . $task);
        };
    };
    
    # While loop simulation using recursion and counters
    :local counter 0;
    :local maxCount 5;
    
    # While loop equivalent
    :while ($counter < $maxCount) do={
        /log info ("Counter value: " . $counter);
        :set counter ($counter + 1);
        
        # Break condition simulation
        :if ($counter >= 3) do={
            /log info "Breaking early at counter 3";
            :set counter $maxCount;  # Force exit
        };
    };
    
    # For loop simulation with range
    :for i from=1 to=10 step=2 do={
        :local message ("Odd number: " . $i);
        /log info $message;
    };
    
    # Countdown loop
    :for i from=10 to=1 step=-1 do={
        /log info ("Countdown: " . $i);
        :if ($i = 1) do={
            /log info "Blast off!";
        };
    };
}
```
{% endcode %}

***

## Functions and procedures

### Built-in function usage

{% code overflow="wrap" %}
```bash
# Comprehensive built-in function examples
{
    # String manipulation functions
    :local originalText "Hello World RouterOS";
    
    # Length function
    :local textLength [:len $originalText];         # 19
    /log info ("Text length: " . $textLength);
    
    # Substring extraction with :pick
    :local greeting [:pick $originalText 0 5];      # "Hello"
    :local world [:pick $originalText 6 11];        # "World"
    :local routeros [:pick $originalText 12 20];    # "RouterOS"
    
    # Find substring position
    :local worldPos [:find $originalText "World"];   # 6
    :local osPos [:find $originalText "OS"];         # 17
    :local notFound [:find $originalText "Linux"];   # Returns nothing/empty
    
    # String replacement
    :local replaced [:replace $originalText "World" "Universe"];
    /log info ("Replaced: " . $replaced);  # "Hello Universe RouterOS"
    
    # Type conversion functions
    :local numberString "42";
    :local stringNumber [:tonum $numberString];      # 42 (number)
    :local backToString [:tostr $stringNumber];      # "42" (string)
    
    # IP address conversion
    :local ipString "192.168.1.1";
    :local ipAddress [:toip $ipString];              # IP type
    :local ipBack [:tostr $ipAddress];               # "192.168.1.1"
    
    # Time conversion
    :local timeString "1d2h3m4s";
    :local timeValue [:totime $timeString];          # Time type
    :local timeBack [:tostr $timeValue];             # "1d2h3m4s"
    
    # Array functions
    :local testArray {"one"; "two"; "three"; "four"; "five"};
    :local arrayLen [:len $testArray];               # 5
    :local firstTwo [:pick $testArray 0 2];         # {"one"; "two"}
    :local lastTwo [:pick $testArray 3 5];          # {"four"; "five"}
    
    # Mathematical functions (if available)
    :local angle 45;
    :local result [:sin $angle];                     # Sine of 45
    :local logResult [:log 100];                     # Logarithm of 100
    
    /log info ("Original: " . $originalText);
    /log info ("Greeting: " . $greeting . ", World at position: " . $worldPos);
}
```
{% endcode %}

### Custom function definition

{% code overflow="wrap" %}
```bash
# Custom function examples and patterns
{
    # Simple function with single parameter
    :local greetUser do={
        :local userName $1;
        :local greeting ("Hello, " . $userName . "!");
        :return $greeting;
    };
    
    # Usage of simple function
    :local message [$greetUser "Alice"];
    /log info $message;  # "Hello, Alice!"
    
    # Function with multiple parameters
    :local calculateArea do={
        :local width $1;
        :local height $2;
        :local area ($width * $height);
        :return $area;
    };
    
    # Usage with multiple parameters
    :local roomArea [$calculateArea 10 12];
    /log info ("Room area: " . $roomArea . " square meters");
    
    # Function with parameter validation
    :local safeDivision do={
        :local numerator $1;
        :local denominator $2;
        
        # Input validation
        :if ($denominator = 0) do={
            /log error "Division by zero attempted";
            :return "ERROR";
        };
        
        :local result ($numerator / $denominator);
        :return $result;
    };
    
    # Test the safe division function
    :local validResult [$safeDivision 10 2];        # 5
    :local errorResult [$safeDivision 10 0];        # "ERROR"
    
    # Complex function with multiple return values (using array)
    :local analyzeSystem do={
        :local sysInfo [/system resource get];
        :local cpu ($sysInfo->"cpu-load");
        :local memory ($sysInfo->"free-memory");
        :local uptime ($sysInfo->"uptime");
        
        # Return multiple values as array
        :local results {
            cpu=$cpu;
            memory=$memory;
            uptime=$uptime;
            status="normal"
        };
        
        # Determine status based on values
        :if ($cpu > 80) do={
            :set ($results->"status") "high-cpu";
        };
        
        :return $results;
    };
    
    # Use complex function
    :local systemStatus [$analyzeSystem];
    /log info ("System status: " . ($systemStatus->"status"));
    /log info ("CPU load: " . ($systemStatus->"cpu") . "%");
    
    # Recursive function example
    :local factorial do={
        :local n $1;
        
        # Base case
        :if ($n <= 1) do={
            :return 1;
        };
        
        # Recursive case
        :local previousResult [$factorial ($n - 1)];
        :local result ($n * $previousResult);
        :return $result;
    };
    
    # Test recursive function
    :local fact5 [$factorial 5];  # 120
    /log info ("5! = " . $fact5);
    
    # Function with local variables and complex logic
    :local formatFileSize do={
        :local sizeInBytes $1;
        :local units {"B"; "KB"; "MB"; "GB"; "TB"};
        :local unitIndex 0;
        :local size $sizeInBytes;
        
        # Convert to appropriate unit
        :while ($size >= 1024 && $unitIndex < ([:len $units] - 1)) do={
            :set size ($size / 1024);
            :set unitIndex ($unitIndex + 1);
        };
        
        :local unit [:pick $units $unitIndex];
        :local formatted ($size . " " . $unit);
        :return $formatted;
    };
    
    # Test file size formatting
    :local largeFile [$formatFileSize 1073741824];  # "1 GB"
    /log info ("File size: " . $largeFile);
}
```
{% endcode %}

***

## Array and data structure manipulation

### Array operations

{% code overflow="wrap" %}
```bash
# Comprehensive array manipulation
{
    # Array creation and initialization
    :local emptyArray {};
    :local numberArray {1; 2; 3; 4; 5};
    :local stringArray {"red"; "green"; "blue"};
    :local mixedArray {42; "text"; true; 192.168.1.1};
    
    # Array access and modification
    :local firstNumber [:pick $numberArray 0];       # 1
    :local lastNumber [:pick $numberArray 4];        # 5
    :local middleNumbers [:pick $numberArray 1 4];   # {2; 3; 4}
    
    # Array length
    :local arrayLength [:len $numberArray];          # 5
    /log info ("Array has " . $arrayLength . " elements");
    
    # Array concatenation (simulated)
    :local array1 {1; 2; 3};
    :local array2 {4; 5; 6};
    :local combined {};
    
    # Combine arrays using foreach
    :foreach item in=$array1 do={
        :set combined ($combined, $item);
    };
    :foreach item in=$array2 do={
        :set combined ($combined, $item);
    };
    
    # Array searching
    :local searchArray {"apple"; "banana"; "cherry"; "date"};
    :local found false;
    :local searchTerm "cherry";
    
    :foreach item in=$searchArray do={
        :if ($item = $searchTerm) do={
            :set found true;
            /log info ("Found " . $searchTerm . " in array");
        };
    };
    
    # Array filtering (create new array with matching elements)
    :local numbers {1; 2; 3; 4; 5; 6; 7; 8; 9; 10};
    :local evenNumbers {};
    
    :foreach number in=$numbers do={
        :if (($number % 2) = 0) do={
            :set evenNumbers ($evenNumbers, $number);
        };
    };
    
    /log info ("Even numbers: " . [:tostr $evenNumbers]);
    
    # Multi-dimensional arrays
    :local matrix {
        {1; 2; 3};
        {4; 5; 6};
        {7; 8; 9}
    };
    
    # Access multi-dimensional array elements
    :local firstRow [:pick $matrix 0];               # {1; 2; 3}
    :local middleElement [:pick [:pick $matrix 1] 1]; # 5
    
    # Array of objects (associative arrays)
    :local users {
        {name="John"; age=30; role="admin"};
        {name="Jane"; age=25; role="user"};
        {name="Bob"; age=35; role="guest"}
    };
    
    # Process array of objects
    :foreach user in=$users do={
        :local userName ($user->"name");
        :local userAge ($user->"age");
        :local userRole ($user->"role");
        /log info ("User: " . $userName . ", Age: " . $userAge . ", Role: " . $userRole);
    };
}
```
{% endcode %}

### Associative arrays (dictionaries)

{% code overflow="wrap" %}
```bash
# Dictionary/associative array operations
{
    # Create associative array (dictionary)
    :local config {
        hostname="router1";
        ip="192.168.1.1";
        mask=24;
        gateway="192.168.1.254";
        dns1="8.8.8.8";
        dns2="8.8.4.4";
        enabled=true;
        vlan=100
    };
    
    # Access dictionary values
    :local routerName ($config->"hostname");
    :local ipAddress ($config->"ip");
    :local isEnabled ($config->"enabled");
    
    /log info ("Router: " . $routerName . " at " . $ipAddress);
    
    # Modify dictionary values
    :set ($config->"hostname") "new-router";
    :set ($config->"enabled") false;
    
    # Add new key-value pairs
    :set ($config->"location") "DataCenter1";
    :set ($config->"contact") "admin@company.com";
    
    # Check if key exists (indirect method)
    :local keyExists false;
    :do {
        :local testValue ($config->"nonexistent");
        :set keyExists true;
    } on-error={
        :set keyExists false;
    };
    
    # Iterate through dictionary
    :foreach key,value in=$config do={
        /log info ("Key: " . $key . ", Value: " . [:tostr $value]);
    };
    
    # Nested dictionaries
    :local networkConfig {
        lan={
            interface="bridge1";
            ip="192.168.1.1/24";
            dhcp=true
        };
        wan={
            interface="ether1";
            type="dhcp-client";
            backup="ether2"
        };
        wifi={
            ssid="MyNetwork";
            password="SecurePass123";
            channel=36;
            enabled=true
        }
    };
    
    # Access nested dictionary values
    :local wifiSSID (($networkConfig->"wifi")->"ssid");
    :local lanInterface (($networkConfig->"lan")->"interface");
    :local wanType (($networkConfig->"wan")->"type");
    
    /log info ("WiFi SSID: " . $wifiSSID);
    /log info ("LAN Interface: " . $lanInterface);
    /log info ("WAN Type: " . $wanType);
    
    # Complex data structure example
    :local inventory {
        switches={
            count=5;
            models={"CRS328"; "CRS326"; "CRS312"};
            locations={"Floor1"; "Floor2"; "DataCenter"}
        };
        routers={
            count=3;
            models={"hAP ax3"; "CCR2004"; "RB5009"};
            firmware="7.16.1"
        };
        accessories={
            cables=50;
            adapters=20;
            licenses=10
        }
    };
    
    # Process complex data structure
    :local switchCount (($inventory->"switches")->"count");
    :local routerModels (($inventory->"routers")->"models");
    :local cableCount (($inventory->"accessories")->"cables");
    
    /log info ("Total switches: " . $switchCount);
    /log info ("Available cables: " . $cableCount);
    
    # Iterate through router models
    :foreach model in=$routerModels do={
        /log info ("Router model in inventory: " . $model);
    };
}
```
{% endcode %}

***

## Error handling and flow control

### Exception handling

{% code overflow="wrap" %}
```bash
# Comprehensive error handling examples
{
    # Basic try-catch equivalent
    :do {
        # Potentially risky operation
        /interface disable ether999;  # Interface might not exist
        /log info "Interface disabled successfully";
    } on-error={
        /log error "Failed to disable interface - interface may not exist";
    };
    
    # Error handling with error message capture (simulated)
    :local operationSuccess false;
    :local errorMessage "";
    
    :do {
        # Another risky operation
        /ip address add address=192.168.1.1/24 interface=ether1;
        :set operationSuccess true;
    } on-error={
        :set operationSuccess false;
        :set errorMessage "Failed to add IP address";
    };
    
    # React based on operation result
    :if ($operationSuccess) do={
        /log info "IP address configured successfully";
    } else={
        /log error $errorMessage;
        # Implement fallback procedure
        /log info "Trying alternative configuration...";
    };
    
    # Nested error handling
    :do {
        # Outer operation
        /log info "Starting complex operation";
        
        :do {
            # Inner operation that might fail
            /system reboot;
        } on-error={
            /log warning "Reboot failed, trying shutdown";
            
            :do {
                /system shutdown;
            } on-error={
                /log error "System shutdown also failed";
            };
        };
        
    } on-error={
        /log error "Entire operation sequence failed";
    };
    
    # Error handling with retry logic
    :local maxRetries 3;
    :local retryCount 0;
    :local success false;
    
    :while ($retryCount < $maxRetries && !$success) do={
        :do {
            # Operation that might fail
            /tool fetch url="http://example.com/test" dst-path="test.txt";
            :set success true;
            /log info "Download completed successfully";
        } on-error={
            :set retryCount ($retryCount + 1);
            /log warning ("Download failed, attempt " . $retryCount . " of " . $maxRetries);
            
            :if ($retryCount < $maxRetries) do={
                /log info "Retrying in 5 seconds...";
                :delay 5s;
            } else={
                /log error "All retry attempts failed";
            };
        };
    };
    
    # Error handling with different error types (simulated)
    :local handleNetworkOperation do={
        :local operation $1;
        :local target $2;
        
        :if ($operation = "ping") do={
            :do {
                /tool ping address=$target count=3;
                /log info ("Ping to " . $target . " successful");
            } on-error={
                /log warning ("Ping to " . $target . " failed - host unreachable");
            };
        } else={
            :if ($operation = "fetch") do={
                :do {
                    /tool fetch url=$target;
                    /log info ("Fetch from " . $target . " successful");
                } on-error={
                    /log warning ("Fetch from " . $target . " failed - connection error");
                };
            } else={
                /log error ("Unknown operation: " . $operation);
            };
        };
    };
    
    # Use the error handling function
    [$handleNetworkOperation "ping" "8.8.8.8"];
    [$handleNetworkOperation "fetch" "http://example.com"];
    [$handleNetworkOperation "invalid" "target"];
}
```
{% endcode %}

***

## Advanced syntax features

### Pattern matching and regular expressions

{% code overflow="wrap" %}
```bash
# Pattern matching and text processing
{
    # Basic pattern matching with ~ operator
    :local text "Hello World RouterOS";
    :local hasHello ($text ~ "Hello");           # true
    :local hasRouter ($text ~ "Router");         # true  
    :local hasLinux ($text ~ "Linux");           # false
    
    # Case-insensitive matching (RouterOS specific)
    :local textLower "hello world routeros";
    :local matchCase ($textLower ~ "Hello");     # false (case sensitive)
    
    # Wildcard patterns (simplified regex-like)
    :local filename "config.backup.rsc";
    :local isBackup ($filename ~ ".*backup.*");  # true
    :local isConfig ($filename ~ "config.*");    # true
    :local isScript ($filename ~ ".*\\.rsc");    # true
    
    # IP address pattern matching
    :local ipAddr "192.168.1.100";
    :local isPrivateA ($ipAddr ~ "^10\\..*");           # false
    :local isPrivateB ($ipAddr ~ "^172\\.(1[6-9]|2[0-9]|3[0-1])\\..*"); # false
    :local isPrivateC ($ipAddr ~ "^192\\.168\\..*");    # true
    
    # MAC address validation pattern
    :local macAddr "00:11:22:AA:BB:CC";
    :local validMAC ($macAddr ~ "^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$");
    
    # Log file pattern matching
    :local logEntry "2024-11-30 10:15:30 [ERROR] Connection failed";
    :local isError ($logEntry ~ ".*\\[ERROR\\].*");     # true
    :local isWarning ($logEntry ~ ".*\\[WARNING\\].*"); # false
    
    # Extract information using pattern matching
    :local interfaceList [/interface print as-value];
    :foreach interface in=$interfaceList do={
        :local ifName ($interface->"name");
        
        # Check interface type by name pattern
        :if ($ifName ~ "^ether.*") do={
            /log info ($ifName . " is an Ethernet interface");
        } else={
            :if ($ifName ~ "^wlan.*") do={
                /log info ($ifName . " is a Wireless interface");
            } else={
                :if ($ifName ~ "^bridge.*") do={
                    /log info ($ifName . " is a Bridge interface");
                } else={
                    /log info ($ifName . " is another type of interface");
                };
            };
        };
    };
    
    # Pattern-based string replacement (using find and replace functions)
    :local inputText "The router IP is 192.168.1.1 and gateway is 192.168.1.254";
    :local processedText $inputText;
    
    # Simple find and replace (since RouterOS doesn't have full regex replace)
    :if ($processedText ~ "192\\.168\\.1\\.") do={
        /log info "Found private IP addresses in text";
        # Manual replacement logic would go here
    };
    
    /log info ("Pattern matching results:");
    /log info ("Has 'Hello': " . $hasHello);
    /log info ("Is backup file: " . $isBackup);
    /log info ("Is private IP: " . $isPrivateC);
}
```
{% endcode %}

### Dynamic command execution

{% code overflow="wrap" %}
```bash
# Dynamic command construction and execution
{
    # Dynamic command building
    :local commandParts {"/interface"; "set"; "ether1"; "comment=\"Dynamic comment\""};
    :local fullCommand "";
    
    # Build command string
    :foreach part in=$commandParts do={
        :set fullCommand ($fullCommand . $part . " ");
    };
    
    /log info ("Would execute: " . $fullCommand);
    
    # Dynamic interface configuration
    :local configureInterface do={
        :local interfaceName $1;
        :local comment $2;
        :local enabled $3;
        
        # Build and execute command dynamically
        :if ($enabled) do={
            /interface set $interfaceName comment=$comment disabled=no;
        } else={
            /interface set $interfaceName comment=$comment disabled=yes;
        };
        
        /log info ("Configured interface " . $interfaceName);
    };
    
    # Use dynamic function
    [$configureInterface "ether1" "WAN Interface" true];
    [$configureInterface "ether2" "LAN Interface" false];
    
    # Dynamic rule creation based on conditions
    :local createFirewallRule do={
        :local chain $1;
        :local srcAddress $2;
        :local action $3;
        :local comment $4;
        
        # Dynamic rule creation
        /ip firewall filter add chain=$chain src-address=$srcAddress action=$action comment=$comment;
        /log info ("Created firewall rule: " . $chain . " " . $srcAddress . " -> " . $action);
    };
    
    # Create rules dynamically
    :local blockedIPs {"192.168.100.50"; "10.0.0.100"; "172.16.1.200"};
    :foreach blockedIP in=$blockedIPs do={
        [$createFirewallRule "input" $blockedIP "drop" ("Block suspicious IP: " . $blockedIP)];
    };
    
    # Dynamic VLAN creation
    :local createVLAN do={
        :local vlanId $1;
        :local vlanName $2;
        :local bridgeInterface $3;
        
        # Create VLAN interface
        /interface vlan add name=$vlanName vlan-id=$vlanId interface=$bridgeInterface;
        
        # Configure IP address
        :local ipAddress ("192.168." . $vlanId . ".1/24");
        /ip address add address=$ipAddress interface=$vlanName;
        
        /log info ("Created VLAN " . $vlanId . " with name " . $vlanName);
    };
    
    # Create multiple VLANs
    :local vlanConfigs {
        {id=10; name="VLAN10-Management"};
        {id=20; name="VLAN20-Users"};
        {id=30; name="VLAN30-Guest"}
    };
    
    :foreach vlanConfig in=$vlanConfigs do={
        :local vlanId ($vlanConfig->"id");
        :local vlanName ($vlanConfig->"name");
        [$createVLAN $vlanId $vlanName "bridge1"];
    };
    
    # Conditional command execution
    :local systemInfo [/system resource get];
    :local freeMemory ($systemInfo->"free-memory");
    :local totalMemory ($systemInfo->"total-memory");
    :local memoryUsage ((($totalMemory - $freeMemory) * 100) / $totalMemory);
    
    # Execute different commands based on system state
    :if ($memoryUsage > 90) do={
        /log error "Critical memory usage - clearing logs";
        /log clear;
    } else={
        :if ($memoryUsage > 75) do={
            /log warning "High memory usage - monitoring enabled";
            # Enable additional monitoring
        } else={
            /log info "Memory usage normal";
        };
    };
}
```
{% endcode %}

This comprehensive syntax reference covers all major aspects of RouterOS scripting language, providing the foundation for creating powerful automation scripts.

