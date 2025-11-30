# RouterOS scripts and templates
Collection of Jinja2 interactive templates and RouterOS RSC scripts for configuration automation. 
Includes modular Bash scripts for user-driven generation of MikroTik configs, 
focusing on scalability, reusability, and rapid deployment.


## 🛠️ Install jinja2
To install jinja you'll need python with pip, you can install it using:
```bash
sudo apt update
sudo apt install python3-pip
```
And then install the jinja using:
```bash
pip3 install jinja2-cli
```


## 🧑🏻‍💻 Usage

### DHCP Templates

#### DHCP Server (templates/dhcp/server.j2)
Configures a DHCP server with IP pool, DNS, and optional NTP settings.

Variables:
- `pool_name`: Name of the IP pool (default: 'dhcp-pool')
- `pool_range`: IP range for DHCP clients (default: '192.168.88.100-192.168.88.200')
- `interface`: Interface to run DHCP server on (default: 'bridge1')
- `server_name`: DHCP server name (default: 'dhcp1')
- `network`: Network address (default: '192.168.88.0/24')
- `gateway`: Gateway IP (default: '192.168.88.1')
- `dns1`, `dns2`: DNS servers (default: '8.8.8.8', '8.8.4.4')
- `domain`: Optional domain name
- `ntp`: Enable NTP configuration (optional)
- `primary_ntp`, `secondary_ntp`: NTP servers if enabled

#### DHCP Client (templates/dhcp/client.j2)
Sets up DHCP client for WAN interfaces with connection monitoring.

Variables:
- `interface`: WAN interface name (default: 'ether1')
- `use_peer_dns`: Use DNS from DHCP server (default: 'yes')
- `use_peer_ntp`: Use NTP from DHCP server (default: 'yes')
- `add_default_route`: Add default route (default: 'yes')
- `hostname`: Router hostname (optional)
- `monitor_host`: Host to monitor (default: '8.8.8.8')
- `monitor_interval`: Check interval (default: '30s')

### Failover Templates

#### Dual WAN Failover (templates/failover/dual-wan.j2)
Implements failover between two WAN connections with optional load balancing.

Variables:
- `primary_gateway`: Primary WAN gateway IP
- `secondary_gateway`: Secondary WAN gateway IP
- `primary_interface`: Primary WAN interface (default: 'ether1')
- `secondary_interface`: Secondary WAN interface (default: 'ether2')
- `load_balance`: Enable load balancing (default: false)
- `monitor_host`: Host to monitor (default: '8.8.8.8')
- `monitor_interval`: Check interval (default: '30s')

#### WAN with LTE Backup (templates/failover/wan-lte.j2)
Configures WAN failover with LTE backup connection.

Variables:
- `wan_gateway`: Primary WAN gateway IP
- `use_sim_config`: Configure SIM settings (default: false)
- `apn`: APN for LTE (if use_sim_config=true)
- `username`: SIM username (optional)
- `password`: SIM password (optional)
- `monitor_host`: Host to monitor (default: '8.8.8.8')
- `check_interval`: Failover check interval (default: '1m')

### OpenVPN Templates
Located in templates/openvpn/:
- `certificates.j2`: SSL certificate configuration
- `server.j2`: OpenVPN server setup
- `profiles.j2`: VPN profiles configuration
- `secrets.j2`: User authentication settings

### PPPoE Templates

#### CETIN PPPoE (templates/pppoe/cetin.j2)
Configures PPPoE client for CETIN ISP with VLAN support.

Variables:
- `interface`: Physical interface (default: 'ether1')
- `username`: PPPoE username (default: 'pppoe')
- `password`: PPPoE password (default: 'pppoe')
- `dns1`, `dns2`: DNS servers (default: '8.8.8.8', '8.8.4.4')

### Example Usage

To generate a configuration file from a template:
```bash
jinja2 template.j2 -D variable=value -D variable2=value2 > output.rsc
```

Multiple variables can be passed using the -D flag for each variable. The generated .rsc file can then be imported into RouterOS.


## 🛠️ Tech Stack
- [RouterOS scripts](https://help.mikrotik.com/docs/spaces/ROS/pages/47579229/Scripting)
- [Jinja2](https://jinja.palletsprojects.com/en/stable/)
- [Bash](https://www.gnu.org/software/bash/)


## ❤️ Support  
A simple star to this project repo is enough to keep me motivated. If you find your self very much excited with this project let me know with a tweet.


## ➤ License
Distributed under the MIT  License. See [LICENSE](LICENSE) for more information.
