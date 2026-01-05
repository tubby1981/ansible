# Ansible Role: Mail Server

A production-ready Ansible role for deploying a fully functional, secure mail server with Postfix, Dovecot, PostfixAdmin, and comprehensive security measures (DKIM, SPF, DMARC, SSL/TLS, anti-spam, anti-virus).

## Features

- ✅ **Full Mail Stack**: Postfix (MTA) + Dovecot (IMAP/POP3)
- ✅ **Web Management**: PostfixAdmin with Apache + Let's Encrypt SSL
- ✅ **Email Options**: Virtual mailboxes AND/OR mail forwarding
- ✅ **Security Standards**: DKIM, SPF, DMARC, TLS/SSL
- ✅ **Anti-Spam**: SpamAssassin with Bayesian filtering
- ✅ **Anti-Virus**: ClamAV with automatic updates
- ✅ **Firewall Integration**: Foomuuri multi-zone firewall
- ✅ **Database Management**: MariaDB + phpMyAdmin
- ✅ **Modular Design**: Integrates with existing roles

## Architecture

This role orchestrates multiple specialized roles:
- **apache**: Web server with Let's Encrypt SSL
- **mariadb**: Database for virtual domains/users
- **phpmyadmin**: Database management interface
- **foomuuri**: Multi-zone firewall configuration
- **package_manager**: Cross-platform package management
- **clamav**: Anti-virus scanning
- **spamassassin**: Spam filtering

## Requirements

- **Ansible**: 2.10 or higher
- **Target OS**: Ubuntu 22.04 LTS or Debian 11+ (recommended)
- **RAM**: Minimum 2GB (4GB recommended for production)
- **Storage**: 20GB+ SSD
- **Network**: Public IPv4 address with reverse DNS (PTR record)
- **Root access**: SSH access with sudo privileges

## Dependencies

Required Ansible roles (automatically included):
```yaml
dependencies:
  - role: package_manager
  - role: mariadb
  - role: phpmyadmin
  - role: apache
  - role: foomuuri
  - role: clamav
  - role: spamassassin
```

## DNS Records Required

Before deployment, configure these DNS records:

| Type | Host | Value | Priority |
|------|------|-------|----------|
| A | mail.example.com | `YOUR_SERVER_IP` | - |
| MX | example.com | mail.example.com | 10 |
| TXT | example.com | `v=spf1 mx ~all` | - |
| TXT | _dmarc.example.com | `v=DMARC1; p=quarantine; rua=mailto:postmaster@example.com` | - |

**Note**: DKIM record will be generated during deployment.

**Critical**: Configure reverse DNS (PTR) through your VPS provider.

## Role Variables

### Domain and Hostname Configuration

```yaml
mail_domain: "example.com"
mail_hostname: "mail.example.com"
mail_server_admin: "admin@example.com"
```

### Mail Server Mode

```yaml
# Choose your mail server mode:
# - 'mailboxes': Store emails in virtual mailboxes (full email hosting)
# - 'forwarding': Forward emails to external addresses (mail relay)
# - 'both': Support both mailboxes and forwarding (most flexible)
mail_server_mode: "both"
```

### Database Configuration

```yaml
# MariaDB configuration (uses mariadb role)
mail_db_name: "postfixadmin"
mail_db_user: "postfixadmin"
mail_db_password: "{{ vault_postfixadmin_db_password }}"
```

### PostfixAdmin Configuration

```yaml
mailserver_postfixadmin_version: "3.3.13"
mailserver_postfixadmin_install_dir: "/var/www/postfixadmin"
mailserver_postfixadmin_setup_password: "{{ vault_postfixadmin_setup_password }}"
mailserver_postfixadmin_admin_email: "{{ mailserver_server_admin }}"
mailserver_postfixadmin_admin_password: "{{ vault_postfixadmin_admin_password }}"
mailserver_postfixadmin_url_path: "postfixadmin"  # Change for security through obscurity
```

### SSL/TLS Configuration

```yaml
# Uses apache role with Let's Encrypt
mailserver_certbot_enabled: true
mailserver_certbot_email: "{{ mailserver_server_admin }}"
```

### DKIM Configuration

```yaml
mailserver_opendkim_selector: "mail"
mailserver_opendkim_key_size: 2048
```

### Mail Storage Configuration (when using mailboxes)

```yaml
mail_storage_path: "/var/mail/vhosts"
mail_uid: 5000
mail_gid: 5000
```

### Firewall Configuration (uses foomuuri role)

```yaml
mail_firewall_enabled: true
mail_firewall_zone: "public"
mail_firewall_management_ips:
  - "192.168.1.0/24"
  - "10.0.0.0/8"
```

### Anti-Spam Configuration (uses spamassassin role)

```yaml
mailserver_spamassassin_enabled: true
mailserver_spamassassin_required_score: 5.0
mailserver_spamassassin_use_bayes: 1
```

### Anti-Virus Configuration (uses clamav role)

```yaml
mailserver_clamav_enabled: true
mailserver_clamav_max_scan_size: "100M"
mailserver_clamav_max_file_size: "25M"
```

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install tubby1981.system
```

### Manual Installation

```bash
git clone https://github.com/tubby1981/ansible.git
cd ansible
ansible-galaxy collection build
ansible-galaxy collection install tubby1981-system-*.tar.gz --force
```

## Quick Start

### 1. Configure Inventory

Create `inventory/hosts.ini`:
```ini
[mailservers]
mail.example.com ansible_user=root ansible_python_interpreter=/usr/bin/python3
```

### 2. Configure Variables

Create `host_vars/mail.example.com.yml`:
```yaml
---
# Domain configuration
mail_domain: "example.com"
mail_hostname: "mail.example.com"
mail_server_admin: "admin@example.com"

# Mail server mode
mail_server_mode: "both"  # mailboxes, forwarding, or both

# Database passwords (use ansible-vault in production)
vault_mariadb_root_password: "SecureRootPassword123!"
vault_mail_db_password: "SecureDBPassword456!"
vault_postfixadmin_setup_password: "SecureSetupPassword789!"
vault_postfixadmin_admin_password: "SecureAdminPassword012!"

# Firewall configuration
mail_firewall_management_ips:
  - "YOUR_ADMIN_IP/32"
```

### 3. Create Playbook

Create `mail-server.yml`:
```yaml
---
- name: Deploy Secure Mail Server
  hosts: mailservers
  become: true
  
  roles:
    - role: tubby1981.system.mailserver
```

### 4. Deploy

```bash
# Test configuration
ansible-playbook -i inventory/hosts.ini mail-server.yml --check

# Deploy
ansible-playbook -i inventory/hosts.ini mail-server.yml
```

## Usage Examples

### Example 1: Virtual Mailboxes Only

```yaml
# host_vars/mail.example.com.yml
---
mail_domain: "example.com"
mail_hostname: "mail.example.com"
mail_server_mode: "mailboxes"

# Apache virtual host
apache_vhosts:
  - server_name: "{{ mailserver_hostname }}"
    document_root: "{{ mailserver_postfixadmin_install_dir }}/public"
    ssl: true
    with_letsencrypt: true

# MariaDB database
mariadb_databases:
  - name: "{{ mailserver_db_name }}"
    encoding: utf8mb4
    collation: utf8mb4_unicode_ci

mariadb_users:
  - name: "{{ mailserver_db_user }}"
    password: "{{ vault_mail_db_password }}"
    priv: "{{ mailserver_db_name }}.*:ALL"
    host: localhost
```

### Example 2: Mail Forwarding Only

```yaml
# host_vars/relay.example.com.yml
---
mail_domain: "example.com"
mail_hostname: "relay.example.com"
mail_server_mode: "forwarding"

# Forward all mail to external addresses
mail_forwarding_enabled: true
mail_default_forward_destination: "external-mail@gmail.com"
```

### Example 3: Both Mailboxes and Forwarding

```yaml
# host_vars/mail.example.com.yml
---
mail_domain: "example.com"
mail_hostname: "mail.example.com"
mail_server_mode: "both"

# Some users have mailboxes, others forward
# Configured via PostfixAdmin web interface
```

### Example 4: Multiple Domains

```yaml
# host_vars/mail.example.com.yml
---
mail_domain: "primary.com"
mail_hostname: "mail.primary.com"
mail_additional_domains:
  - "secondary.com"
  - "another-domain.org"

# All domains managed through PostfixAdmin
```

### Example 5: Custom PostfixAdmin URL Path

```yaml
# host_vars/mail.example.com.yml
---
mailserver_postfixadmin_url_path: "secretmailadmin"

# Accessible at: https://mail.example.com/secretmailadmin
```

## Mail Server Modes Explained

### Mode: mailboxes

**Use Case**: Full email hosting with IMAP/POP3 access

**Features**:
- Virtual mailboxes stored on server
- IMAP/POP3 access via Dovecot
- Webmail possible (not included in this role)
- Email stored in `/var/mail/vhosts/`

**PostfixAdmin Usage**:
- Create mailboxes with passwords
- Set quotas
- Manage aliases

### Mode: forwarding

**Use Case**: Email relay/forwarding service

**Features**:
- No local mailbox storage
- All emails forwarded to external addresses
- Lightweight setup
- No IMAP/POP3 needed

**PostfixAdmin Usage**:
- Create forwarding aliases only
- Map local@domain.com → external@gmail.com

### Mode: both

**Use Case**: Mixed environment (recommended)

**Features**:
- Some users have local mailboxes
- Other users forward to external addresses
- Maximum flexibility
- Choose per user/alias

**PostfixAdmin Usage**:
- Create mailboxes for local users
- Create forwarding aliases for remote users
- Combine both as needed

## Firewall Configuration

This role integrates with the **foomuuri** role for advanced firewall management.

### Default Firewall Rules

```yaml
# Automatically configured by the role
foomuuri_extra_zone_policies:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      # SMTP (receive mail)
      - "smtp"
      
      # Submission (send mail with auth)
      - "submission"
      
      # SMTPS (secure SMTP)
      - "smtps"
      
      # IMAP/IMAPS (when mode is mailboxes or both)
      - "imap"
      - "imaps"
      
      # POP3/POP3S (when mode is mailboxes or both)
      - "pop3"
      - "pop3s"
      
      # PostfixAdmin web interface
      - "https saddr {{ mailserver_firewall_management_ips | join(' ') }}"
      
      # Management access
      - "ssh saddr {{ mailserver_firewall_management_ips | join(' ') }}"
      
      # Drop everything else
      - "drop log"
```

### Custom Firewall Configuration

Override firewall rules in your host variables:

```yaml
# host_vars/mail.example.com.yml
---
mail_firewall_custom_rules:
  - from_zone: "public"
    to_zone: "localhost"
    rules:
      - "smtp"
      - "submission"
      - "smtps"
      - "https saddr @trusted_admins"
      - "drop log"

# Use dynamic IP lists
foomuuri_iplists:
  "@trusted_admins":
    - "admin-home.dyndns.org"
    - "office.example.com"
```

## Apache Integration

This role uses your existing **apache** role with Let's Encrypt support.

### PostfixAdmin Virtual Host

```yaml
# Automatically configured
apache_vhosts:
  - server_name: "{{ mailserver_hostname }}"
    document_root: "{{ mailserver_postfixadmin_install_dir }}/public"
    ssl: true
    with_letsencrypt: true
    redirect_https: true
    
    # Restrict access to PostfixAdmin
    url_protection:
      - path: "/{{ mailserver_postfixadmin_url_path }}"
        allowed_ips: "{{ mailserver_firewall_management_ips }}"
```

### Multiple Virtual Hosts

You can combine mail server with other services:

```yaml
apache_vhosts:
  # PostfixAdmin
  - server_name: "mail.example.com"
    document_root: "{{ mailserver_postfixadmin_install_dir }}/public"
    ssl: true
    with_letsencrypt: true
  
  # Main website
  - server_name: "www.example.com"
    document_root: "/var/www/example"
    ssl: true
    with_letsencrypt: true
    aliases:
      - "example.com"
```

## Post-Deployment Steps

### 1. Retrieve and Configure DKIM Record

```bash
ssh root@mail.example.com
cat /etc/opendkim/keys/example.com/mail.txt
```

Add the DKIM TXT record to your DNS:
- **Host**: `mail._domainkey.example.com`
- **Type**: TXT
- **Value**: Copy the value from `mail.txt`

### 2. Access PostfixAdmin

Open your browser:
```
https://mail.example.com/postfixadmin/
```

**First-time setup**:
1. Navigate to `https://mail.example.com/postfixadmin/setup.php`
2. Enter the setup password
3. Create the admin user
4. Secure or delete `setup.php`

### 3A. Create Mailboxes (mailboxes mode)

In PostfixAdmin:
1. Go to **Domain List** → Select your domain
2. Click **Add Mailbox**
3. Create mailbox: `user@example.com`
4. Set password and quota

**Email Client Settings**:
- **IMAP**: mail.example.com:993 (SSL/TLS)
- **SMTP**: mail.example.com:587 (STARTTLS)
- **Username**: full email address
- **Password**: mailbox password

### 3B. Create Forwarding (forwarding mode)

In PostfixAdmin:
1. Go to **Domain List** → Select your domain
2. Click **Add Alias**
3. Create alias: `local@example.com` → `external@gmail.com`
4. Test forwarding

### 4. Access phpMyAdmin

Database management:
```
https://mail.example.com/phpmyadmin/
```

Use MariaDB root credentials to manage the PostfixAdmin database.

## Testing

### Test DNS Configuration

```bash
# Check MX record
dig MX example.com +short

# Check A record
dig A mail.example.com +short

# Check SPF record
dig TXT example.com +short | grep spf

# Check DKIM record
dig TXT mail._domainkey.example.com +short

# Check DMARC record
dig TXT _dmarc.example.com +short

# Check reverse DNS
dig -x YOUR_SERVER_IP +short
```

### Test Mail Flow

**Send Test Email**:
```bash
# From server
echo "Test email body" | mail -s "Test Subject" recipient@example.com

# Check queue
mailq

# Check logs
tail -f /var/log/mail.log
```

**Receive Test Email**:
Send an email from external service (Gmail, etc.) to your domain.

### Test Security Configuration

Online testing tools:
- [mail-tester.com](https://www.mail-tester.com/) - Overall mail server score
- [MXToolbox](https://mxtoolbox.com/SuperTool.aspx) - DNS and MX checks
- Send to `check-auth@verifier.port25.com` - Detailed SPF/DKIM/DMARC report

## Security Features

- ✅ **TLS/SSL**: Let's Encrypt certificates with auto-renewal (apache role)
- ✅ **DKIM**: Email signing with OpenDKIM
- ✅ **SPF**: Sender Policy Framework configured
- ✅ **DMARC**: Domain-based Message Authentication
- ✅ **Anti-Spam**: SpamAssassin with Bayesian learning (spamassassin role)
- ✅ **Anti-Virus**: ClamAV with daily updates (clamav role)
- ✅ **Firewall**: Multi-zone firewall with Foomuuri (foomuuri role)
- ✅ **Access Control**: IP-based restrictions for admin interfaces
- ✅ **Secure Authentication**: TLS-only authentication for SMTP/IMAP

## Troubleshooting

### Mail Not Sending

```bash
# Check Postfix status
systemctl status postfix

# Check queue
mailq

# Check logs
tail -100 /var/log/mail.log

# Test SMTP
telnet mail.example.com 25
```

### Mail Not Receiving

```bash
# Test MX records
dig MX example.com

# Check Postfix logs
grep "example.com" /var/log/mail.log

# Test port 25
telnet mail.example.com 25
```

### IMAP Issues (mailboxes mode)

```bash
# Check Dovecot status
systemctl status dovecot

# Test IMAP
openssl s_client -connect mail.example.com:993

# Check Dovecot logs
tail -f /var/log/mail.log | grep dovecot
```

### PostfixAdmin Access Issues

```bash
# Check Apache status
systemctl status apache2

# Check SSL certificate
certbot certificates

# Check logs
tail /var/log/apache2/error.log
```

### Firewall Issues

```bash
# Check Foomuuri status
foomuuri status

# Validate configuration
foomuuri check

# View active rules
foomuuri list
```

## Maintenance

### Update System

```bash
ansible-playbook -i inventory/hosts.ini mail-server.yml --tags update
```

### Certificate Renewal

Automatic via certbot timer (apache role manages this).

Manual test:
```bash
certbot renew --dry-run
```

### Backup Configuration

```bash
# Create backup script
cat > /usr/local/bin/mail-backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/mail/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR
mysqldump -u root postfixadmin > $BACKUP_DIR/postfixadmin.sql
tar czf $BACKUP_DIR/mail-config.tar.gz /etc/postfix /etc/dovecot /etc/opendkim
tar czf $BACKUP_DIR/mail-data.tar.gz /var/mail
EOF

chmod +x /usr/local/bin/mail-backup.sh

# Schedule backup
crontab -e
# Add: 0 2 * * * /usr/local/bin/mail-backup.sh
```

## Tags

Run specific parts of the role:

```bash
# Install packages only
ansible-playbook mail-server.yml --tags packages

# Configure Postfix only
ansible-playbook mail-server.yml --tags postfix

# Configure Dovecot only
ansible-playbook mail-server.yml --tags dovecot

# Setup DKIM only
ansible-playbook mail-server.yml --tags dkim

# Configure firewall only
ansible-playbook mail-server.yml --tags firewall
```

Available tags:
- `packages` - Package installation
- `mariadb` - Database setup
- `postfix` - Postfix configuration
- `dovecot` - Dovecot configuration (mailboxes mode)
- `dkim` - DKIM setup
- `postfixadmin` - PostfixAdmin installation
- `apache` - Apache/SSL configuration
- `firewall` - Firewall rules
- `clamav` - Anti-virus setup
- `spamassassin` - Anti-spam setup

## Best Practices

1. **Use ansible-vault** for all passwords
2. **Configure PTR record** through your VPS provider
3. **Test with mail-tester.com** to achieve 10/10 score
4. **Restrict PostfixAdmin access** via IP whitelist
5. **Monitor logs** regularly
6. **Keep system updated** with security patches
7. **Backup regularly** (config + database + mailboxes)
8. **Use strong passwords** for all accounts
9. **Enable fail2ban** for brute-force protection (via foomuuri role)

## License

MIT

## Author Information

This role was created by Tubby1981 as part of the `tubby1981.system` collection.

- **GitHub**: https://github.com/tubby1981/ansible
- **Ansible Galaxy**: https://galaxy.ansible.com/tubby1981/system

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Support

- **Issues**: https://github.com/tubby1981/ansible/issues
- **Documentation**: https://github.com/tubby1981/ansible/roles/mailserver/README.md
