# Ansible Role: foomuuri

[Role](https://galaxy.ansible.com/tubby1981/system)
[Downloads](https://galaxy.ansible.com/tubby1981/system)
[Quality Score](https://galaxy.ansible.com/tubby1981/system)

Configure and manage [Foomuuri](https://github.com/FoobarOy/foomuuri), a modern multizone bidirectional nftables firewall for Linux systems.

## Table of Contents

- [Description](#description)
- [Requirements](#requirements)
- [Supported Platforms](#supported-platforms)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Advanced Features](#advanced-features)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Description

Foomuuri is a modern firewall based on nftables that supports:
- **Multiple security zones** with bidirectional policies
- **Declarative configuration** through simple syntax
- **Advanced NAT** (SNAT/DNAT/masquerading)
- **Dynamic IP lists** from DNS or URLs
- **Traffic marking** and QoS support
- **Zone mapping** for advanced routing scenarios

This Ansible role automates the installation and configuration of Foomuuri across Debian, Ubuntu, and Arch Linux distributions.

## Requirements

- Ansible >= 2.10
- Foomuuri (automatically installed)
- nftables support in kernel (Linux 4.18+)
- Root or sudo privileges on target hosts
- Python >= 3.6 on target hosts
- **For Arch Linux**: `base-devel` and `git` packages for AUR builds

## Supported Platforms

| Operating System | Versions | Installation Method | Status |
|-----------------|----------|---------------------|--------|
| Debian | 12+ (Bookworm) | Package | ✅ Tested |
| Ubuntu | 24.10+ | Package | ✅ Tested |
| Debian | 11 (Bullseye) | GitHub source | ✅ Tested |
| Ubuntu | 20.04, 22.04 | GitHub source | ✅ Tested |
| Arch Linux | Rolling | AUR | ✅ Tested |

## Role Variables

### Installation Variables

```yaml
# Installation method: auto, package, aur, github
# 'auto' automatically selects the best method for your OS
foomuuri_install_method: "auto"

# For Arch Linux: AUR helper
foomuuri_aur_helper: "makepkg"  # Options: makepkg, yay, paru, trizen
```

### Basic Zone Configuration

```yaml
# Minimal zone setup - localhost is always required
foomuuri_zones:
  - name: "public"
    interfaces: 
      - "{{ ansible_default_ipv4.interface | default('eth0') }}"

# Add extra zones in host_vars/group_vars
foomuuri_extra_zones:
  - name: "dmz"
    interfaces: ["eth1"]
  - name: "internal"
    interfaces: ["eth2", "eth3"]
```

### Global Foomuuri Configuration

```yaml
# Only set values that differ from Foomuuri defaults
# See: https://github.com/FoobarOy/foomuuri/wiki/Configuration
foomuuri_config:
  log_input: "yes"
  log_output: "no"
  rpfilter: "yes"
```

### Macro Definitions

Foomuuri supports flexible macro definitions with **new protocol mapping syntax**:

```yaml
foomuuri_macros:
  # Service macros with new protocol syntax
  - name: "web-services"
    protocols:
      tcp: [80, 443, 8080]
  
  - name: "mail-services"
    protocols:
      tcp: [25, 143, 587, 993, 995]
  
  # Multi-protocol services
  - name: "dns-full"
    protocols:
      tcp: [53]
      udp: [53]
  
  - name: "netbios"
    protocols:
      tcp: [139, 445]
      udp: [137, 138]
  
  # IP address macros
  - name: "admin-networks"
    ips: ["192.168.1.0/24", "10.0.0.0/8", "203.0.113.10"]
  
  - name: "monitoring-servers"
    ips: ["83.247.109.32", "83.247.109.43"]
  
  # IPv6 support
  - name: "ipv6-monitoring"
    ips: ["2001:9e0:5:32::43", "2001:9e0:4:32::43"]
  
  # Combining existing macros
  - name: "monitoring-stack"
    use_macros: ["nagios", "snmp", "ping", "ssh"]

# Custom macros in host_vars/group_vars
foomuuri_extra_macros:
  - name: "custom-service"
    protocols:
      tcp: [9000, 9001]
```

**Legacy syntax** (still supported):
```yaml
foomuuri_macros:
  - name: "legacy-service"
    protocols: ["tcp"]
    ports: [80, 443]
```

### Zone Policies

```yaml
# Default policy for localhost zone
foomuuri_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "ping"
      - "ssh"
      - "drop log"

# Custom zone policies
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "http"
      - "https"
      - "ssh saddr admin-networks"
      - "drop log"
  
  - from_zone: "localhost"
    to_zone: "public"  
    rules:
      - "accept"
  
  - from_zone: "dmz"
    to_zone: "internal"
    rules:
      - "mysql saddr 10.0.0.10"
      - "drop log"
```

### Templates (Reusable Rule Groups)

```yaml
foomuuri_extra_templates:
  web_services:
    description: "Common web services"
    rules:
      - "http"
      - "https"
  
  database_access:
    description: "Database services"
    rules:
      - "mysql saddr trusted-sources"
      - "postgresql saddr trusted-sources"

# Use in zone policies
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "template web_services"
      - "ssh saddr admin-networks"
      - "drop log"
```

### NAT Configuration

```yaml
# SNAT (Source NAT) - Masquerading and source address translation
foomuuri_snat_rules:
  - "saddr 10.0.0.0/8 oifname eth0 masquerade"
  - "saddr 192.168.0.0/16 oifname eth1 snat to 203.0.113.10"
  - "saddr fd00:f00:4444::/64 oifname eth2 snat_prefix to 2a03:1111:222:8888::/64"

foomuuri_extra_snat_rules: []

# DNAT (Destination NAT) - Port forwarding
foomuuri_dnat_rules:
  - "daddr 203.0.113.10 http dnat to 10.0.0.10:8080"
  - "daddr 203.0.113.10 https dnat to 10.0.0.10:8443"
  - "iifname eth1 daddr 203.0.113.11 dnat to 10.0.0.11"

foomuuri_extra_dnat_rules: []
```

### Dynamic IP Lists

```yaml
# DNS-based and URL-based IP lists
foomuuri_iplists:
  "@trusted_hosts": 
    - "trusted.example.com"
    - "backup.example.org"
  "@country_nl":
    - "https://raw.githubusercontent.com/ipverse/rir-ip/master/country/nl/ipv4-aggregated.txt"
  "@blacklist":
    - "/etc/foomuuri/blacklist.txt"

# Global iplist settings
foomuuri_iplist_global_settings:
  dns_refresh: "15m"
  dns_timeout: "24h"
  url_refresh: "1d"
  url_timeout: "10d"

# Use in zone policies
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "ssh saddr @trusted_hosts"
      - "http saddr @blacklist drop log"
      - "https saddr @country_nl"
```

### Any-Zone Rules

```yaml
# Rules that apply to any zone combination
foomuuri_any_zone_rules:
  "any-localhost":  # From any zone to localhost
    - "ssh saddr 192.168.100.0/24"
    - "ping"
  "localhost-any":  # From localhost to any zone
    - "domain"
    - "ntp"
    - "http"
    - "https"

foomuuri_extra_any_zone_rules: {}
```

### Advanced Features

- Zone mapping (advanced routing)
- Traffic marking (mangle rules)
- Special chain rules
- Custom nftables rules
- Network monitoring

## Dependencies

None. This role is completely self-contained.

## Example Playbooks

### Basic Web Server

```yaml
---
- name: Configure firewall for web server
  hosts: webservers
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_zones:
        - name: "public"
          interfaces: ["eth0"]
      
      foomuuri_extra_zone_policies:
        - from_zone: "public"
          to_zone: "localhost"
          rules:
            - "http"
            - "https"
            - "ssh saddr 192.168.1.0/24"
            - "drop log"
```

### Multi-Zone DMZ Setup

```yaml
---
- name: Configure DMZ with multiple zones
  hosts: dmz_servers
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_zones:
        - name: "public"
          interfaces: ["eth0"]
        - name: "dmz"
          interfaces: ["eth1"]
        - name: "internal"
          interfaces: ["eth2"]
      
      foomuuri_extra_macros:
        - name: "internal-networks"
          ips: ["10.0.0.0/8", "192.168.0.0/16"]
      
      foomuuri_extra_zone_policies:
        # Public to DMZ
        - from_zone: "public"
          to_zone: "dmz"
          rules:
            - "http"
            - "https"
            - "drop log"
        
        # DMZ to internal (database)
        - from_zone: "dmz"
          to_zone: "internal"
          rules:
            - "mysql saddr internal-networks"
            - "postgresql saddr internal-networks"
            - "drop log"
```

### NAT Gateway

```yaml
---
- name: Configure NAT gateway
  hosts: gateways
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_zones:
        - name: "external"
          interfaces: ["eth0"]
        - name: "internal"
          interfaces: ["eth1"]
      
      # Enable IP forwarding
      foomuuri_config:
        ip_forward: "yes"
      
      # Masquerading for internal network
      foomuuri_snat_rules:
        - "saddr 10.0.0.0/8 oifname eth0 masquerade"
      
      # Port forwarding to internal services
      foomuuri_dnat_rules:
        - "daddr 203.0.113.10 http dnat to 10.0.0.10:8080"
        - "daddr 203.0.113.10 https dnat to 10.0.0.10:8443"
        - "daddr 203.0.113.10 tcp 2222 dnat to 10.0.0.20:22"
```

### Dynamic IP Lists with Geo-Blocking

```yaml
---
- name: Configure firewall with geo-blocking
  hosts: webservers
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_iplists:
        "@country_nl":
          - "https://raw.githubusercontent.com/ipverse/rir-ip/master/country/nl/ipv4-aggregated.txt"
        "@country_be":
          - "https://raw.githubusercontent.com/ipverse/rir-ip/master/country/be/ipv4-aggregated.txt"
        "@trusted_clients":
          - "trusted-client1.example.com"
          - "trusted-client2.example.com"
      
      foomuuri_extra_zone_policies:
        - from_zone: "public"
          to_zone: "localhost"
          rules:
            - "https saddr @trusted_clients"
            - "http saddr @country_nl"
            - "http saddr @country_be"
            - "ssh saddr @trusted_clients"
            - "drop log"
```

## Testing

### Manual Testing

```bash
# Clone the repository
git clone https://github.com/tubby1981/ansible.git
cd ansible

# Install the collection locally
ansible-galaxy collection build
ansible-galaxy collection install tubby1981-system-*.tar.gz --force

# Run test playbook
ansible-playbook tests/test-foomuuri.yml -i tests/inventory
```

### Verify Firewall Configuration

After running the role:

```bash
# Check Foomuuri status
sudo foomuuri status

# Validate configuration
sudo foomuuri check

# Show active nftables rules
sudo foomuuri list

# View detailed nftables ruleset
sudo nft list ruleset

# Check service status
sudo systemctl status foomuuri
```

## Troubleshooting

### Common Issues

#### SSH Lockout Prevention

**Always include SSH rule in your zone policies:**

```yaml
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "ssh"  # MUST BE INCLUDED
      - "drop log"
```

**Recovery**: Use console access:
```bash
sudo foomuuri stop
```

#### NAT Not Working

Ensure IP forwarding is enabled:

```yaml
foomuuri_config:
  ip_forward: "yes"
```

Manual check:
```bash
sysctl net.ipv4.ip_forward
```

#### Dynamic IP Lists Not Updating

Check IP list status:
```bash
sudo foomuuri iplist list
sudo foomuuri iplist show @listname
sudo foomuuri iplist refresh @listname
```

### Debug Commands

```bash
# Validate configuration
sudo foomuuri check

# Show zones
sudo foomuuri zones

# View logs
sudo journalctl -u foomuuri -f

# Check nftables rules
sudo nft list ruleset
```

## Performance Considerations

- **Rule count**: Foomuuri handles thousands of rules efficiently
- **Connection tracking**: nftables connection tracking is very efficient
- **Dynamic IP lists**: DNS lookups happen asynchronously
- **Logging**: Can impact performance on very high-traffic servers (100k+ pps)

## Security Best Practices

1. **Default Deny**: Always end policies with `drop log`
2. **Limit SSH**: Restrict SSH to specific networks
3. **Enable Logging**: Log dropped packets for security monitoring
4. **Use Dynamic Blacklists**: Block known attackers
5. **Separate Management Zone**: Isolate management traffic

## Commands Reference

```bash
# Status and information
sudo foomuuri status          # Show firewall status
sudo foomuuri zones           # List configured zones
sudo foomuuri list            # Show active rules
sudo foomuuri version         # Show version

# Configuration management
sudo foomuuri check           # Validate configuration
sudo foomuuri reload          # Reload configuration
sudo foomuuri restart         # Restart firewall

# Service control
sudo systemctl start foomuuri
sudo systemctl stop foomuuri
sudo systemctl restart foomuuri
sudo systemctl status foomuuri

# IP list management
sudo foomuuri iplist list
sudo foomuuri iplist show @listname
sudo foomuuri iplist refresh @listname
```

## License

MIT

## Author Information

This role was created by Tubby1981 for managing Foomuuri firewalls in production environments.

- **GitHub**: https://github.com/tubby1981/ansible
- **Ansible Galaxy**: https://galaxy.ansible.com/tubby1981/system

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](../../CONTRIBUTING.md) for details.

## Changelog

See [CHANGELOG.md](../../CHANGELOG.md) for version history.

## Additional Resources

- [Foomuuri GitHub Repository](https://github.com/FoobarOy/foomuuri)
- [Foomuuri Wiki](https://github.com/FoobarOy/foomuuri/wiki)
- [Foomuuri Configuration Guide](https://github.com/FoobarOy/foomuuri/wiki/Configuration)
- [nftables Documentation](https://netfilter.org/projects/nftables/)

## Frequently Asked Questions

### Q: Can I use Foomuuri alongside other firewalls?

**A**: No, Foomuuri manages nftables directly. Disable other firewalls (ufw, firewalld) before using Foomuuri.

### Q: Does Foomuuri work with Docker?

**A**: Yes, configure zone policies to allow Docker bridge traffic.

### Q: Can I manage IPv6 with Foomuuri?

**A**: Yes, Foomuuri supports IPv6 natively. Use IPv6 addresses in IP macros and rules.

### Q: How do I backup my configuration?

**A**: Configuration is in `/etc/foomuuri/`:
```bash
sudo tar -czf foomuuri-backup-$(date +%Y%m%d).tar.gz /etc/foomuuri/
```

### Q: What's the difference between Foomuuri and UFW?

**A**: 
- **UFW**: Simple single-zone firewall (iptables/nftables backend)
- **Foomuuri**: Advanced multi-zone bidirectional firewall (nftables only)
- Foomuuri supports: multiple zones, advanced NAT, dynamic IP lists, zone mapping
