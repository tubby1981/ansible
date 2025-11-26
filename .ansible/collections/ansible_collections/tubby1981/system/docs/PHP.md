# Ansible Role: PHP

This Ansible role installs and configures PHP and PHP-FPM on Debian/Ubuntu and RHEL/CentOS systems with configurable PHP versions and extensions.

## Requirements

- Ansible 2.10 or higher
- Supported OS:
  - Ubuntu 20.04, 22.04, 24.04
  - Debian 11, 12
  - RHEL/CentOS 8, 9

## Role Variables

All variables are defined in `defaults/main.yml` and can be overridden:

### General Configuration

- `php_install_enabled` (boolean, default: `true`): Enable or disable PHP installation
- `php_version` (string, default: `"8.1"`): PHP version to install (7.4, 8.0, 8.1, 8.2, 8.3)
- `php_webserver_name` (string, default: `apache2`): Web server to integrate with (`apache2` or `nginx`)

### PHP-FPM Configuration

- `php_fpm_enabled` (boolean, default: `true`): Enable PHP-FPM
- `php_fpm_pool_user` (string, default: `www-data`): PHP-FPM pool user
- `php_fpm_pool_group` (string, default: `www-data`): PHP-FPM pool group
- `php_fpm_listen` (string, default: `/run/php/php{{ php_version }}-fpm.sock`): Socket path
- `php_fpm_pm_mode` (string, default: `dynamic`): Process manager mode
- `php_fpm_pm_max_children` (integer, default: `5`): Maximum number of child processes
- `php_fpm_pm_start_servers` (integer, default: `2`): Number of child processes at startup
- `php_fpm_pm_min_spare_servers` (integer, default: `1`): Minimum spare servers
- `php_fpm_pm_max_spare_servers` (integer, default: `3`): Maximum spare servers
- `php_fpm_pm_max_requests` (integer, default: `500`): Maximum requests per child

### PHP Configuration Settings

- `php_memory_limit` (string, default: `"256M"`): PHP memory limit
- `php_max_execution_time` (integer, default: `300`): Maximum execution time in seconds
- `php_max_input_time` (integer, default: `300`): Maximum input time in seconds
- `php_max_input_vars` (integer, default: `3000`): Maximum input variables
- `php_post_max_size` (string, default: `"64M"`): Maximum POST size
- `php_upload_max_filesize` (string, default: `"64M"`): Maximum upload file size
- `php_max_file_uploads` (integer, default: `20`): Maximum file uploads

### Error Handling

- `php_display_errors` (string, default: `"Off"`): Display errors
- `php_display_startup_errors` (string, default: `"Off"`): Display startup errors
- `php_error_reporting` (string, default: `"E_ALL & ~E_DEPRECATED & ~E_STRICT"`): Error reporting level
- `php_log_errors` (string, default: `"On"`): Enable error logging
- `php_error_log` (string, default: `/var/log/php_errors.log`): Error log path

### Session Configuration

- `php_session_save_handler` (string, default: `files`): Session save handler
- `php_session_save_path` (string, default: `/var/lib/php/sessions`): Session save path
- `php_session_gc_maxlifetime` (integer, default: `1440`): Session garbage collection max lifetime

### Timezone

- `php_timezone` (string, default: `"Europe/Amsterdam"`): PHP timezone

### OPcache Configuration

- `php_opcache_enabled` (boolean, default: `true`): Enable OPcache
- `php_opcache_memory_consumption` (integer, default: `128`): OPcache memory in MB
- `php_opcache_interned_strings_buffer` (integer, default: `8`): Interned strings buffer
- `php_opcache_max_accelerated_files` (integer, default: `4000`): Maximum cached files
- `php_opcache_revalidate_freq` (integer, default: `60`): Revalidation frequency in seconds

### PHP Extensions

- `php_extensions` (list): Required PHP extensions to install
```yaml
  php_extensions:
    - cli
    - fpm
    - common
    - mysql
    - zip
    - gd
    - mbstring
    - curl
    - xml
    - bcmath
    - json
```

- `php_optional_extensions` (list, default: `[]`): Optional extensions
```yaml
  php_optional_extensions:
    - redis
    - memcached
    - imagick
```

## Repository Configuration

### Ubuntu Systems

The role uses Ondřej Surý's PPA repository for Ubuntu:
- Repository: `ppa:ondrej/php`
- Website: https://launchpad.net/~ondrej/+archive/ubuntu/php/

### Debian Systems

The role uses Ondřej Surý's Debian repository:
- Repository: `https://packages.sury.org/php/`
- Website: https://packages.sury.org/php/

### RHEL/CentOS Systems

The role uses the Remi repository:
- Repository: Remi's RPM repository
- Website: https://rpms.remirepo.net/

### Disable External Repositories

If you want to use only the default distribution repositories:
```yaml
php_enable_ppa: false
```

**Note**: Default repositories may have older PHP versions.

## Repository Setup

The role automatically:
1. Detects your operating system (Ubuntu, Debian, or RHEL/CentOS)
2. Installs required repository management tools
3. Adds the appropriate PHP repository
4. Configures GPG keys for package verification

### Manual Repository Verification

**Ubuntu:**
```bash
apt-cache policy php8.2
```

**Debian:**
```bash
apt-cache policy php8.2
cat /etc/apt/sources.list.d/php-sury.list
```

**RHEL/CentOS:**
```bash
dnf module list php
```

## Dependencies

This role has no external dependencies but requires a web server (Apache or Nginx) to be useful.

## Example Playbook

### Basic Installation
```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: php
```

### Custom Configuration
```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: php
      vars:
        php_version: "8.2"
        php_memory_limit: "512M"
        php_max_execution_time: 600
        php_fpm_pm_max_children: 10
        php_optional_extensions:
          - redis
          - imagick
          - intl
```

### High-Performance Setup
```yaml
---
- hosts: production
  become: true
  roles:
    - role: php
      vars:
        php_version: "8.3"
        php_memory_limit: "1024M"
        php_opcache_memory_consumption: 256
        php_opcache_max_accelerated_files: 10000
        php_fpm_pm_mode: dynamic
        php_fpm_pm_max_children: 50
        php_fpm_pm_start_servers: 10
        php_fpm_pm_min_spare_servers: 5
        php_fpm_pm_max_spare_servers: 15
```

## Example Host Variables

Create a file `host_vars/webserver01.yml`:
```yaml
---
php_install_enabled: true
php_version: "8.2"
php_webserver_name: apache2

# Performance tuning
php_memory_limit: "512M"
php_max_execution_time: 600
php_upload_max_filesize: "128M"
php_post_max_size: "128M"

# PHP-FPM pool configuration
php_fpm_enabled: true
php_fpm_pm_mode: dynamic
php_fpm_pm_max_children: 20
php_fpm_pm_start_servers: 5
php_fpm_pm_min_spare_servers: 3
php_fpm_pm_max_spare_servers: 8

# OPcache for better performance
php_opcache_enabled: true
php_opcache_memory_consumption: 256
php_opcache_max_accelerated_files: 10000

# Additional extensions
php_optional_extensions:
  - redis
  - imagick
  - intl
  - soap

# Security settings
php_display_errors: "Off"
php_log_errors: "On"

# Timezone
php_timezone: "Europe/Amsterdam"
```

## PHP-FPM Status and Ping Pages

This role configures PHP-FPM status and ping pages:

- **Status page**: `http://your-server/fpm-status`
- **Ping page**: `http://your-server/fpm-ping`

You need to configure your web server to proxy these URLs to PHP-FPM.

## Integration with phpMyAdmin Role

This PHP role is designed to work seamlessly with the `phpmyadmin` role:
```yaml
---
- hosts: webservers
  become: true
  roles:
    - role: php
      vars:
        php_version: "8.2"
        php_optional_extensions:
          - mbstring
          - zip
          - gd
    - role: phpmyadmin
      vars:
        phpmyadmin_url_path: phpmydinges
```

## PHP Version Switching

To switch PHP versions, change the `php_version` variable and re-run the playbook:
```yaml
php_version: "8.3"  # Switch from 8.1 to 8.3
```

## Security Considerations

1. **Disable dangerous functions**: The role disables potentially dangerous functions in PHP-FPM
2. **Open basedir**: Restricts file access to specific directories
3. **Error display**: Disabled in production (set via `php_display_errors`)
4. **Session security**: Strict mode and httponly cookies enabled
5. **OPcache**: Improves performance and security

## Troubleshooting

### Check PHP version
```bash
php -v
```

### Check PHP-FPM status
```bash
systemctl status php8.1-fpm
```

### Test PHP-FPM configuration
```bash
php-fpm8.1 -t
```

### Check loaded extensions
```bash
php -m
```

### View PHP configuration
```bash
php -i
```

## Ansible Lint

This role is fully compatible with ansible-lint:
```bash
ansible-lint roles/php
```

## Testing the Role
```bash
# Ansible lint check
ansible-lint roles/php

# Syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook playbook.yml --check

# Actual execution
ansible-playbook playbook.yml
```

## License

MIT
