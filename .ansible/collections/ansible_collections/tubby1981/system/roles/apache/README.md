# Ansible Role: Apache with Let's Encrypt

A comprehensive Ansible role for deploying and managing Apache web server with automatic Let's Encrypt SSL certificate provisioning. Supports both HTTP-01 and DNS-01 ACME challenges.

## Features

- ✅ **Automated SSL**: Let's Encrypt certificate provisioning and renewal
- 🔒 **Security Hardened**: Modern TLS configuration, security headers, and best practices
- 🌐 **Two Challenge Methods**: HTTP-01 (default) and DNS-01 (for wildcards)
- 🔄 **Auto-Renewal**: Automatic certificate renewal via systemd timer or cron
- 📋 **Pre-flight Checks**: DNS validation before certificate requests
- 🎯 **Multi-Domain Support**: Single certificate for multiple domains/aliases
- 🔧 **Flexible Configuration**: Override per-host or per-group
- 📦 **Cross-Platform**: Ubuntu, Debian, CentOS, RHEL support

## Requirements

- **Ansible**: 2.9 or higher
- **Target OS**: Ubuntu 20.04+, Debian 10+, CentOS 8+, RHEL 8+
- **Certbot**: Automatically installed by role
- **Python 3**: Required for Certbot

### HTTP-01 Challenge Requirements
- Port 80 must be accessible from the internet
- DNS A-records must point to the target server
- Apache must be able to serve `.well-known/acme-challenge/` directory

### DNS-01 Challenge Requirements
- DNS provider API credentials (Cloudflare, Route53, DigitalOcean, etc.)
- Appropriate Certbot DNS plugin installed (handled by role)
- API token/key with DNS zone modification permissions

## Installation

### From Ansible Galaxy (recommended)
```bash
ansible-galaxy collection install tubby1981.system
```

### From Git Repository
```bash
# Clone the repository
git clone https://github.com/tubby1981/ansible.git
cd ansible

# Install the collection locally
ansible-galaxy collection build
ansible-galaxy collection install tubby1981-system-*.tar.gz --force

# Run test playbook
ansible-playbook playbook/apache//your_host.yml -i tests/inventory
```

### Manual Installation
```bash
mkdir -p roles/apache
# Copy role files to roles/apache/
```

## Quick Start

### Basic HTTP-01 Setup (Easiest)

```yaml
# group_vars/webservers.yml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"
apache_certbot_challenge_method: "http"  # Default

apache_vhosts:
  - server_name: "www.example.com"
    document_root: "/var/www/example"
    ssl: true
    with_letsencrypt: true
    aliases:
      - "example.com"
```

### Playbook
```yaml
- hosts: webservers
  become: true
  roles:
    - apache
```

## Configuration

### Global Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_enabled` | `true` | Enable/disable the role |
| `apache_document_root` | `/var/www/html` | Default document root |
| `apache_server_name` | `ansible_fqdn` | ServerName directive |
| `apache_listen_ports` | `[80]` | HTTP listening ports |
| `apache_ssl_ports` | `[]` | HTTPS listening ports |
| `apache_listen_addresses` | `['*']` | IP addresses to bind |
| `apache_server_tokens` | `Prod` | Server token verbosity |
| `apache_server_signature` | `Off` | Show server signature |
| `apache_trace_enable` | `Off` | HTTP TRACE method |

### Apache Runtime Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_runtime_dir` | `/var/run/apache2` | Runtime directory for PID files |
| `apache_lock_dir` | `/var/lock/apache2` | Lock file directory |
| `apache_pid_file` | `/var/run/apache2/apache2.pid` | PID file location |
| `apache_log_dir` | `/var/log/apache2` | Log directory |
| `apache_run_user` | `www-data` | Apache process user |
| `apache_run_group` | `www-data` | Apache process group |

### Performance Tuning

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_timeout` | `60` | Request timeout in seconds |
| `apache_keepalive` | `On` | Enable persistent connections |
| `apache_max_keepalive_requests` | `100` | Max requests per connection |
| `apache_keepalive_timeout` | `5` | Keepalive timeout in seconds |
| `apache_log_level` | `warn` | Log verbosity level |

### Let's Encrypt Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_certbot_enabled` | `false` | Enable Let's Encrypt |
| `apache_certbot_email` | `""` | Email for cert notifications (required) |
| `apache_certbot_challenge_method` | `http` | Challenge type: `http` or `dns` |
| `apache_certbot_auto_renew` | `true` | Enable automatic renewal |
| `apache_certbot_strict_dns_check` | `true` | Fail if DNS not configured (HTTP-01) |

### HTTP-01 Challenge Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_certbot_package` | `certbot` | Certbot package name |
| `apache_certbot_apache_plugin` | `python3-certbot-apache` | Apache plugin package |

### DNS-01 Challenge Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_certbot_dns_provider` | `""` | DNS provider (e.g., `cloudflare`) |
| `apache_certbot_dns_credentials_file` | `""` | Path to credentials file |
| `apache_certbot_dns_credentials_content` | `""` | Credentials content (written securely) |
| `apache_certbot_dns_propagation_seconds` | `60` | DNS propagation wait time |

### Virtual Host Configuration

```yaml
apache_vhosts:
  - server_name: "example.com"           # Primary domain (required)
    document_root: "/var/www/example"    # Website files location
    port: 80                              # HTTP port
    ssl: true                             # Enable HTTPS
    ssl_port: 443                         # HTTPS port
    redirect_https: true                  # Redirect HTTP → HTTPS
    aliases:                              # Additional domains
      - "www.example.com"
      - "alias.example.com"
    enabled: true                         # Enable/disable vhost
    with_letsencrypt: true                # Request Let's Encrypt cert
    rewrite_rules:                        # Custom Apache rewrites (optional)
      - "RewriteEngine on"
      - "RewriteRule ^/old$ /new [R=301,L]"
```

### URL Protection (Access Control per Path)

The role provides fine-grained access control for specific URLs or directories via the `url_protection` option inside each VirtualHost.  
This is useful for restricting admin areas (e.g., `/wp-admin`), private directories, or custom application endpoints to specific IPv4/IPv6 addresses.

Each entry generates an Apache `<Location>` block with appropriate `Require` rules.

#### Example

```yaml
apache_vhosts:
  - server_name: "example.com"
    document_root: "/var/www/example"

    url_protection:
      - path: "/wp-admin"
        allowed_ips:
          - "203.0.113.10"
          - "127.0.0.1"
          - "::1"

      # Public override (e.g., required by WordPress plugins)
      - path: "/wp-admin/admin-ajax.php"
        allowed_ips:
          - "all"

      - path: "/private"
        allowed_ips:
          - "192.168.1.0/24"
          - "2001:db8::/64"

#### Behavior

All protected paths default to:

```apache
Require all denied

Allowed IPs are added as:

```apache
Require ip <allowed_ip>

When allowed_ips contains only "all", access is unrestricted:

```apache
Require all granted

Supports:
* IPv4
* IPv6
* CIDR ranges
* Application-required overrides (e.g., WordPress AJAX)

Typical Use Cases
* Restricting WordPress /wp-admin
* Allowing /wp-admin/admin-ajax.php for plugins (e.g., WordFence, WooCommerce)
* Securing private directories (/private, /admin, /internal)
* Protecting API routes
* Limiting monitoring or debug endpoints to internal IP ranges


## Usage Examples

### Example 1: Single Domain with HTTP-01

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"

apache_vhosts:
  - server_name: "blog.example.com"
    document_root: "/var/www/blog"
    ssl: true
    with_letsencrypt: true
```

### Example 2: Multiple Domains with Aliases

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"

apache_vhosts:
  - server_name: "www.debron-urk.nl"
    document_root: "/var/www/debron"
    ssl: true
    aliases:
      - "debron-urk.nl"
      - "www.hervormdegemeente-debron.nl"
      - "hervormdegemeente-debron.nl"
    with_letsencrypt: true
```

### Example 3: Wildcard Certificate with DNS-01 (Cloudflare)

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"
apache_certbot_challenge_method: "dns"
apache_certbot_dns_provider: "cloudflare"
apache_certbot_dns_credentials_file: "/root/.secrets/certbot/cloudflare.ini"
apache_certbot_dns_credentials_content: |
  # Cloudflare API credentials
  dns_cloudflare_email = admin@example.com
  dns_cloudflare_api_key = your-global-api-key

apache_vhosts:
  - server_name: "example.com"
    document_root: "/var/www/example"
    ssl: true
    aliases:
      - "*.example.com"  # Wildcard support!
    with_letsencrypt: true
```

### Example 4: Manual SSL Certificates (No Let's Encrypt)

```yaml
apache_certbot_enabled: false

apache_vhosts:
  - server_name: "secure.example.com"
    document_root: "/var/www/secure"
    ssl: true
    redirect_https: true
    with_letsencrypt: false
    # Note: You must manually configure SSL certificate paths
    # in /etc/apache2/sites-available/secure.example.com.conf
```

### Example 5: Multiple Sites with Mixed Configurations

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"

apache_vhosts:
  # Site 1: Let's Encrypt SSL
  - server_name: "www.site1.com"
    document_root: "/var/www/site1"
    ssl: true
    with_letsencrypt: true
    aliases:
      - "site1.com"

  # Site 2: HTTP only (development)
  - server_name: "dev.site2.com"
    document_root: "/var/www/site2"
    ssl: false
    with_letsencrypt: false

  # Site 3: Manual SSL certificates
  - server_name: "custom.site3.com"
    document_root: "/var/www/site3"
    ssl: true
    with_letsencrypt: false
```

## DNS-01 Challenge Setup

### Supported DNS Providers

- **Cloudflare** (`cloudflare`)
- **Amazon Route 53** (`route53`)
- **DigitalOcean** (`digitalocean`)
- **Google Cloud DNS** (`google`)
- **OVH** (`ovh`)
- And many more...

See [Certbot DNS Plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins) for the full list.

### Cloudflare Credentials File

Create `/root/.secrets/certbot/cloudflare.ini`:
```ini
# Cloudflare API credentials
dns_cloudflare_email = your-cloudflare-email@example.com
dns_cloudflare_api_key = your-global-api-key
```

Or use API token (recommended):
```ini
# Cloudflare API token (more secure)
dns_cloudflare_api_token = your-api-token
```

**Permissions**: `chmod 600 /root/.secrets/certbot/cloudflare.ini`

### Route53 Credentials File

Create `/root/.secrets/certbot/route53.ini`:
```ini
# AWS credentials
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

### DigitalOcean Credentials File

Create `/root/.secrets/certbot/digitalocean.ini`:
```ini
# DigitalOcean API token
dns_digitalocean_token = your-digitalocean-api-token
```

### Using Credentials Content (More Secure)

Instead of manually creating files, provide credentials in variables:

```yaml
apache_certbot_dns_credentials_content: |
  dns_cloudflare_email = admin@example.com
  dns_cloudflare_api_key = {{ vault_cloudflare_api_key }}
```

This is securely written with `0600` permissions and `no_log: true`.

## Pre-flight Checks

### HTTP-01 Challenge Checks

The role automatically verifies:
- ✅ DNS A-records point to the target server
- ✅ All domains (primary + aliases) are resolvable
- ⚠️ Warnings for missing DNS records
- ❌ Playbook fails if `apache_certbot_strict_dns_check: true`

To skip DNS validation:
```yaml
apache_certbot_strict_dns_check: false
```

### DNS-01 Challenge Checks

The role verifies:
- ✅ DNS provider is specified
- ✅ Credentials file path is configured
- ✅ Credentials content or file exists

## Certificate Renewal

### Automatic Renewal (Systemd)

On systems with systemd, `certbot.timer` is enabled:
```bash
# Check renewal timer status
systemctl status certbot.timer

# Test renewal (dry-run)
certbot renew --dry-run
```

### Automatic Renewal (Cron)

On systems without systemd, a cron job runs daily at 2 AM:
```
0 2 * * * certbot renew --quiet --no-self-upgrade
```

### Manual Renewal

```bash
# Renew all certificates
certbot renew

# Renew specific domain
certbot renew --cert-name example.com

# Force renewal (for testing)
certbot renew --force-renewal
```

## Security Features

### Modern TLS Configuration

- ✅ TLS 1.2 and 1.3 only (SSLv3, TLS 1.0, TLS 1.1 disabled)
- ✅ Strong cipher suites (ECDHE-ECDSA, ECDHE-RSA, ChaCha20-Poly1305)
- ✅ HSTS (HTTP Strict Transport Security) enabled
- ✅ Security headers (X-Frame-Options, X-Content-Type-Options)
- ✅ Server signature and tokens minimized

### Security Headers

Automatically applied to HTTPS virtual hosts:
```apache
Header always set Strict-Transport-Security "max-age=31536000"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"
```

### Directory Permissions

```apache
<Directory /var/www/html>
    Options -Indexes +FollowSymLinks  # Directory listing disabled
    AllowOverride All                  # .htaccess support
    Require all granted
</Directory>
```

## Troubleshooting

### Issue: APACHE_RUN_DIR Variable Not Defined

**Error**: 
```
AH00111: Config variable ${APACHE_RUN_DIR} is not defined
apache2: Syntax error on line 80: DefaultRuntimeDir must be a valid directory
```

**Root Cause**: 
The default Apache configuration uses environment variables like `${APACHE_RUN_DIR}` which are only loaded when Apache is started via systemctl/service commands, not when started directly.

**Solution**:
This role automatically manages the main `apache2.conf` file without environment variable dependencies. The configuration uses direct paths instead:

```apache
DefaultRuntimeDir /var/run/apache2
Mutex file:/var/lock/apache2 default
PidFile /var/run/apache2/apache2.pid
```

**If you need to start Apache manually**:
```bash
# Option 1: Use systemctl (recommended)
systemctl start apache2

# Option 2: Use apache2ctl
apache2ctl start

# Option 3: Source envvars first (if needed)
source /etc/apache2/envvars && /usr/sbin/apache2 -DFOREGROUND
```

**What the role does**:
1. Deploys managed `apache2.conf` with direct paths (no environment variables)
2. Deploys `envvars` file for compatibility
3. Creates required directories (`/var/run/apache2`, `/var/lock/apache2`)
4. Validates configuration before starting Apache

**Benefits of managed apache2.conf**:
- ✅ Works with direct Apache invocation
- ✅ No environment variable dependency issues
- ✅ Consistent configuration across systems
- ✅ Ansible manages all settings
- ✅ Automatic backup of original config

### Issue: DNS Check Fails (HTTP-01)

**Error**: `DNS records are missing!`

**Solution**:
1. Verify DNS A-records point to your server:
   ```bash
   dig www.example.com +short
   nslookup www.example.com
   ```
2. Wait for DNS propagation (up to 48 hours)
3. Temporarily disable strict checking:
   ```yaml
   apache_certbot_strict_dns_check: false
   ```

### Issue: Port 80 Not Accessible

**Error**: `Connection refused on port 80`

**Solution**:
1. Check firewall rules:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```
2. Verify Apache is running:
   ```bash
   sudo systemctl status apache2
   ```
3. Consider using DNS-01 challenge instead

### Issue: Certificate Request Fails

**Error**: `Failed authorization procedure`

**Solutions**:
- **HTTP-01**: Ensure `.well-known/acme-challenge/` is accessible
- **DNS-01**: Verify API credentials are correct
- **Rate Limits**: Let's Encrypt has rate limits (50 certs/week per domain)
- **Staging**: Test with staging environment first:
  ```bash
  certbot --staging --apache -d example.com
  ```

### Issue: Wildcard Certificates Fail

**Error**: `Wildcard domains not supported with HTTP-01`

**Solution**: Use DNS-01 challenge method:
```yaml
apache_certbot_challenge_method: "dns"
```

### Issue: DNS-01 Propagation Timeout

**Error**: `DNS propagation timeout`

**Solution**: Increase propagation wait time:
```yaml
apache_certbot_dns_propagation_seconds: 120
```

## Advanced Configuration

### Custom Apache Modules

```yaml
apache_modules_enabled:
  - rewrite
  - ssl
  - headers
  - proxy
  - proxy_http
  - http2
```

### Custom SSL Cipher Suite

```yaml
apache_cipher_suite: "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256"
```

### Multiple Listen Addresses

```yaml
apache_listen_addresses:
  - '10.0.0.5'
  - '192.168.1.10'
```

### Custom Rewrite Rules (Without Let's Encrypt)

```yaml
apache_vhosts:
  - server_name: "example.com"
    with_letsencrypt: false
    rewrite_rules:
      - "RewriteEngine on"
      - "RewriteCond %{HTTP_HOST} ^example\.com$ [NC]"
      - "RewriteRule ^(.*)$ https://www.example.com$1 [R=301,L]"
```

## Tags

Use tags to run specific parts of the role:

```bash
# Install Apache only
ansible-playbook site.yml --tags apache,installation

# Configure virtual hosts only
ansible-playbook site.yml --tags apache,vhosts

# Request certificates only
ansible-playbook site.yml --tags certbot,certificates

# Setup renewal only
ansible-playbook site.yml --tags certbot,renewal
```

Available tags:
- `apache` - All Apache tasks
- `installation` - Package installation
- `configuration` - Configuration file deployment
- `ports` - Ports configuration
- `modules` - Apache module management
- `vhosts` - Virtual host configuration
- `certbot` - All Certbot tasks
- `letsencrypt` - Let's Encrypt certificate tasks
- `certificates` - Certificate request tasks
- `renewal` - Auto-renewal setup
- `http-01` - HTTP-01 specific tasks
- `dns-01` - DNS-01 specific tasks
- `preflight` - Pre-flight checks

## Best Practices

1. **Use DNS-01 for wildcards**: If you need `*.example.com`, use DNS-01 challenge
2. **Test with staging first**: Use `--staging` to avoid rate limits
3. **Keep email updated**: Let's Encrypt sends expiration notices
4. **Monitor renewals**: Check logs regularly (`/var/log/letsencrypt/`)
5. **Use Ansible Vault**: Store sensitive credentials encrypted:
   ```bash
   ansible-vault encrypt_string 'your-api-key' --name 'vault_cloudflare_api_key'
   ```
6. **Backup certificates**: Store `/etc/letsencrypt/` in backups
7. **Document custom configs**: Use comments in your variables files

## Testing

### Test DNS Resolution
```bash
# Check if domains resolve
for domain in www.example.com example.com; do
  echo -n "$domain: "
  dig +short $domain
done
```

### Test SSL Configuration
```bash
# Test with OpenSSL
openssl s_client -connect example.com:443 -servername example.com

# Test with SSL Labs (online)
# https://www.ssllabs.com/ssltest/
```

### Test Certificate Renewal
```bash
# Dry-run renewal (doesn't actually renew)
certbot renew --dry-run
```

### Test Apache Configuration
```bash
# Check syntax
apache2ctl configtest

# View loaded modules
apache2ctl -M
```

## License

MIT

## Author Information

This role was created by Tubby1981 for managing Foomuuri firewalls in production environments.

- **GitHub**: https://github.com/tubby1981/ansible
- **Ansible Galaxy**: https://galaxy.ansible.com/tubby1981/system

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Support

- **Issues**: https://github.com/tubby1981/ansible/issues
- **Documentation**: 
- **Let's Encrypt**: https://github.com/tubby1981/ansible/roles/apache/README.md
- **Certbot**: https://eff-certbot.readthedocs.io/
