# Ansible Role: phpMyAdmin

This Ansible role installs and configures phpMyAdmin on Debian/Ubuntu and RHEL/CentOS systems.

## Requirements

- Ansible 2.10 or higher
- A working web server (Apache or Nginx)
- PHP 7.4 or higher
- MySQL or MariaDB database server

## Role Variables

All variables are defined in `defaults/main.yml` and can be overridden:

### General Configuration

- `phpmyadmin_install_enabled` (boolean, default: `true`): Enable or disable phpMyAdmin installation
- `phpmyadmin_package_name` (string, default: `phpmyadmin`): Name of the phpMyAdmin package
- `phpmyadmin_webserver_name` (string, default: `apache2`): Name of the web server service (`apache2` or `nginx`)
- `phpmyadmin_config_dir` (string, default: `/etc/phpmyadmin`): Location of configuration files
- `phpmyadmin_web_root` (string, default: `/usr/share/phpmyadmin`): Web root directory of phpMyAdmin

### URL Path Configuration

- `phpmyadmin_url_path` (string, default: `phpmyadmin`): URL path where phpMyAdmin is accessible
  - Default: accessible via `http://domain.com/phpmyadmin`
  - Example: set to `phpmydinges` to access via `http://domain.com/phpmydinges`
  - Example: set to `secretdbadmin` for security through obscurity

### PHP Configuration

- `phpmyadmin_upload_max_filesize` (string, default: `"64M"`): Maximum upload file size
- `phpmyadmin_post_max_size` (string, default: `"64M"`): Maximum POST data size
- `phpmyadmin_memory_limit` (string, default: `"256M"`): PHP memory limit

### Security Configuration

- `phpmyadmin_restrict_access` (boolean, default: `false`): Restrict access to specific IP addresses
- `phpmyadmin_allowed_hosts` (list, default: `[]`): List of allowed IP addresses/CIDR ranges
- `phpmyadmin_blowfish_secret` (string): Secret key for cookie authentication (automatically generated)
- `phpmyadmin_allow_arbitrary_server` (boolean, default: `false`): Allow arbitrary server connections
- `phpmyadmin_default_server` (string, default: `localhost`): Default database server

## Dependencies

This role depends on the `php` role to be executed first. The PHP role must be configured before phpMyAdmin can be installed.

### Required Role

- **php**: Installs and configures PHP and PHP-FPM

### Automatic Dependency Resolution

The role automatically includes the `php` role as a dependency via `meta/main.yml`. However, you can also explicitly define it in your playbook for better control:
```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: php
      vars:
        php_version: "8.2"
        php_extensions:
          - cli
          - fpm
          - mysql
          - mbstring
          - zip
          - gd
          - curl
          - xml
    - role: phpmyadmin
      vars:
        phpmyadmin_url_path: phpmydinges
```

### Pre-Installation Checks

The role performs the following checks before installation:

1. Verifies PHP is installed and accessible
2. Checks for Ansible facts created by the PHP role
3. Automatically detects the installed PHP version
4. Validates PHP configuration directories exist

If PHP is not properly installed, the role will fail with a helpful error message instructing you to install the `php` role first.

## Example Playbook
```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: phpmyadmin
      vars:
        phpmyadmin_url_path: secretdbmanager
        phpmyadmin_restrict_access: true
        phpmyadmin_allowed_hosts:
          - "192.168.1.0/24"
          - "10.0.0.100"
        phpmyadmin_upload_max_filesize: "128M"
        phpmyadmin_memory_limit: "512M"
```

## Example Host Variables

Create a file `host_vars/your_host.yml`:
```yaml
---
phpmyadmin_install_enabled: true
phpmyadmin_webserver_name: apache2

# Custom URL path for security through obscurity
phpmyadmin_url_path: phpmydinges

# Security settings - restrict access
phpmyadmin_restrict_access: true
phpmyadmin_allowed_hosts:
  - "192.168.1.0/24"       # Local network
  - "10.0.0.100"           # Specific admin IP
  - "172.16.50.0/24"       # VPN network

# Increase PHP limits for larger databases
phpmyadmin_upload_max_filesize: "256M"
phpmyadmin_post_max_size: "256M"
phpmyadmin_memory_limit: "512M"

# Security configuration
phpmyadmin_blowfish_secret: "change-this-to-a-secure-32char-string!"
phpmyadmin_allow_arbitrary_server: false
phpmyadmin_default_server: localhost
```

## Accessing phpMyAdmin

After installation, phpMyAdmin is accessible via:
- **Default**: `http://your-server/phpmyadmin`
- **Custom URL**: `http://your-server/{{ phpmyadmin_url_path }}` (e.g., `/phpmydinges`)

## Security

### Important security recommendations:

1. **Restrict Access**: Always use `phpmyadmin_restrict_access: true` in production
2. **Custom URL Path**: Change `phpmyadmin_url_path` to something non-standard for security through obscurity
3. **Strong Blowfish Secret**: Override `phpmyadmin_blowfish_secret` with a unique 32-character string
4. **HTTPS Only**: Use phpMyAdmin only via HTTPS
5. **Database Users**: Don't use root access; create dedicated database users with limited privileges
6. **Firewall**: Combine IP restrictions with firewall rules for defense in depth

### Security through Obscurity

While not a replacement for proper security measures, changing the default URL path can help reduce automated attacks:
```yaml
# Instead of the default /phpmyadmin, use something less obvious:
phpmyadmin_url_path: my-secret-db-admin-2024
```

## Ansible Lint

This role is fully compatible with ansible-lint. Test with:
```bash
ansible-lint roles/phpmyadmin
```

## Testing the Role
```bash
# Ansible lint check
ansible-lint roles/phpmyadmin

# Syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook playbook.yml --check

# Actual execution
ansible-playbook playbook.yml
```

## License

MIT

## Author

Generated by Claude for Ansible automation
