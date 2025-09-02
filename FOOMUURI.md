# Ansible Role: Foomuuri

An Ansible role for managing [Foomuuri](https://github.com/FoobarOy/foomuuri), a multizone bidirectional nftables firewall.

## Description

Foomuuri is a modern firewall based on nftables that supports multiple zones. This Ansible role automates the installation and configuration of Foomuuri on Debian/Ubuntu systems.

## Requirements

- Ansible 2.9 or higher
- Debian 11+, Ubuntu 20.04+, or Arch Linux
- Root or sudo access on the target hosts
- For Arch Linux: `base-devel` and `git` packages for AUR builds

## Role Variables

### Basic Configuration

```yaml
# Installation method - automatically determined based on OS version
foomuuri_install_method: "auto"  # auto, package, aur, github

# For Arch Linux: AUR helper (optional)
foomuuri_aur_helper: "yay"  # yay, paru, trizen, makepkg

# Global configuration (optional - use only if deviating from defaults)
foomuuri_config:
  log_input: "yes"
  log_output: "no"
  rpfilter: "yes"

# Zones - minimal configuration
foomuuri_zones:
  - name: "public"
    interfaces:
      - "{{ ansible_default_ipv4.interface }}"
```

### Custom Macros

Foomuuri supports flexible macro definitions for services, IP addresses, and combinations:

#### New Macro Syntax

```yaml
# Custom macros
foomuuri_macros:
  # Services with new protocol mapping syntax
  - name: "checkmk-agent"
    protocols:
      tcp: [6556]

  - name: "observium-xinetd"
    protocols:
      tcp: [36602]

  # Multiple protocols with different ports
  - name: "dns-services"
    protocols:
      tcp: [53]
      udp: [53]

  - name: "mail-services"
    protocols:
      tcp: [143, 587, 993, 995, 9001]

  # Mixed protocol example
  - name: "netbios"
    protocols:
      tcp: [139, 445]
      udp: [137, 138]

  # IP addresses, subnets and ranges
  - name: "public-checkmk-ips"
    ips: ["83.247.109.32", "83.247.109.43", "212.45.32.36", "212.45.32.43"]

  - name: "management-networks"
    ips: ["10.6.109.0/24", "10.4.32.0/24", "192.168.1.0/24"]

  # IPv6 support
  - name: "ipv6-monitoring"
    ips: ["2001:9e0:5:32::43", "2001:9e0:4:32::43"]

  # Combining existing macros (references built-in or custom macros)
  - name: "checkmk-services"
    use_macros: ["checkmk-agent", "ping", "snmp", "ssh"]

  - name: "observium-services"
    use_macros: ["observium-xinetd", "snmp", "ping"]
```

#### Legacy Macro Syntax (still supported)

```yaml
# Legacy syntax with protocols + ports arrays
foomuuri_macros:
  - name: "legacy-service"
    protocols: ["tcp"]
    ports: [80, 443]
```

#### Macro Types

1. **Service Macros** - Define protocols and ports using new mapping syntax:
   ```yaml
   - name: "web-services"
     protocols:
       tcp: [80, 443, 8080]
   ```
   Generates: `tcp 80 443 8080`

2. **Multi-Protocol Services** - Different protocols with same or different ports:
   ```yaml
   - name: "dns-full"
     protocols:
       tcp: [53]
       udp: [53]
   ```
   Generates: `tcp 53; udp 53`

3. **IP Address Macros** - Networks, ranges, individual IPs:
   ```yaml
   - name: "trusted-sources"
     ips: ["192.168.1.0/24", "10.0.0.10", "203.0.113.0/24"]
   ```
   Generates: `192.168.1.0/24 10.0.0.10 203.0.113.0/24`

4. **Macro References** - Combine existing macros:
   ```yaml
   - name: "monitoring-stack"
     use_macros: ["nagios", "snmp", "ping", "ssh"]
   ```
   Generates: `nagios; snmp; ping; ssh`

You can use `foomuuri_extra_macros` in the host_vars or group_vars:

```yaml
foomuuri_extra_macros:
  - name: "my-service"
    protocols:
      tcp: [8080]
  - name: "trusted-ips"
    ips: ["192.168.1.0/24", "10.0.0.0/8"]
```

### Extended Configuration

```yaml
# Add extra zones
foomuuri_extra_zones:
  - name: "dmz"
    interfaces:
      - "eth1"
  - name: "internal" 
    interfaces:
      - "eth2"

# Templates for reuse
foomuuri_extra_templates:
  web_services:
    description: "Common web services"
    rules:
      - "http"
      - "https"

# Zone policies
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost" 
    rules:
      - "ssh saddr trusted-ips"
      - "template web_services"
      - "drop log"
```

### Monitoring Services

The role supports generic monitoring configuration that works with any monitoring solution:

```yaml
# Basic monitoring configuration
foomuuri_monitoring_enabled: true
foomuuri_monitoring_services:
  - name: "monitoring-agent"
    port: "tcp 5666"  # optional, if different from default
    sources:
      public: ["203.0.113.10"]
      management: ["10.0.0.10"]
  - name: "snmp"  # uses default macro
    sources:
      management: ["10.0.0.0/8"]

# Management access
foomuuri_management_enabled: true
foomuuri_management_sources:
  - "192.168.1.0/24"
  - "10.0.0.0/8"
```

### Advanced Configuration

#### NAT (Network Address Translation)

```yaml
# SNAT (Source NAT) - outbound traffic
foomuuri_snat_rules:
  - "saddr 10.0.0.0/8 oifname eth0 masquerade"
  - "saddr 192.168.0.0/16 oifname eth1 snat to 203.0.113.10"

# DNAT (Destination NAT) - port forwarding  
foomuuri_dnat_rules:
  - "daddr 203.0.113.10 http dnat to 10.0.0.10:8080"
  - "daddr 203.0.113.10 ssh dnat to 10.0.0.20:22"
```

#### Zone Mapping (Advanced)

```yaml
# Route traffic to other zones
foomuuri_zonemap_rules:
  - "dipsec dzone public new_dzone vpn"  # IPsec to VPN zone
  - "uid webservice szone localhost new_szone web-zone"  # User-based zones
```

#### Any-Zone Rules

```yaml 
# Rules that apply to all zones
foomuuri_any_zone_rules:
  "any-localhost":  # From any zone to localhost
    - "ssh saddr 192.168.100.0/24"
    - "ping"
  "localhost-any":  # From localhost to any zone
    - "domain"
    - "ntp"
```

#### Traffic Shaping & Marking

```yaml
# Packet marking for QoS
foomuuri_mangle_rules:
  prerouting:
    - "mark_set 0x100/0xff00"
    - "iifname eth2 ssh mark_set 0x300/0xff00"
  postrouting:
    - "oifname eth0 mark_set 0x400/0xff00"
```

#### Dynamic IP Lists

```yaml
# DNS resolving and external IP lists
foomuuri_iplists:
  "@trusted_hosts": 
    - "trusted.example.com"
    - "backup.example.org"
  "@country_nl":
    - "https://raw.githubusercontent.com/ipverse/rir-ip/master/country/nl/ipv4-aggregated.txt"
  "@blacklist":
    - "/etc/foomuuri/blacklist.txt"

# Use in zone policies
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "ssh saddr @trusted_hosts"
      - "http saddr @blacklist drop log"
```

#### Custom nftables Rules

```yaml
# For very advanced configurations
foomuuri_raw_nft_files:
  - name: "custom-qos.nft"
    content: |
      table inet qos {
        chain prerouting {
          type filter hook prerouting priority -150;
          tcp dport {22, 53} mark set 0x1
        }
      }
```

#### Network Monitoring

```yaml
# Network monitoring targets
foomuuri_monitor_targets:
  - name: "gateway"
    target: "192.168.1.1"
    interval: "30s"
  - name: "dns"
    target: "8.8.8.8"
    interval: "60s"
```

#### Special Chain Rules

```yaml
# Special chain rules (invalid, rpfilter, smurfs)
foomuuri_special_chain_rules:
  invalid:
    - "https"  # Accept HTTPS IPVS traffic
  rpfilter:
    - "log"    # Log rpfilter drops
  smurfs:
    - "log"    # Log smurf attacks
```

#### Advanced IPList Configuration

```yaml
# Global iplist settings
foomuuri_iplist_global_settings:
  dns_refresh: "15m"
  dns_timeout: "24h"
  url_refresh: "1d"
  url_timeout: "10d"
```

## Example Playbook

```yaml
---
- hosts: servers
  become: yes
  vars:
    firewall: "foomuuri"
  roles:
    - foomuuri
  vars:
    foomuuri_extra_zones:
      - name: "internal"
        interfaces: ["eth1"]
    
    foomuuri_extra_zone_policies:
      - from_zone: "internal"
        to_zone: "localhost"
        rules:
          - "ssh"
          - "ping" 
          - "drop log"
      
      - from_zone: "localhost"
        to_zone: "internal"
        rules:
          - "accept"
```

## Directory Structure

```
roles/foomuuri/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── install.yml
│   └── configure.yml
├── templates/
│   └── foomuuri.conf.j2
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    ├── Debian.yml
    └── Archlinux.yml
```

## Features

- ✅ Automatic detection of installation method
- ✅ Package installation (Debian 12+, Ubuntu 24.10+)
- ✅ AUR installation (Arch Linux)
- ✅ GitHub source installation (older versions)
- ✅ Interface auto-detection
- ✅ Flexible zone configuration
- ✅ **New macro syntax with protocol mapping**
- ✅ **Legacy macro syntax support**
- ✅ Macro and template support
- ✅ Monitoring services (generic)
- ✅ **SNAT/DNAT (NAT) support**
- ✅ **Zone mapping (advanced routing)**
- ✅ **Any-zone rules (global policies)**
- ✅ **Traffic marking/shaping (mangle)**
- ✅ **Dynamic IP lists (DNS/URLs)**
- ✅ **Custom nftables rules**
- ✅ **Network monitoring**
- ✅ **Special chain rules**
- ✅ Configuration validation
- ✅ Service management (systemd)
- ✅ Multi-OS support (Debian, Ubuntu, Arch Linux)

## Usage

### Basic usage

```yaml
# examples/foomuuri/host_vars/webserver.yml
firewall: "foomuuri"
foomuuri_extra_zones:
  - name: "web"
    interfaces: ["eth0"]

foomuuri_extra_zone_policies:
  - from_zone: "web"
    to_zone: "localhost"
    rules:
      - "http"
      - "https" 
      - "ssh saddr 192.168.1.0/24"
      - "drop log"
```

### Advanced usage with new macro syntax

```yaml
# examples/foomuuri/group_vars/webservers.yml
foomuuri_extra_macros:
  - name: "admin-ips"
    ips: ["192.168.1.100", "192.168.1.101"]
  - name: "web-stack"
    protocols:
      tcp: [80, 443, 8080]

foomuuri_extra_templates:
  web_services:
    description: "Full web application stack"
    rules:
      - "template web-stack"
      - "mysql saddr 10.0.0.0/8"

foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "template web_services"
      - "ssh saddr admin-ips"
      - "drop log"
```

## Commands

After installation, these commands are available:

```bash
# Check status
foomuuri status

# Validate configuration
foomuuri check

# Reload firewall
foomuuri reload

# Show active rules
foomuuri list

# Manage IP lists
foomuuri iplist list
foomuuri iplist add @mylist 192.168.1.100
```

## License

MIT

## Author Information

This role was developed for managing Foomuuri firewalls in enterprise environments.

For more information about Foomuuri itself, see the [official documentation](https://github.com/FoobarOy/foomuuri/wiki).

## Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you would like to change.

## Links

- [Foomuuri GitHub](https://github.com/FoobarOy/foomuuri)
- [Foomuuri Wiki](https://github.com/FoobarOy/foomuuri/wiki)
- [nftables documentation](https://netfilter.org/projects/nftables/)
