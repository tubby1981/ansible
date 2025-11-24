# Quick Start Guide - MariaDB/MySQL Ansible Role

This guide will help you quickly deploy MariaDB/MySQL using this Ansible role.

## Prerequisites

1. **Install Ansible** (2.10+):
```bash
# Debian/Ubuntu
sudo apt update && sudo apt install ansible

# RHEL/AlmaLinux
sudo dnf install ansible
```

2. **Install required Ansible collections**:
```bash
ansible-galaxy collection install community.mysql
```

3. **Verify connectivity to target hosts**:
```bash
ansible all -i inventory/hosts.yml -m ping
```

## Step 1: Set Up Directory Structure

```bash
mkdir -p ansible-mysql/{roles,inventory,vault,group_vars,host_vars}
cd ansible-mysql

# Clone or copy the mariadb role
git clone <repository-url> roles/mariadb
# OR copy manually:
# cp -r /path/to/mariadb/role roles/
```

## Step 2: Create Inventory File

Create `inventory/hosts.yml`:

```yaml
---
all:
  children:
    database_servers:
      hosts:
        db01.example.com:
          ansible_host: 192.168.1.10
      
      vars:
        ansible_user: ansible
        ansible_become: yes
        ansible_python_interpreter: /usr/bin/python3
```

## Step 3: Create Vault for Passwords

```bash
# Create vault file
cat > vault/passwords.yml << 'EOF'
---
vault_mysql_root_password: "ChangeThisPassword123!"
vault_app_db_password: "AppPassword456!"
EOF

# Encrypt the vault file
ansible-vault encrypt vault/passwords.yml
# Enter vault password when prompted
```

## Step 4: Create Playbook

Create `playbook.yml`:

```yaml
---
- name: Install and Configure MariaDB
  hosts: database_servers
  become: yes
  
  vars_files:
    - vault/passwords.yml
  
  roles:
    - role: mariadb
      vars:
        # Root password from vault
        mysql_root_password: "{{ vault_mysql_root_password }}"
        
        # Allow remote connections
        mysql_bind_address: '0.0.0.0'
        
        # Create database
        mysql_databases:
          - name: myapp_db
            encoding: utf8mb4
            collation: utf8mb4_unicode_ci
        
        # Create application user
        mysql_users:
          - name: myapp_user
            host: '%'
            password: "{{ vault_app_db_password }}"
            priv: 'myapp_db.*:ALL'
```

## Step 5: Run the Playbook

```bash
# Check what would change (dry-run)
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass --check

# Apply the configuration
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass
```

## Step 6: Verify Installation

```bash
# SSH to the server
ssh ansible@db01.example.com

# Test MySQL connection
mysql -u root -p
# Enter the root password you set

# Check databases
SHOW DATABASES;

# Check users
SELECT User, Host FROM mysql.user;

# Exit MySQL
exit
```

## Common Use Cases

### Case 1: Simple Single Server

**Inventory:**
```yaml
all:
  hosts:
    mydb.local:
      ansible_host: 192.168.1.50
```

**Playbook:**
```yaml
- hosts: all
  become: yes
  roles:
    - role: mariadb
      vars:
        mysql_root_password: "SimplePassword123"
        mysql_databases:
          - name: wordpress
        mysql_users:
          - name: wpuser
            host: localhost
            password: "WpPass456"
            priv: 'wordpress.*:ALL'
```

**Run:**
```bash
ansible-playbook -i inventory/hosts.yml playbook.yml
```

### Case 2: Production Server with Backups

**host_vars/db-prod.example.com.yml:**
```yaml
mysql_root_password: "{{ vault_prod_root_password }}"
mysql_bind_address: '0.0.0.0'

# Performance tuning (8GB RAM)
mysql_innodb_buffer_pool_size: "5G"
mysql_max_connections: "300"

# Enable backups
mysql_backup_enabled: true
mysql_backup_hour: "3"
mysql_backup_minute: "0"
mysql_backup_retention_days: 14

mysql_databases:
  - name: production_app
    encoding: utf8mb4

mysql_users:
  - name: app_user
    host: '192.168.1.0/255.255.255.0'
    password: "{{ vault_app_password }}"
    priv: 'production_app.*:ALL'
```

### Case 3: Master-Slave Replication

**Master (db-master.yml):**
```yaml
mysql_server_id: "1"
mysql_replication_role: 'master'
mysql_root_password: "{{ vault_master_root_password }}"

mysql_replication_user:
  name: repl_user
  host: '%'
  password: "{{ vault_repl_password }}"

mysql_databases:
  - name: replicated_db
    replicate: 1
```

**Slave (db-slave.yml):**
```yaml
mysql_server_id: "2"
mysql_replication_role: 'slave'
mysql_replication_master: '192.168.1.10'
mysql_root_password: "{{ vault_slave_root_password }}"

mysql_replication_user:
  name: repl_user
  password: "{{ vault_repl_password }}"
```

## Useful Commands

### Working with Ansible Vault

```bash
# Create encrypted file
ansible-vault create vault/secrets.yml

# Edit encrypted file
ansible-vault edit vault/secrets.yml

# View encrypted file
ansible-vault view vault/secrets.yml

# Encrypt existing file
ansible-vault encrypt vault/passwords.yml

# Decrypt file
ansible-vault decrypt vault/passwords.yml

# Change vault password
ansible-vault rekey vault/passwords.yml

# Use vault password file (avoid typing password)
echo "MyVaultPassword" > .vault_pass
chmod 600 .vault_pass
ansible-playbook -i inventory/hosts.yml playbook.yml --vault-password-file .vault_pass
```

### Using Tags

```bash
# Only install packages
ansible-playbook playbook.yml --tags "mysql_install"

# Only update configuration
ansible-playbook playbook.yml --tags "mysql_config"

# Only manage users
ansible-playbook playbook.yml --tags "mysql_users"

# Only configure backups
ansible-playbook playbook.yml --tags "mysql_backup"

# Multiple tags
ansible-playbook playbook.yml --tags "mysql_config,mysql_users"

# Skip specific tags
ansible-playbook playbook.yml --skip-tags "mysql_backup"
```

### Limiting Execution

```bash
# Run on specific host
ansible-playbook -i inventory/hosts.yml playbook.yml --limit db01.example.com

# Run on specific group
ansible-playbook -i inventory/hosts.yml playbook.yml --limit mysql_production

# Exclude hosts
ansible-playbook -i inventory/hosts.yml playbook.yml --limit '!db02.example.com'
```

### Testing and Debugging

```bash
# Dry run (check mode)
ansible-playbook playbook.yml --check

# Show differences
ansible-playbook playbook.yml --check --diff

# Verbose output
ansible-playbook playbook.yml -v    # verbose
ansible-playbook playbook.yml -vv   # more verbose
ansible-playbook playbook.yml -vvv  # debug

# List all tasks
ansible-playbook playbook.yml --list-tasks

# List all tags
ansible-playbook playbook.yml --list-tags

# Syntax check
ansible-playbook playbook.yml --syntax-check
```

## Security Best Practices

1. **Always use Ansible Vault for passwords**:
```bash
ansible-vault encrypt_string 'MySecurePassword' --name 'mysql_root_password'
```

2. **Use strong passwords** (minimum 16 characters, mixed case, numbers, symbols)

3. **Limit bind address** where possible:
```yaml
mysql_bind_address: '127.0.0.1'  # Local only
# OR
mysql_bind_address: '192.168.1.10'  # Specific interface
```

4. **Use host-based access control** for users:
```yaml
mysql_users:
  - name: app_user
    host: '192.168.1.100'  # Specific IP
    # OR
    host: '192.168.1.0/255.255.255.0'  # Subnet
```

5. **Principle of least privilege**:
```yaml
# Good - minimal privileges
- name: readonly_user
  priv: 'mydb.*:SELECT'

# Bad - too permissive
- name: app_user
  priv: '*.*:ALL'
```

6. **Configure firewall** (external to this role):
```bash
# Example with UFW
sudo ufw allow from 192.168.1.0/24 to any port 3306
```

## Troubleshooting

### Connection Refused

```bash
# Check if MySQL is running
systemctl status mariadb

# Check if MySQL is listening
netstat -tlnp | grep mysql
ss -tlnp | grep mysql

# Check firewall
sudo ufw status
sudo firewall-cmd --list-all
```

### Authentication Failed

```bash
# Check user exists
mysql -u root -p -e "SELECT User, Host FROM mysql.user;"

# Reset root password if needed
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &
mysql -u root -e "FLUSH PRIVILEGES; ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';"
sudo systemctl restart mariadb
```

### Playbook Fails

```bash
# Run with increased verbosity
ansible-playbook playbook.yml -vvv

# Check syntax
ansible-playbook playbook.yml --syntax-check

# Verify inventory
ansible-inventory -i inventory/hosts.yml --list
```

## Next Steps

1. **Configure monitoring** - Set up Nagios, Zabbix, or Prometheus to monitor MySQL
2. **Set up backups** - Verify backup scripts are working and test restore procedures
3. **Configure replication** - Set up master-slave replication for high availability
4. **Performance tuning** - Adjust memory settings based on server resources
5. **Security hardening** - Implement SSL/TLS, review user privileges, enable audit logging

## Additional Resources

- [Official MariaDB Documentation](https://mariadb.com/kb/en/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Ansible MySQL Module Docs](https://docs.ansible.com/ansible/latest/collections/community/mysql/)
- [MySQL Performance Tuning](https://www.percona.com/blog/)

## Support

For issues, questions, or contributions, please refer to the main README.md file.
