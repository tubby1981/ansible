# Apache Role - Complete Directory Structure

## Overview
This document describes the complete file structure of the Apache Ansible role with Let's Encrypt support.

## Directory Layout

```
roles/apache/
├── README.md                           # Main documentation
├── DIRECTORY_STRUCTURE.md              # This file
├── UPGRADE_GUIDE.md                    # Migration guide for existing installations
├── defaults/
│   └── main.yml                        # Default variables (all configurable options)
├── files/
│   └── envvars                         # Apache environment variables template
├── handlers/
│   └── main.yml                        # Apache restart handler
├── meta/
│   └── main.yml                        # Role metadata (Galaxy info)
├── tasks/
│   └── main.yml                        # Main task execution flow
├── templates/
│   ├── apache2.conf.j2                 # Main Apache configuration (NEW!)
│   ├── ports.conf.j2                   # Port listening configuration
│   ├── security.conf.j2                # Security hardening configuration
│   ├── vhost.conf.j2                   # Virtual host template
│   └── vhost-ssl.conf.j2               # SSL virtual host (DNS-01 challenge)
└── vars/
    ├── Debian.yml                      # Debian/Ubuntu specific variables
    └── RedHat.yml                      # RedHat/CentOS specific variables
```

## File Descriptions

### Root Level

#### `README.md`
- Complete role documentation
- Usage examples for HTTP-01 and DNS-01 challenges
- Configuration reference
- Troubleshooting guide
- DNS provider credential formats

#### `DIRECTORY_STRUCTURE.md`
- This file
- Overview of role architecture
- File-by-file explanation

### `defaults/main.yml`
**Purpose**: Default variable definitions (lowest precedence)

**Key sections**:
- General Apache settings
- Port and address configuration
- Apache module list
- Virtual host structure
- SSL/TLS security settings
- Let's Encrypt configuration (both HTTP-01 and DNS-01)
- DNS provider settings

**Variables**: ~30 configurable options

### `files/envvars`
**Purpose**: Apache environment variable definitions

**Contains**:
- `APACHE_RUN_USER` - Apache process user (www-data)
- `APACHE_RUN_GROUP` - Apache process group (www-data)
- `APACHE_PID_FILE` - PID file location
- `APACHE_RUN_DIR` - Runtime directory (/var/run/apache2)
- `APACHE_LOCK_DIR` - Lock file directory
- `APACHE_LOG_DIR` - Log directory

**Why needed**: Fixes `${APACHE_RUN_DIR} is not defined` error

### `handlers/main.yml`
**Purpose**: Event handlers triggered by task notifications

**Handlers**:
- `restart apache` - Restarts Apache service after configuration changes

**Usage**: Tasks notify this handler with `notify: restart apache`

### `meta/main.yml`
**Purpose**: Role metadata for Ansible Galaxy

**Contains**:
- Author information
- Role description
- License (MIT)
- Minimum Ansible version (2.9)
- Supported platforms (Ubuntu, Debian)
- Galaxy tags (web, apache, ssl, letsencrypt)

### `tasks/main.yml`
**Purpose**: Main task orchestration

**Task flow** (9 major sections):
1. **OS-specific variables** - Load Debian.yml or RedHat.yml
2. **Package installation** - Install Apache package
3. **Environment setup** - Deploy envvars, create directories
4. **Configuration deployment** - ports.conf, security.conf
5. **Module management** - Enable required Apache modules
6. **Virtual host setup** - Deploy and enable vhost configs
7. **Configuration validation** - Test Apache config syntax
8. **Apache startup** - Ensure Apache is running (HTTP-01)
9. **Let's Encrypt** - Certificate request and renewal setup
   - HTTP-01 pre-flight checks (DNS validation)
   - DNS-01 pre-flight checks (credentials validation)
   - Certificate obtainment
   - Auto-renewal configuration

**Total tasks**: ~40 tasks with conditional execution

### `templates/apache2.conf.j2`
**Purpose**: Main Apache server configuration file

**Replaces**: `/etc/apache2/apache2.conf`

**Key features**:
- **No environment variable dependencies** (fixes `${APACHE_RUN_DIR}` error)
- Uses direct paths instead of `${VARIABLE}` syntax
- Fully Ansible-managed configuration
- Automatic backup of original on first deployment

**Contains**:
- ServerName directive
- ServerRoot path
- DefaultRuntimeDir (direct path, not `${APACHE_RUN_DIR}`)
- Mutex configuration (direct path, not `${APACHE_LOCK_DIR}`)
- PidFile location (direct path, not `${APACHE_PID_FILE}`)
- Timeout settings
- KeepAlive configuration
- User/Group settings
- Log configuration
- Module includes
- Security defaults
- Virtual host includes
- Optional FreeIPA LDAP authentication

**Variables used**:
- `apache_server_name` - ServerName (defaults to `ansible_fqdn`)
- `apache_config_dir` - Apache config directory
- `apache_runtime_dir` - Runtime directory path
- `apache_lock_dir` - Lock directory path
- `apache_pid_file` - PID file path
- `apache_log_dir` - Log directory path
- `apache_run_user` - Process user
- `apache_run_group` - Process group
- `apache_timeout` - Request timeout
- `apache_keepalive` - KeepAlive setting
- `apache_max_keepalive_requests` - Max requests per connection
- `apache_keepalive_timeout` - KeepAlive timeout
- `apache_log_level` - Log verbosity
- `apache_document_root` - Default document root
- `apache_enable_freeipa_auth` - Enable FreeIPA LDAP auth

**Why this is important**:
- Solves `AH00111: Config variable ${APACHE_RUN_DIR} is not defined` error
- Allows Apache to start directly without sourcing envvars
- Makes configuration explicit and version-controlled
- Prevents environment variable dependency issues

### `templates/ports.conf.j2`
**Purpose**: Apache port listening configuration

**Generates**:
```apache
Listen 80
Listen 443

<IfModule ssl_module>
    Listen 443
</IfModule>
```

**Variables used**:
- `apache_listen_ports` - HTTP ports
- `apache_ssl_ports` - HTTPS ports
- `apache_listen_addresses` - Bind addresses

### `templates/security.conf.j2`
**Purpose**: Security hardening configuration

**Contains**:
- Server token configuration (`ServerTokens Prod`)
- Server signature (`ServerSignature Off`)
- HTTP TRACE disable (`TraceEnable Off`)
- Security headers:
  - X-Frame-Options (clickjacking protection)
  - X-Content-Type-Options (MIME sniffing protection)
  - X-XSS-Protection
  - Referrer-Policy
- Directory restrictions
- Sensitive file protection (.git, dotfiles)

**Variables used**:
- `apache_server_tokens`
- `apache_server_signature`
- `apache_trace_enable`

### `templates/vhost.conf.j2`
**Purpose**: Virtual host configuration template

**Generates**:
1. **HTTP VirtualHost** (port 80)
   - ServerName and ServerAlias
   - DocumentRoot
   - Directory permissions
   - Error/access logs
   - Optional HTTP→HTTPS redirect (manual SSL)
   - Optional custom rewrite rules (when not using Let's Encrypt)

2. **HTTPS VirtualHost** (port 443) - Only for manual SSL
   - SSL engine configuration
   - Certificate paths (commented, requires manual setup)
   - Modern TLS configuration
   - HSTS header
   - Security headers

3. **Let's Encrypt placeholder** - When `with_letsencrypt: true`
   - HTTP-01: Certbot modifies this file automatically
   - DNS-01: Separate SSL config created via vhost-ssl.conf.j2

**Variables used**:
- All vhost-specific variables from `apache_vhosts[]`
- `apache_cipher_suite`
- `apache_document_root`

### `templates/vhost-ssl.conf.j2`
**Purpose**: SSL virtual host for DNS-01 challenge

**When used**: Only when `apache_certbot_challenge_method: "dns"`

**Generates**:
- HTTPS VirtualHost (port 443)
- Let's Encrypt certificate paths
- Modern SSL/TLS configuration
- Security headers
- HSTS
- Optional custom rewrite rules

**Why separate**: DNS-01 doesn't modify existing configs like HTTP-01 does

### `vars/Debian.yml`
**Purpose**: Debian/Ubuntu specific variables (higher precedence than defaults)

**Variables**:
- `apache_package_name: "apache2"`
- `apache_service_name: "apache2"`
- `apache_config_dir: "/etc/apache2"`
- `apache_sites_available_dir: "/etc/apache2/sites-available"`
- `apache_sites_enabled_dir: "/etc/apache2/sites-enabled"`
- `apache_modules_path: "/etc/apache2/mods-enabled"`

### `vars/RedHat.yml`
**Purpose**: RedHat/CentOS/Fedora specific variables

**Variables**:
- `apache_package_name: "httpd"`
- `apache_service_name: "httpd"`
- `apache_config_dir: "/etc/httpd"`
- `apache_sites_available_dir: "/etc/httpd/sites-available"`
- `apache_sites_enabled_dir: "/etc/httpd/sites-enabled"`
- `apache_modules_path: "/etc/httpd/conf.modules.d"`

## Variable Precedence

Ansible variable precedence (lowest to highest):
1. `defaults/main.yml` - Role defaults
2. `vars/Debian.yml` or `vars/RedHat.yml` - OS-specific
3. `group_vars/` - Group variables
4. `host_vars/` - Host-specific variables
5. Playbook variables
6. Command-line extra vars (`-e`)

## Task Execution Flow

```
1. Load OS-specific variables (Debian.yml or RedHat.yml)
2. Install Apache package
3. Deploy environment configuration
   ├── Create envvars file
   ├── Create runtime directory
   └── Create lock directory
4. Deploy configuration files
   ├── ports.conf
   └── security.conf
5. Enable Apache modules (ssl, rewrite, headers, etc.)
6. Create document root directories
7. Deploy virtual host configurations
8. Enable virtual hosts (symlink)
9. Validate Apache configuration
10. Start Apache service
11. Let's Encrypt provisioning (if enabled)
    ├── Install Certbot + plugins
    ├── Verify configuration (email, DNS, credentials)
    ├── Run pre-flight checks
    ├── Request certificates
    └── Setup auto-renewal
```

## Conditional Execution

### OS Family Checks
Many tasks only run on specific OS families:
```yaml
when:
  - ansible_os_family == "Debian"
```

### Let's Encrypt Conditionals
Tasks execute based on challenge method:
```yaml
when:
  - apache_certbot_enabled
  - apache_certbot_challenge_method == "http"
```

### Virtual Host Conditionals
VHost tasks respect enable flags:
```yaml
when:
  - apache_enabled
  - (item.enabled | default(true))
  - (item.with_letsencrypt | default(false))
```

## Tags

Use tags for selective execution:

| Tag | Purpose |
|-----|---------|
| `apache` | All Apache tasks |
| `installation` | Package installation only |
| `configuration` | Config file deployment |
| `ports` | Port configuration |
| `security` | Security settings |
| `modules` | Module management |
| `vhosts` | Virtual host setup |
| `validation` | Config syntax checking |
| `certbot` | All Certbot tasks |
| `letsencrypt` | Let's Encrypt operations |
| `certificates` | Certificate requests |
| `renewal` | Auto-renewal setup |
| `http-01` | HTTP-01 specific tasks |
| `dns-01` | DNS-01 specific tasks |
| `preflight` | Pre-flight checks |

## Dependencies

### System Dependencies
- Apache2 or httpd package
- Python 3
- Certbot
- Certbot Apache plugin (HTTP-01)
- Certbot DNS plugins (DNS-01)

### Ansible Dependencies
- `community.general` collection (for `apache2_module`)
- Ansible 2.9+

### No External Role Dependencies
This role is self-contained and has no role dependencies.

## File Generation

When the role runs, it creates these files on target systems:

### Configuration Files
```
/etc/apache2/
├── envvars                                    # From files/envvars
├── ports.conf                                 # From templates/ports.conf.j2
├── conf-available/security.conf               # From templates/security.conf.j2
├── conf-enabled/security.conf                 # Symlink
├── sites-available/
│   ├── www.example.com.conf                  # From templates/vhost.conf.j2
│   └── www.example.com-le-ssl.conf           # From templates/vhost-ssl.conf.j2 (DNS-01)
└── sites-enabled/
    ├── www.example.com.conf                  # Symlink
    └── www.example.com-le-ssl.conf           # Symlink (DNS-01)
```

### Runtime Files
```
/var/run/apache2/
├── apache2.pid                                # Apache PID file
└── ...

/var/lock/apache2/                             # Lock files
```

### Let's Encrypt Files
```
/etc/letsencrypt/
├── accounts/                                  # Certbot account info
├── archive/
│   └── example.com/
│       ├── cert1.pem
│       ├── chain1.pem
│       ├── fullchain1.pem
│       └── privkey1.pem
├── live/
│   └── example.com/                          # Symlinks to archive/
│       ├── cert.pem
│       ├── chain.pem
│       ├── fullchain.pem
│       └── privkey.pem
└── renewal/
    └── example.com.conf                      # Renewal configuration
```

### DNS-01 Credentials
```
/root/.secrets/certbot/
├── cloudflare.ini                            # If using Cloudflare
├── route53.ini                               # If using Route53
└── digitalocean.ini                          # If using DigitalOcean
```

## Security Considerations

### File Permissions
- Configuration files: `0644` (readable by all)
- Envvars: `0644`
- DNS credentials: `0600` (root only)
- Directories: `0755`
- Document roots: `0755` owned by www-data

### Sensitive Data Handling
- DNS credentials never logged (`no_log: true`)
- Credentials stored with restricted permissions
- Environment variables not exposed in logs

### Security Defaults
- Server tokens minimal (`Prod`)
- Server signature disabled
- HTTP TRACE disabled
- Modern TLS only (1.2+)
- Strong cipher suites
- Security headers enabled

## Maintenance

### Adding New DNS Provider

1. Update `defaults/main.yml`:
   ```yaml
   apache_certbot_dns_packages:
     newprovider: "python3-certbot-dns-newprovider"
   ```

2. Document credentials format in README.md

3. Test with staging environment

### Adding New OS Family

1. Create `vars/NewFamily.yml`
2. Define OS-specific paths
3. Update `tasks/main.yml` conditionals
4. Test package names and paths

### Updating Security Defaults

1. Update cipher suite in `defaults/main.yml`
2. Modify `templates/security.conf.j2`
3. Test SSL configuration with SSL Labs

## Testing

### Syntax Validation
```bash
ansible-playbook site.yml --syntax-check
```

### Dry Run
```bash
ansible-playbook site.yml --check
```

### Specific Tags
```bash
ansible-playbook site.yml --tags validation
```

### Debug Mode
```bash
ansible-playbook site.yml -vvv
```

## Version Control

### Recommended .gitignore
```gitignore
# Sensitive files
files/envvars.local
**/secrets/
*.key
*.pem

# Ansible artifacts
*.retry
.ansible/

# Editor files
.vscode/
.idea/
*.swp
```

### Branching Strategy
- `main` - Stable releases
- `develop` - Active development
- `feature/*` - New features
- `fix/*` - Bug fixes

## Support Resources

- **Ansible Docs**: https://docs.ansible.com/
- **Apache Docs**: https://httpd.apache.org/docs/
- **Let's Encrypt**: https://letsencrypt.org/docs/
- **Certbot**: https://eff-certbot.readthedocs.io/

## Change Log

See `README.md` for version history and changes.
