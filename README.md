# Tubby1981 System Collection

[![Ansible Galaxy](https://img.shields.io/badge/galaxy-tubby1981.system-blue.svg)](https://galaxy.ansible.com/tubby1981/system)
[![CI](https://github.com/tubby1981/ansible/actions/workflows/ci.yml/badge.svg)](https://github.com/tubby1981/ansible/actions)
[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)
[![Ansible Version](https://img.shields.io/badge/ansible-%3E%3D2.10-blue.svg)](https://www.ansible.com/)

Ansible collection for Linux system management featuring advanced nftables-based firewall automation, Apache web server management with Let's Encrypt SSL, database server configuration, Hetzner Cloud network optimization, and production-ready mail server deployment.

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

### apache
Deploy and manage Apache web server with automatic Let's Encrypt SSL certificate provisioning.

- ✅ Automated SSL certificate provisioning and renewal
- 🔒 Security hardened with modern TLS configuration
- 🌐 HTTP-01 and DNS-01 ACME challenge support
- 🔄 Automatic certificate renewal via systemd timer
- 🎯 Multi-domain support with aliases
- 📋 Pre-flight DNS validation checks
- 🔧 Flexible per-host/per-group configuration

### mariadb 
Configure and manage the MariaDB or MySQL database server, including secure installation, user/database creation, and replication setup.

- ✅ Install and secure MariaDB server
- 👥 Automated user and database management
- 🔄 Idempotent password and privilege management
- 💾 Configuration of key performance settings (InnoDB, memory buffers)
- ⚙️ Support for Master/Slave replication setup
- 🔒 Hardening steps (e.g., remove test database)

### hetzner_network_override
Override cloud-init network configuration on Hetzner Cloud VMs with static networking and configure PowerDNS Recursor for true recursive DNS resolution.

- ✅ Automatic Hetzner Cloud environment detection
- 🌐 Static network configuration with IPv4/IPv6 support
- 🔧 Disables cloud-init network management
- 🔍 PowerDNS Recursor for true recursive DNS (essential for mail servers)
- 🛡️ DNSSEC validation support
- 📧 Spamhaus compliant (prevents spam filter issues)
- 💾 Automatic configuration backups
- 🔒 Immutable resolv.conf to prevent overwrites
- 🔌 Optional Hetzner Cloud API integration
- 🚨 Safe network restart with connection retry

Perfect for mail servers and other services requiring local recursive DNS resolution to avoid issues with spam filters like Spamhaus.

### mailserver
Deploy and manage a production-ready mail server with Postfix, Dovecot, PostfixAdmin, and comprehensive spam/virus protection.

- ✅ Complete mail server stack (SMTP/IMAP)
- 📧 Postfix for mail transfer with virtual domain support
- 📬 Dovecot for IMAP/POP3 with SQL authentication
- 🎛️ PostfixAdmin for web-based administration
- 🛡️ SpamAssassin for spam filtering
- 🦠 ClamAV antivirus integration
- ✉️ OpenDKIM for email authentication
- 🔒 Security-hardened configuration
- 🗄️ MySQL/MariaDB backend for virtual users

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

## 🔧 Requirements

- **Ansible**: >= 2.10
- **Python**: >= 3.6
- **Target systems**: 
  - Debian 11+, Ubuntu 20.04+, or Arch Linux (foomuuri)
  - Ubuntu 20.04+, Debian 10+, CentOS 8+, RHEL 8+ (apache)
  - Debian 11+, Ubuntu 20.04+ (hetzner_network_override)
  - Ubuntu 20.04+, Debian 11+ (mailserver)
- **Privileges**: Root or sudo access required
- **Arch Linux**: `base-devel` and `git` for AUR builds (foomuuri)
- **Mail server**: MariaDB/MySQL database server and Apache web server recommended
- **Hetzner**: Running on Hetzner Cloud (auto-detected by hetzner_network_override)

## 📖 Documentation

- [Apache Role Documentation](roles/apache/README.md)
- [Foomuuri Role Documentation](roles/foomuuri/README.md)
- [MariaDB Role Documentation](roles/mariadb/README.md)
- [Hetzner Network Override Role Documentation](roles/hetzner_network_override/README.md)
- [Mailserver Role Documentation](roles/mailserver/README.md)
- [Example Playbooks](playbooks/)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Changelog](CHANGELOG.md)
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md)

## 💡 Quick Start Examples

### Deploy Foomuuri Firewall
```yaml
- hosts: servers
  become: yes
  roles:
    - tubby1981.system.foomuuri
  vars:
    foomuuri_zones:
      - name: wan
        interface: eth0
```

### Apache with Let's Encrypt SSL
```yaml
- hosts: webservers
  become: yes
  roles:
    - tubby1981.system.apache
  vars:
    apache_domains:
      - name: example.com
        ssl: true
```

### Hetzner Cloud Network Setup
```yaml
- hosts: hetzner_servers
  become: yes
  roles:
    - tubby1981.system.hetzner_network_override
  vars:
    hetzner_use_local_dns: true
    hetzner_install_pdns_recursor: true
```

### Complete Mail Server
```yaml
- hosts: mailserver
  become: yes
  roles:
    - tubby1981.system.mariadb
    - tubby1981.system.apache
    - tubby1981.system.hetzner_network_override
    - tubby1981.system.mailserver
```

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
system hardening, devops automation, network security, apache webserver,
letsencrypt ssl, certbot automation, https configuration, dns-01 challenge,
http-01 challenge, wildcard certificates, apache ansible role,
mariadb ansible role, mysql configuration, database server, replication setup, 
sql automation, hetzner cloud, cloud-init override, powerdns recursor,
recursive dns, dnssec, network automation, static networking,
mail server, postfix, dovecot, postfixadmin, spamassassin,
clamav, opendkim, email server, smtp server, imap server, virtual mailboxes,
spam filtering, antivirus, email security
