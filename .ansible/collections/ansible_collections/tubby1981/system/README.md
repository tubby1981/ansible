# Tubby1981 System Collection

[![Ansible Galaxy](https://img.shields.io/badge/galaxy-tubby1981.system-blue.svg)](https://galaxy.ansible.com/tubby1981/system)
[![CI](https://github.com/tubby1981/ansible/actions/workflows/ci.yml/badge.svg)](https://github.com/tubby1981/ansible/actions)
[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)
[![Ansible Version](https://img.shields.io/badge/ansible-%3E%3D2.10-blue.svg)](https://www.ansible.com/)

Ansible collection for Linux system management featuring advanced nftables-based firewall automation.

## 📦 Included Roles

### foomuuri
Configure and manage [Foomuuri](https://github.com/FoobarOy/foomuuri), a modern multizone bidirectional nftables firewall for Debian, Ubuntu, and Arch Linux systems.

- ✅ Multi-distribution support (Debian/Ubuntu/Arch)
- 🔒 Multi-zone firewall architecture
- 🎯 Declarative nftables configuration
- 🌐 NAT/SNAT/DNAT support
- 📊 Dynamic IP lists (DNS/URL-based)
- 🔄 Idempotent operations
- 🚀 Production-ready

## 🚀 Installation

### Via Ansible Galaxy

```bash
ansible-galaxy collection install tubby1981.system
```

### Via Git (development)

```bash
git clone https://github.com/tubby1981/ansible.git
cd ansible
ansible-galaxy collection build
ansible-galaxy collection install tubby1981-system-*.tar.gz
```

### Via requirements.yml

```yaml
---
collections:
  - name: tubby1981.system
    version: ">=1.0.0"
```

Then install:
```bash
ansible-galaxy collection install -r requirements.yml
```

## 📚 Quick Start

### Basic Multi-Zone Firewall Setup

```yaml
# playbook.yml
---
- name: Configure multi-zone firewall
  hosts: servers
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_zones:
        - name: "public"
          interfaces: ["eth0"]
        - name: "internal"
          interfaces: ["eth1"]
      
      foomuuri_extra_zone_policies:
        - from_zone: "public"
          to_zone: "localhost"
          rules:
            - "http"
            - "https"
            - "ssh saddr 192.168.1.0/24"
            - "drop log"
```

Run the playbook:
```bash
ansible-playbook -i inventory/hosts.yml playbook.yml
```

### Advanced Example with Custom Macros

```yaml
---
- name: Configure advanced firewall with monitoring
  hosts: webservers
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      foomuuri_extra_macros:
        # Define custom service with new protocol syntax
        - name: "web-services"
          protocols:
            tcp: [80, 443, 8080]
        
        # Define trusted IP addresses
        - name: "admin-networks"
          ips: ["192.168.1.0/24", "10.0.0.0/8"]
      
      foomuuri_monitoring_enabled: true
      foomuuri_monitoring_services:
        - name: "nagios"
          port: "tcp 5666"
          sources:
            public: ["203.0.113.10"]
      
      foomuuri_extra_zone_policies:
        - from_zone: "public"
          to_zone: "localhost"
          rules:
            - "web-services"
            - "ssh saddr admin-networks"
            - "nagios saddr 203.0.113.10"
            - "drop log"
```

### NAT/Port Forwarding Example

```yaml
---
- name: Configure gateway with NAT
  hosts: gateways
  become: true
  
  roles:
    - role: tubby1981.system.foomuuri
      # Masquerading for internal network
      foomuuri_snat_rules:
        - "saddr 10.0.0.0/8 oifname eth0 masquerade"
      
      # Port forwarding to internal services
      foomuuri_dnat_rules:
        - "daddr 203.0.113.10 http dnat to 10.0.0.10:8080"
        - "daddr 203.0.113.10 https dnat to 10.0.0.10:8443"
        - "daddr 203.0.113.10 ssh dnat to 10.0.0.20:22"
```

## 🔧 Requirements

- **Ansible**: >= 2.10
- **Python**: >= 3.6
- **Target systems**: Debian 11+, Ubuntu 20.04+, or Arch Linux
- **Privileges**: Root or sudo access required
- **Arch Linux**: `base-devel` and `git` for AUR builds

## 📖 Documentation

- [Foomuuri Role Documentation](roles/foomuuri/README.md)
- [Example Playbooks](playbooks/examples/)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Changelog](CHANGELOG.md)
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md)

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guidelines](CONTRIBUTING.md) first.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 🐛 Bug Reports

Found a bug? Please [open an issue](https://github.com/tubby1981/ansible/issues) with:
- Description of the problem
- Steps to reproduce
- Expected vs actual behavior
- Ansible version and target OS

## 📄 License

[MIT](LICENSE)

## 🔗 Links

- **Ansible Galaxy**: https://galaxy.ansible.com/tubby1981/system
- **Issues**: https://github.com/tubby1981/ansible/issues
- **Repository**: https://github.com/tubby1981/ansible
- **Foomuuri Project**: https://github.com/FoobarOy/foomuuri
- **Documentation**: https://tubby1981.github.io/ansible/

## ⭐ Support

If you find this collection useful, please consider:
- Starring the repository on GitHub
- Rating the collection on Ansible Galaxy
- Sharing it with others

---

**Keywords**: ansible collection, nftables firewall, foomuuri ansible, 
multizone firewall, linux firewall automation, debian ansible role, 
ubuntu firewall configuration, archlinux security, infrastructure as code, 
ansible galaxy collection, advanced firewall, nat configuration, 
system hardening, devops automation, network security
