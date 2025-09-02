# Ansible Roles Collection

This repository contains a collection of Ansible roles designed to automate the configuration and management of various services and systems. Each role is modular, reusable, and follows Ansible best practices to ensure reliability and ease of use in enterprise environments.

## Available Roles

- **Foomuuri**: A role for managing [Foomuuri](https://github.com/FoobarOy/foomuuri), a multi-zone bidirectional nftables firewall.  
  For detailed documentation, see [FOOMUURI.md](./FOOMUURI.md).

*Additional roles will be added to this repository over time. Check back for updates or refer to the [Issues](https://github.com/your-repo-link/issues) section for planned roles.*

## Requirements

- Ansible 2.9 or higher
- Supported OS: Debian 11+, Ubuntu 20.04+, Arch Linux (varies by role)
- Root or sudo access on target hosts

## Usage

Each role has its own documentation with specific variables, examples, and usage instructions. To use a role, include it in your playbook and configure the necessary variables. For example:

```yaml
---
- hosts: servers
  become: yes
  roles:
    - foomuuri
```

Refer to the individual role documentation (e.g., [FOOMUURI.md](./FOOMUURI.md)) for detailed configuration options.

## Directory Structure

```
ansible-roles/
├── roles/
│   ├── foomuuri/
│   │   ├── defaults/
│   │   ├── tasks/
│   │   ├── templates/
│   │   └── ...
│   └── [other-roles]/
├── FOOMUURI.md
└── README.md
```

## Contributing

Contributions are welcome! To add a new role, fix bugs, or improve documentation, please submit a pull request. For major changes, open an issue first to discuss your proposal.

## License

MIT

## Links

- [Project Repository](https://github.com/tubby1981/ansible)
- [Foomuuri GitHub](https://github.com/FoobarOy/foomuuri)
- [Issue Tracker](https://github.com/tubby1981/ansible/issues)
