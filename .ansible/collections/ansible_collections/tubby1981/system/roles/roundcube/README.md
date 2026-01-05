# Ansible Role: Roundcube

An Ansible role for installing and configuring Roundcube webmail.

## Description

This role installs and configures Roundcube webmail including:
- Download and installation of Roundcube
- Database setup (MySQL/PostgreSQL)
- Apache or Nginx vhost configuration
- SSL configuration
- Plugins (managesieve, password, enigma, etc.)
- Composer dependencies
- Security (deny sensitive directories, security headers)

## Requirements

- Ansible 2.9 or higher
- A running web server (Apache or Nginx)
- PHP 7.4 or higher
- MySQL/MariaDB or PostgreSQL database server
- A working mail server with IMAP and SMTP

## Role Variables

### Basic Configuration

```yaml
# Roundcube version
roundcube_version: "1.6.7"

# Download URL
roundcube_download_url: >-
  https://github.com/roundcube/roundcubemail/releases/download/{{ roundcube_version }}/roundcubemail-{{ roundcube_version }}-complete.tar.gz

# Installation paths
roundcube_install_dir: "/var/www/roundcube"
roundcube_temp_dir: "/tmp/roundcube"

# Web server (apache or nginx)
roundcube_webserver: "apache"
roundcube_vhost_enabled: true

# Domain configuration
roundcube_server_name: "webmail.example.com"
roundcube_enable_ssl: true
roundcube_ssl_certificate: "/etc/ssl/certs/ssl-cert-snakeoil.pem"
roundcube_ssl_certificate_key: "/etc/ssl/private/ssl-cert-snakeoil.key"
```

### Integration with Existing Roles

```yaml
# Use existing role configurations
roundcube_use_existing_php: true
roundcube_use_existing_apache: true
roundcube_use_existing_mariadb: true
```

### Database Configuration

```yaml
roundcube_database_type: "mysql"
roundcube_database_host: "localhost"
roundcube_database_port: 3306
roundcube_database_name: "roundcube"
roundcube_database_user: "roundcube"
roundcube_database_password: "changeme"  # Use vault!
```

### Mail Server Configuration

```yaml
# IMAP
roundcube_default_host: "ssl://localhost"
roundcube_default_port: 993

# SMTP
roundcube_smtp_server: "tls://localhost"
roundcube_smtp_port: 587
roundcube_smtp_user: "%u"
roundcube_smtp_pass: "%p"
```

### General Settings

```yaml
roundcube_product_name: "Webmail"
roundcube_support_url: ""
roundcube_skin: "elastic"
roundcube_language: "nl_NL"
roundcube_timezone: "Europe/Amsterdam"
```

### Password Settings

```yaml
roundcube_password_charset: "UTF-8"
roundcube_password_minimum_length: 8
roundcube_password_require_digit: true
roundcube_password_require_special: true
```

### Session Settings

```yaml
roundcube_session_lifetime: 30
roundcube_session_domain: ""
roundcube_session_path: "/"
```

### Upload Limits

```yaml
roundcube_upload_max_filesize: "10M"
roundcube_post_max_size: "10M"
```

### Logging

```yaml
roundcube_log_driver: "file"
roundcube_log_dir: "logs/"
roundcube_debug_level: 1
```

### Plugins

```yaml
roundcube_plugins:
  - archive
  - zipdownload
  - managesieve
  - password
  - enigma
```

### Plugin Configuration

#### Managesieve (Sieve Filters)

```yaml
roundcube_managesieve_enabled: true
roundcube_managesieve_host: "localhost"
roundcube_managesieve_port: 4190
roundcube_managesieve_usetls: true
```

#### Password (Password Change)

```yaml
roundcube_password_enabled: true
roundcube_password_driver: "sql"
roundcube_password_db_dsn: "mysql://user:pass@localhost/postfixadmin"
roundcube_password_query: "UPDATE mailbox SET password=%c WHERE username=%u"
roundcube_password_crypt_hash: "bcrypt"
```

#### Enigma (PGP Encryption)

```yaml
roundcube_enigma_enabled: false
roundcube_enigma_pgp_homedir: "/var/www/roundcube/enigma/home"
```

### PHP-FPM Configuration

```yaml
roundcube_php_fpm_enabled: false
roundcube_php_fpm_pool_name: "roundcube"
roundcube_php_fpm_user: "www-data"
roundcube_php_fpm_group: "www-data"
roundcube_php_fpm_listen: "/run/php/php8.4-fpm.sock"
```

### Security

```yaml
roundcube_des_key: ""
roundcube_enable_installer: false
```

### Required PHP Extensions

```yaml
roundcube_required_php_extensions:
  - "php8.4-intl"
  - "php8.4-ldap"
  - "php8.4-pspell"
  - "php8.4-enchant"
  - "php8.4-imagick"
```

See `defaults/main.yml` for all available variables.

## Dependencies

No mandatory dependencies, but the following roles are recommended:
- `tubby1981.system.apache` or nginx role
- `tubby1981.system.php`
- `tubby1981.system.mariadb`

## Example Playbooks

### Basic Installation

```yaml
---
- hosts: webmail
  become: yes
  
  vars:
    roundcube_server_name: "webmail.example.com"
    roundcube_database_password: "{{ vault_roundcube_db_password }}"
    roundcube_default_host: "ssl://mail.example.com"
    roundcube_smtp_server: "tls://mail.example.com"
  
  roles:
    - tubby1981.system.apache
    - tubby1981.system.php
    - tubby1981.system.mariadb
    - tubby1981.system.roundcube
```

### With Custom Plugins and Configuration

```yaml
---
- hosts: webmail
  become: yes
  
  vars:
    # Server configuration
    roundcube_server_name: "webmail.example.com"
    roundcube_enable_ssl: true
    roundcube_ssl_certificate: "/etc/letsencrypt/live/webmail.example.com/fullchain.pem"
    roundcube_ssl_certificate_key: "/etc/letsencrypt/live/webmail.example.com/privkey.pem"
    
    # Database
    roundcube_database_password: "{{ vault_roundcube_db_password }}"
    
    # Mail server
    roundcube_default_host: "ssl://mail.example.com"
    roundcube_smtp_server: "tls://mail.example.com"
    
    # Plugins
    roundcube_plugins:
      - archive
      - zipdownload
      - managesieve
      - password
      - enigma
      - newmail_notifier
      - vcard_attachments
    
    # Managesieve configuration
    roundcube_managesieve_enabled: true
    roundcube_managesieve_host: "mail.example.com"
    roundcube_managesieve_port: 4190
    
    # Password plugin
    roundcube_password_enabled: true
    roundcube_password_driver: "sql"
    roundcube_password_query: "UPDATE mailbox SET password=ENCRYPT(%p, CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))) WHERE username=%u"
    
    # Localization
    roundcube_language: "en_US"
    roundcube_timezone: "America/New_York"
  
  roles:
    - tubby1981.system.roundcube
```

### With Nginx

```yaml
---
- hosts: webmail
  become: yes
  
  vars:
    roundcube_webserver: "nginx"
    roundcube_server_name: "webmail.example.com"
    roundcube_php_fpm_enabled: true
    roundcube_php_fpm_listen: "/run/php/php8.4-fpm.sock"
  
  roles:
    - nginx
    - tubby1981.system.php
    - tubby1981.system.roundcube
```

## Host Vars Example

```yaml
# host_vars/webmail.example.com/roundcube.yml
---
roundcube_server_name: "webmail.example.com"
roundcube_enable_ssl: true
roundcube_ssl_certificate: "/etc/ssl/certs/webmail.example.com.crt"
roundcube_ssl_certificate_key: "/etc/ssl/private/webmail.example.com.key"

roundcube_database_host: "localhost"
roundcube_database_name: "roundcube"
roundcube_database_user: "roundcube"
# Password in vault file

roundcube_default_host: "ssl://mail.example.com"
roundcube_default_port: 993
roundcube_smtp_server: "tls://mail.example.com"
roundcube_smtp_port: 587

roundcube_product_name: "Example Webmail"
roundcube_language: "en_US"
roundcube_timezone: "America/New_York"
```

## Initial Installation

1. Set `roundcube_enable_installer: true` in your variables
2. Run the playbook
3. Navigate to `https://webmail.example.com/installer`
4. Follow the installation wizard
5. Set `roundcube_enable_installer: false`
6. Run the playbook again to remove the installer directory

## Security Best Practices

1. **Use Ansible Vault** for sensitive data:
   ```bash
   ansible-vault create host_vars/webmail/vault.yml
   ```

2. **SSL Certificates**: Use Let's Encrypt or other valid certificates

3. **Database**: Create a dedicated database user with minimal privileges

4. **Firewall**: Configure your firewall to allow only HTTPS traffic

5. **Updates**: Keep Roundcube up-to-date by adjusting `roundcube_version`

## Troubleshooting

### Database Connection Errors

Check if the database credentials are correct and if the database is reachable:

```bash
mysql -h localhost -u roundcube -p roundcube
```

### PHP Errors

Check the PHP error log:

```bash
tail -f /var/log/php8.4-fpm.log
```

### Web Server Errors

Apache:
```bash
tail -f /var/log/apache2/roundcube-error.log
```

Nginx:
```bash
tail -f /var/log/nginx/roundcube-error.log
```

### Permission Issues

Ensure that the temp and logs directories are writable:

```bash
chown -R www-data:www-data /var/www/roundcube/temp
chown -R www-data:www-data /var/www/roundcube/logs
```

## License

MIT

## Author

tubby1981

## Contributing

Contributions are welcome! Please submit a pull request or open an issue on the repository.
