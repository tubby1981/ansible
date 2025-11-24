# Ansible Role: MariaDB/MySQL

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Production-ready Ansible role for installing and configuring MariaDB or MySQL on Linux servers. Supports multiple distributions including Debian, Ubuntu, AlmaLinux, Rocky Linux, and RHEL.

## Features

- ✅ Multi-distribution support (Debian 10-13, Ubuntu 20.04-24.04, AlmaLinux 8-9, RHEL 7-9)
- ✅ Automated secure installation
- ✅ Database and user management
- ✅ Master-slave replication support
- ✅ Automated backup with retention policy
- ✅ Performance tuning templates
- ✅ Custom configuration file support
- ✅ Idempotent operations

## Supported Platforms

### Debian Family
- Debian 10 (Buster)
- Debian 11 (Bullseye)
- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 20.04 LTS (Focal Fossa)
- Ubuntu 22.04 LTS (Jammy Jellyfish)
- Ubuntu 24.04 LTS (Noble Numbat)

### RedHat Family
- RHEL 7, 8, 9
- AlmaLinux 8, 9
- Rocky Linux 8, 9
- CentOS 7, 8 (deprecated)

## Requirements

- Ansible 2.10 or higher
- Python 3.6 or higher on target hosts
- Root or sudo access on target hosts

### Required Ansible Collections
```bash
ansible-galaxy collection install community.mysql
```

## Role Variables

### Basic Configuration

```yaml
# MySQL root credentials
mariadb_root_username: root
mariadb_root_password: "SecurePassword123"
mariadb_root_password_update: false

# Service configuration
mariadb_enabled_on_startup: true
mariadb_overwrite_global_mycnf: true
```

### Network Configuration

```yaml
mariadb_port: "3306"
mariadb_bind_address: '127.0.0.1'  # Use 0.0.0.0 for external access
mariadb_skip_name_resolve: false
mariadb_datadir: /var/lib/mysql
```

### Performance Tuning

```yaml
# Memory settings (adjust based on available RAM)
mariadb_key_buffer_size: "256M"
mariadb_max_allowed_packet: "64M"
mariadb_innodb_buffer_pool_size: "1G"
mariadb_innodb_log_file_size: "256M"
mariadb_max_connections: "300"
mariadb_tmp_table_size: "256M"
```

### Database Management

```yaml
mariadb_databases:
  - name: myapp_production
    collation: utf8mb4_general_ci
    encoding: utf8mb4
    state: present

  - name: myapp_staging
    collation: utf8mb4_general_ci
    encoding: utf8mb4
    state: present
```

### User Management

```yaml
mariadb_users:
  - name: myapp_user
    host: localhost
    password: "AppPassword123"
    priv: 'myapp_production.*:ALL'
    state: present

  - name: readonly_user
    host: '%'
    password: "ReadOnlyPass123"
    priv: '*.*:SELECT'
    state: present

  - name: backup_user
    host: '10.0.0.0/255.255.255.0'
    password: "BackupPass123"
    priv: '*.*:SELECT,LOCK TABLES,RELOAD'
    state: present
```

### Backup Configuration

```yaml
mariadb_backup_enabled: true
mariadb_backup_hour: "3"
mariadb_backup_minute: "30"
mariadb_backup_retention_days: 7
mariadb_backup_dir: /root/backups/mysql
```

### Replication Configuration

```yaml
# Master configuration
mariadb_replication_role: 'master'
mariadb_server_id: "1"
mariadb_max_binlog_size: "100M"
mariadb_binlog_format: "ROW"
mariadb_expire_logs_days: "10"

mariadb_replication_user:
  name: repl_user
  host: '%'
  password: "ReplPassword123"
  priv: '*.*:REPLICATION SLAVE,REPLICATION CLIENT'
```

```yaml
# Slave configuration
mariadb_replication_role: 'slave'
mariadb_server_id: "2"
mariadb_replication_master: "192.168.1.10"

mariadb_replication_user:
  name: repl_user
  password: "ReplPassword123"
```

### Logging Configuration

```yaml
mariadb_slow_query_log_enabled: true
mariadb_slow_query_time: "2"
mariadb_log_error: /var/log/mysql/error.log
```

## Dependencies

None.

## Example Playbooks

### Basic Installation

```yaml
---
- name: Install MariaDB/MySQL
  hosts: database_servers
  become: yes
  
  roles:
    - role: mariadb
      vars:
        mariadb_root_password: "YourSecurePassword"
        mariadb_bind_address: '0.0.0.0'
        
        mariadb_databases:
          - name: production_db
            encoding: utf8mb4
            collation: utf8mb4_unicode_ci
        
        mariadb_users:
          - name: app_user
            host: '%'
            password: "AppPassword123"
            priv: 'production_db.*:ALL'
```

### Production Setup with Replication

```yaml
---
- name: Configure MySQL Master
  hosts: mariadb_master
  become: yes
  
  roles:
    - role: mariadb
      vars:
        mariadb_root_password: "MasterRootPassword"
        mariadb_bind_address: '0.0.0.0'
        mariadb_replication_role: 'master'
        mariadb_server_id: "1"
        
        mariadb_databases:
          - name: app_db
            encoding: utf8mb4
            replicate: 1
        
        mariadb_replication_user:
          name: repl_user
          host: '%'
          password: "ReplicationPassword"

---
- name: Configure MySQL Slave
  hosts: mariadb_slave
  become: yes
  
  roles:
    - role: mariadb
      vars:
        mariadb_root_password: "SlaveRootPassword"
        mariadb_bind_address: '0.0.0.0'
        mariadb_replication_role: 'slave'
        mariadb_server_id: "2"
        mariadb_replication_master: "192.168.1.10"
        
        mariadb_replication_user:
          name: repl_user
          password: "ReplicationPassword"
```

### High-Performance Setup

```yaml
---
- name: Install High-Performance MySQL
  hosts: mariadb_production
  become: yes
  
  roles:
    - role: mariadb
      vars:
        mariadb_root_password: "ProductionPassword"
        mariadb_bind_address: '0.0.0.0'
        
        # Performance tuning for 16GB RAM server
        mariadb_innodb_buffer_pool_size: "10G"
        mariadb_innodb_log_file_size: "2G"
        mariadb_max_connections: "500"
        mariadb_thread_cache_size: "32"
        mariadb_table_open_cache: "8192"
        mariadb_tmp_table_size: "512M"
        mariadb_max_heap_table_size: "512M"
        
        # Enable slow query logging
        mariadb_slow_query_log_enabled: true
        mariadb_slow_query_time: "1"
        
        # Backup configuration
        mariadb_backup_enabled: true
        mariadb_backup_hour: "2"
        mariadb_backup_retention_days: 14
```

## Directory Structure

```
roles/mariadb/
├── defaults/
│   └── main.yml              # Default variables
├── handlers/
│   └── main.yml              # Service restart handler
├── tasks/
│   ├── main.yml              # Main task orchestration
│   ├── variables.yml         # OS-specific variable loading
│   ├── setup-Debian.yml      # Debian family installation
│   ├── setup-RedHat.yml      # RedHat family installation
│   ├── configure.yml         # Configuration tasks
│   ├── secure-installation.yml  # Security hardening
│   ├── databases.yml         # Database management
│   ├── users.yml             # User management
│   ├── replication.yml       # Replication setup
│   └── backup-databases.yml  # Backup configuration
├── templates/
│   ├── my.cnf.j2            # Main configuration template
│   ├── root-my.cnf.j2       # Root user credentials
│   ├── user-my.cnf.j2       # Application user credentials
│   └── backup_database.sh.j2 # Backup script
├── vars/
│   ├── Debian.yml           # Debian/Ubuntu variables
│   ├── Debian-10.yml
│   ├── Debian-11.yml
│   ├── Debian-12.yml
│   ├── Debian-13.yml
│   ├── Debian-24.yml        # Ubuntu 24.04
│   ├── RedHat.yml           # RedHat family variables
│   ├── RedHat-7.yml
│   ├── RedHat-8.yml
│   └── RedHat-9.yml
└── README.md
```

## Usage in Inventory

### Host Variables Example

```yaml
# host_vars/db01.example.com.yml
---
mariadb_root_password: "{{ vault_mariadb_root_password }}"
mariadb_bind_address: '0.0.0.0'
mariadb_server_id: "1"

mariadb_innodb_buffer_pool_size: "8G"
mariadb_max_connections: "400"

mariadb_databases:
  - name: wordpress_db
    encoding: utf8mb4
    collation: utf8mb4_unicode_ci

mariadb_users:
  - name: wordpress
    host: '192.168.1.0/255.255.255.0'
    password: "{{ vault_wordpress_db_password }}"
    priv: 'wordpress_db.*:ALL'
```

### Group Variables Example

```yaml
# group_vars/database_servers.yml
---
mariadb_enabled_on_startup: true
mariadb_backup_enabled: true
mariadb_backup_hour: "3"
mariadb_backup_retention_days: 7

mariadb_slow_query_log_enabled: true
mariadb_slow_query_time: "2"

# Common performance settings
mariadb_max_connections: "300"
mariadb_innodb_buffer_pool_size: "2G"
```

## Tags

This role supports the following tags for selective execution:

- `mysql` - All tasks
- `mariadb_install` - Installation tasks only
- `mariadb_config` - Configuration tasks only
- `mariadb_secure` - Security hardening only
- `mariadb_databases` - Database management only
- `mariadb_users` - User management only
- `mariadb_replication` - Replication setup only
- `mariadb_backup` - Backup configuration only

### Example Tag Usage

```bash
# Install and configure MySQL
ansible-playbook playbook.yml --tags "mariadb_install,mariadb_config"

# Only update user passwords
ansible-playbook playbook.yml --tags "mariadb_users"

# Configure backups only
ansible-playbook playbook.yml --tags "mariadb_backup"
```

## Security Considerations

1. **Password Management**: Always use Ansible Vault for storing passwords
   ```bash
   ansible-vault encrypt_string 'YourPassword' --name 'mariadb_root_password'
   ```

2. **Network Access**: Set `mariadb_bind_address` appropriately:
   - `127.0.0.1` - Local access only (most secure)
   - `0.0.0.0` - All interfaces (use with firewall rules)
   - Specific IP - Bind to specific interface

3. **Firewall**: Configure firewall rules separately:
   ```yaml
   # Example with UFW
   - name: Allow MySQL from specific network
     ufw:
       rule: allow
       port: 3306
       proto: tcp
       from_ip: 192.168.1.0/24
   ```

4. **User Privileges**: Follow principle of least privilege when creating users

## Performance Tuning Guidelines

### Memory Allocation (based on available RAM)

#### 2GB RAM Server
```yaml
mariadb_innodb_buffer_pool_size: "512M"
mariadb_innodb_log_file_size: "128M"
mariadb_max_connections: "150"
```

#### 4GB RAM Server
```yaml
mariadb_innodb_buffer_pool_size: "2G"
mariadb_innodb_log_file_size: "512M"
mariadb_max_connections: "200"
```

#### 8GB RAM Server
```yaml
mariadb_innodb_buffer_pool_size: "5G"
mariadb_innodb_log_file_size: "1G"
mariadb_max_connections: "300"
```

#### 16GB+ RAM Server
```yaml
mariadb_innodb_buffer_pool_size: "10G"
mariadb_innodb_log_file_size: "2G"
mariadb_max_connections: "500"
```

## Troubleshooting

### Service Won't Start

```bash
# Check MariaDB/MySQL service status
systemctl status mariadb

# Check error log
tail -f /var/log/mysql/error.log

# Test configuration syntax
mysqld --help --verbose
```

### Connection Issues

```bash
# Test local connection
mysql -u root -p

# Test remote connection
mysql -h 192.168.1.10 -u username -p

# Check listening ports
netstat -tlnp | grep mysql
```

### Replication Issues

```bash
# On master - check binary log status
mysql -u root -p -e "SHOW MASTER STATUS\G"

# On slave - check replication status
mysql -u root -p -e "SHOW SLAVE STATUS\G"
```

## Backup and Restore

### Manual Backup

```bash
# Run backup script manually
/usr/local/bin/mysql-backup.sh

# Backups are stored in
ls -lh /root/backups/mysql/
```

### Restore from Backup

```bash
# Decompress and restore
gunzip < backup_file.sql.gz | mysql -u root -p database_name
```

## License

MIT

## Author Information

This role was created for production use in enterprise Linux environments.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Changelog

### Version 2.0.0 (2024)
- Added support for Debian 13, Ubuntu 24.04, AlmaLinux 9
- Modernized variable structure
- Improved backup script with error handling
- Added comprehensive logging
- Enhanced security defaults
- Added performance tuning templates
- Improved documentation

### Version 1.0.0
- Initial release with basic MariaDB/MySQL support
