# Ansible Role: hetzner_network_override

Override cloud-init network configuration on Hetzner Cloud VMs and configure local DNS resolution using PowerDNS Recursor. This role is particularly useful for mail servers and other services that require local recursive DNS resolution to avoid issues with spam filters like Spamhaus.

## Features

- ✅ Automatic detection of Hetzner Cloud environment
- ✅ Fetches network configuration from Hetzner metadata service
- ✅ Optional integration with Hetzner Cloud API
- ✅ Disables cloud-init network management
- ✅ Configures static network interfaces
- ✅ Installs and configures PowerDNS Recursor for true recursive DNS resolution
- ✅ Automatic backup of existing configurations
- ✅ Safe network restart with connection retry
- ✅ IPv4 and IPv6 support
- ✅ DNSSEC validation support

## Why PowerDNS Recursor?

PowerDNS Recursor is a high-performance recursive DNS resolver that queries authoritative nameservers directly, starting from the root servers. This is essential for mail servers because:

- **Spamhaus compliance**: Spam filters like Spamhaus prefer servers that perform their own recursive DNS lookups rather than relying on public DNS forwarders
- **True recursive resolution**: Queries DNS records directly from authoritative sources, not through intermediaries
- **DNSSEC support**: Built-in DNSSEC validation for enhanced security
- **Better deliverability**: Improves email reputation and reduces chances of being flagged as spam
- **Performance**: High-performance caching and query handling
- **Reliability**: No dependency on external DNS resolvers

Unlike DNS forwarders (like dnsmasq with upstream servers), PowerDNS Recursor is a true recursive resolver that performs the full DNS resolution process itself.

## Requirements

- Ansible >= 2.12
- Target system: Debian/Ubuntu (tested on Debian 11/12, Ubuntu 20.04/22.04)
- Running on Hetzner Cloud (auto-detected)
- Root/sudo access on target system

### Python Dependencies

**None required!** This role uses built-in Ansible modules and regex validation, so no additional Python libraries are needed on the control node or target hosts.

**Optional:** If you want to use the `ansible.utils.ipaddr` filter for more advanced IP validation, you can install the `netaddr` library:

```bash
# If using pipx (recommended for Ansible in Docker)
pipx inject ansible-core netaddr

# Or with pip
pip install netaddr
```

### Optional Requirements

For Hetzner API integration:
- Hetzner Cloud API token
- `hetzner.hcloud` collection: `ansible-galaxy collection install hetzner.hcloud`
- `community.general` collection: `ansible-galaxy collection install community.general`

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install username.hetzner_network_override
```

### Manual Installation

```bash
cd roles/
git clone https://github.com/username/ansible-role-hetzner-network-override.git hetzner_network_override
```

## Role Variables

### Required Variables

None! The role auto-detects network configuration from the system.

### Optional Variables

#### API Configuration

```yaml
# Hetzner Cloud API token (optional, for additional validation)
hetzner_api_token: "your_hetzner_api_token"
```

#### Network Configuration Override

```yaml
# Override auto-detected network settings
hetzner_ipv4_address: "1.2.3.4"
hetzner_ipv4_gateway: "1.2.3.1"
hetzner_ipv6_address: "2a01:4f8:1c1a:66cf::1"
hetzner_ipv6_prefix: "64"
hetzner_ipv6_gateway: "fe80::1"

# Network interface name (default: eth0)
hetzner_network_interface: "eth0"
```

#### DNS Configuration

```yaml
# Enable local DNS resolution (default: true)
hetzner_use_local_dns: true

# Install and configure PowerDNS Recursor (default: true)
hetzner_install_pdns_recursor: true

# PowerDNS Recursor listening address (default: 127.0.0.1)
hetzner_pdns_local_address: "127.0.0.1"

# PowerDNS Recursor listening port (default: 53)
hetzner_pdns_local_port: "53"

# Number of threads (default: 2)
hetzner_pdns_threads: "2"

# Maximum cache entries (default: 1000000)
hetzner_pdns_max_cache_entries: "1000000"

# Maximum negative TTL in seconds (default: 3600)
hetzner_pdns_max_negative_ttl: "3600"

# Log level 0-9 (default: 4)
hetzner_pdns_log_level: "4"

# Log common errors (default: true)
hetzner_pdns_log_common_errors: true

# DNSSEC validation: process, log-fail, or off (default: process)
hetzner_pdns_dnssec: "process"

# Forward specific zones to other nameservers (optional)
hetzner_pdns_forward_zones:
  - zone: "example.local"
    forwarders: "10.0.0.1;10.0.0.2"
  - zone: "internal.corp"
    forwarders: "192.168.1.1"

# resolv.conf options (default: "edns0 trust-ad" for DNSSEC)
hetzner_resolv_conf_options: "edns0 trust-ad"

# Make resolv.conf immutable to prevent DHCP overwrites (default: true)
# This is CRITICAL on Hetzner Cloud to prevent cloud-init/DHCP from resetting DNS
hetzner_make_resolv_immutable: true
```

#### Cloud-init Configuration

```yaml
# Disable cloud-init network management (default: true)
hetzner_disable_cloud_init_network: true

# Keep original cloud-init config as backup (default: false)
hetzner_keep_cloud_init_config: false
```

#### Safety Settings

```yaml
# Require manual confirmation before applying changes (default: false)
hetzner_require_confirmation: true

# Create backup of existing configurations (default: true)
hetzner_backup_configs: true

# Force auto-detection even if variables are set (default: false)
hetzner_force_detection: false
```

## Dependencies

None required. Optional: `hetzner.hcloud` collection for API integration.

## Example Playbook

### Basic Usage (Auto-detection)

```yaml
- hosts: mailserver
  become: yes
  roles:
    - hetzner_network_override
```

### With API Token

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_api_token: "{{ lookup('env', 'HETZNER_API_TOKEN') }}"
  roles:
    - hetzner_network_override
```

### With Safety Confirmation

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_require_confirmation: true
    hetzner_backup_configs: true
  roles:
    - hetzner_network_override
```

### Manual Network Configuration

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_auto_detect: false
    hetzner_ipv4_address: "1.2.3.4"
    hetzner_ipv4_gateway: "1.2.3.1"
    hetzner_ipv6_address: "2a01:4f8:1c1a:66cf::1"
  roles:
    - hetzner_network_override
```

### Custom PowerDNS Configuration

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_pdns_threads: "4"
    hetzner_pdns_max_cache_entries: "2000000"
    hetzner_pdns_dnssec: "process"
    hetzner_pdns_log_level: "6"
  roles:
    - hetzner_network_override
```

### With Forward Zones (for internal domains)

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_pdns_forward_zones:
      - zone: "internal.company.com"
        forwarders: "10.0.0.1;10.0.0.2"
      - zone: "vpn.local"
        forwarders: "192.168.1.1"
  roles:
    - hetzner_network_override
```

## Usage with Ansible Vault

Store your API token securely:

```bash
ansible-vault create group_vars/all/vault.yml
```

```yaml
# group_vars/all/vault.yml
vault_hetzner_api_token: "your_secret_token"
```

```yaml
# group_vars/all/vars.yml
hetzner_api_token: "{{ vault_hetzner_api_token }}"
```

Run playbook:

```bash
ansible-playbook -i inventory playbook.yml --ask-vault-pass
```

## How It Works

1. **Detection Phase**
   - Checks if running on Hetzner Cloud via metadata service
   - Retrieves current network configuration from Ansible facts
   - Optionally fetches additional info from Hetzner Cloud API

2. **Validation Phase**
   - Validates network configuration
   - Optional: prompts for confirmation

3. **Backup Phase**
   - Creates timestamped backup of existing configurations
   - Backs up: cloud-init config, network interfaces, resolv.conf, PowerDNS config

4. **Configuration Phase**
   - Disables cloud-init network management
   - Configures static network interfaces
   - Installs and configures PowerDNS Recursor
   - Updates resolv.conf and makes it immutable

5. **Restart Phase**
   - Safely restarts networking service
   - Waits for connection to restore
   - Restarts PowerDNS Recursor

## DNS Resolution Flow

With PowerDNS Recursor, your server performs true recursive DNS resolution:

1. Application queries `127.0.0.1:53` (PowerDNS Recursor)
2. PowerDNS Recursor checks its cache
3. If not cached, it queries root nameservers
4. Follows referrals to TLD nameservers (.com, .org, etc.)
5. Queries authoritative nameservers for the final answer
6. Validates DNSSEC signatures (if enabled)
7. Caches the result and returns it to the application

This is different from DNS forwarders which simply forward queries to upstream resolvers.

## Safety Features

- **Automatic backups**: All original configurations are backed up before changes
- **Connection retry**: Automatically reconnects after network restart
- **Async network restart**: Prevents Ansible from hanging
- **Immutable resolv.conf**: Prevents other services from overwriting DNS config
- **Validation**: Checks network configuration before applying
- **Optional confirmation**: Can require manual approval before changes
- **PowerDNS health check**: Waits for PowerDNS to be ready before continuing

## Emergency Recovery

If something goes wrong and you lose SSH access:

1. **Use Hetzner Cloud Console** to access your server
2. **Restore from backup**:
   ```bash
   # Find backup directory
   ls -la /root/network_config_backup_*
   
   # Restore configuration
   cd /root/network_config_backup_TIMESTAMP/
   cp 50-cloud-init.backup /etc/network/interfaces.d/50-cloud-init
   cp recursor.conf.backup /etc/powerdns/recursor.conf
   rm /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
   
   # Restart services
   systemctl restart pdns-recursor
   systemctl restart networking
   ```

3. **Or re-enable cloud-init**:
   ```bash
   rm /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
   cloud-init clean
   reboot
   ```

## Testing

### Test in safe mode:

```yaml
- hosts: mailserver
  become: yes
  vars:
    hetzner_require_confirmation: true
    hetzner_backup_configs: true
  roles:
    - hetzner_network_override
  
  post_tasks:
    - name: Verify network connectivity
      ansible.builtin.ping:
    
    - name: Verify DNS resolution
      ansible.builtin.command: host google.com
      changed_when: false
    
    - name: Check PowerDNS Recursor status
      ansible.builtin.systemd:
        name: pdns-recursor
        state: started
      check_mode: yes
    
    - name: Test recursive DNS query
      ansible.builtin.command: dig @127.0.0.1 google.com +trace
      changed_when: false
```

## Troubleshooting

### DNS not resolving

```bash
# Check PowerDNS Recursor status
systemctl status pdns-recursor

# Check PowerDNS Recursor logs
journalctl -u pdns-recursor -n 50

# Test DNS manually
dig @127.0.0.1 google.com

# Test with trace to see recursive resolution
dig @127.0.0.1 google.com +trace

# Check statistics
rec_control get-all
```

### PowerDNS Recursor not starting

```bash
# Check configuration syntax
pdns_recursor --config-check

# View detailed logs
journalctl -u pdns-recursor -f

# Check if port 53 is already in use
netstat -tulpn | grep :53
```

### Network not coming up

```bash
# Check interface status
ip addr show eth0

# Check routing
ip route show

# Restart networking manually
systemctl restart networking
```

### Cloud-init still managing network

```bash
# Verify cloud-init is disabled
cat /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

# Clean cloud-init
cloud-init clean --logs --configs network
```

### Testing DNSSEC validation

```bash
# Test DNSSEC validation (should succeed)
dig @127.0.0.1 dnssec.works

# Test DNSSEC failure (should fail with SERVFAIL if validation is working)
dig @127.0.0.1 dnssec-failed.org
```

## Why This Role?

When running mail servers or similar services on Hetzner Cloud, using public DNS forwarders can cause issues:

- **Spam filters** like Spamhaus require servers to perform their own recursive DNS resolution
- **Email reputation**: Using DNS forwarders can negatively impact your mail server's reputation
- **PTR record lookups**: Work better with recursive resolution
- **Performance**: Local caching improves response times
- **Reliability**: No dependency on external DNS resolvers during outages
- **DNSSEC validation**: Built-in support for enhanced security

This role solves these issues by:
1. Configuring PowerDNS Recursor for true recursive DNS resolution
2. Preventing cloud-init from overwriting your configuration
3. Maintaining static network configuration for reliability
4. Enabling DNSSEC validation for security

## PowerDNS Recursor vs DNS Forwarders

| Feature | PowerDNS Recursor | DNS Forwarders (dnsmasq) |
|---------|------------------|-------------------------|
| Resolution Type | True recursive | Forwards to upstream |
| Starts from | Root nameservers | Configured upstreams |
| Spamhaus Compliant | ✅ Yes | ❌ No |
| DNSSEC Validation | ✅ Built-in | ⚠️ Limited |
| Mail Server Suitability | ✅ Excellent | ❌ Poor |
| Cache Performance | ✅ High | ✅ High |

## License

MIT

## Author Information

Created for managing Hetzner Cloud infrastructure with Ansible, specifically optimized for mail servers requiring recursive DNS resolution.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Changelog

### Version 2.0.0 (2024-12-29)
- **BREAKING**: Replaced dnsmasq with PowerDNS Recursor
- Added true recursive DNS resolution
- Enhanced DNSSEC support
- Improved mail server compatibility
- Better Spamhaus compliance

### Version 1.0.0 (2024-12-29)
- Initial release
- Auto-detection of Hetzner Cloud environment
- Static network configuration
- dnsmasq integration (deprecated)
- Cloud-init override
- IPv6 support

## Support

- Issues: https://github.com/username/ansible-role-hetzner-network-override/issues
- Documentation: https://github.com/username/ansible-role-hetzner-network-override/wiki

## Related Resources

- [PowerDNS Recursor Documentation](https://doc.powerdns.com/recursor/)
- [Spamhaus DNS Best Practices](https://www.spamhaus.org/)
- [DNSSEC Information](https://www.dnssec.net/)
