# Ansible Role: package_manager

A robust and reusable Ansible role for managing packages on both Debian family (Debian, Ubuntu) and RedHat family (CentOS, Fedora, RHEL) Linux systems.

## Requirements

- Ansible 2.9 or higher
- Target systems must have a supported package manager (apt, yum, dnf)

## Role Variables

| Variable | Default Value | Description |
|----------|--------------|-------------|
| `package_manager_state` | `present` | Desired state of packages: `present`, `absent`, or `latest` |
| `package_manager_names` | `[]` | List of package names to manage |
| `package_manager_autoremove_enabled` | `false` | Remove obsolete packages (Debian family only) |
| `package_manager_update_cache` | `true` | Update package cache before installation |

## Example Playbooks

### Basic Installation
```yaml
---
- name: Install webserver packages
  hosts: webservers
  become: true
  roles:
    - role: package_manager
      vars:
        package_manager_names:
          - nginx
          - git
          - vim
        package_manager_state: present
```

### Update to Latest Version
```yaml
---
- name: Update system packages
  hosts: all
  become: true
  roles:
    - role: package_manager
      vars:
        package_manager_names:
          - curl
          - wget
          - openssh-server
        package_manager_state: latest
```

### Remove Packages
```yaml
---
- name: Remove unwanted packages
  hosts: production
  become: true
  roles:
    - role: package_manager
      vars:
        package_manager_names:
          - telnet
          - ftp
        package_manager_state: absent
        package_manager_autoremove_enabled: true
```

## Tags

- `package_manager` - All tasks in this role
- `validation` - Validation tasks only
- `cache_update` - Cache update tasks only
- `package_installation` - Package installation tasks only
- `autoremove` - Autoremove tasks only
- `debug` - Debug output only

### Using Tags
```bash
# Run validation only
ansible-playbook playbook.yml --tags validation

# Skip autoremove
ansible-playbook playbook.yml --skip-tags autoremove
```

## Platform Support

This role uses `ansible.builtin.package` module to provide platform-agnostic package management across:

- **Debian family**: Debian, Ubuntu (using apt)
- **RedHat family**: CentOS, RHEL, Fedora (using yum/dnf)

The `package_manager_autoremove_enabled` feature is only supported on Debian family systems. When enabled on other systems, a warning message will be displayed.

## Dependencies

None.

## License

MIT

## Author Information

This role was created in 2025 as an example of a robust, reusable Ansible role.

## Testing

To test this role with ansible-lint:

```bash
ansible-lint roles/package_manager
```

To test with a playbook:

```bash
ansible-playbook example_playbook.yml --check
```
