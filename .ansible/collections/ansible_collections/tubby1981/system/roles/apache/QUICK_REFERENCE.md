# Apache Role - Quick Reference

## 🚀 Quick Start

### Minimal Configuration (HTTP only)

```yaml
# group_vars/webservers.yml
apache_vhosts:
  - server_name: "www.example.com"
    document_root: "/var/www/example"
```

### With HTTPS (Let's Encrypt - HTTP-01)

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"

apache_vhosts:
  - server_name: "www.example.com"
    document_root: "/var/www/example"
    ssl: true
    with_letsencrypt: true
```

### With HTTPS (Let's Encrypt - DNS-01)

```yaml
apache_certbot_enabled: true
apache_certbot_email: "admin@example.com"
apache_certbot_challenge_method: "dns"
apache_certbot_dns_provider: "cloudflare"
apache_certbot_dns_credentials_content: |
  dns_cloudflare_email = admin@example.com
  dns_cloudflare_api_key = your-api-key

apache_vhosts:
  - server_name: "*.example.com"  # Wildcard support
    document_root: "/var/www/example"
    ssl: true
    with_letsencrypt: true
```

## 📋 Common Tasks

### Run Playbook

```bash
# Full deployment
ansible-playbook site.yml

# Specific host
ansible-playbook site.yml --limit webserver01

# Dry run (check mode)
ansible-playbook site.yml --check

# With diff output
ansible-playbook site.yml --check --diff
```

### Use Tags

```bash
# Install Apache only
ansible-playbook site.yml --tags installation

# Configure virtual hosts only
ansible-playbook site.yml --tags vhosts

# Request SSL certificates only
ansible-playbook site.yml --tags certificates

# Setup renewal only
ansible-playbook site.yml --tags renewal
```

### Manual Apache Commands

```bash
# Start/stop/restart
systemctl start apache2
systemctl stop apache2
systemctl restart apache2
systemctl reload apache2

# Check status
systemctl status apache2

# Test configuration
apache2ctl configtest

# View virtual hosts
apache2ctl -S

# View loaded modules
apache2ctl -M

# Check listening ports
ss -tlnp | grep apache2
```

## 🔧 Configuration Examples

### Multiple Domains

```yaml
apache_vhosts:
  - server_name: "www.site1.com"
    document_root: "/var/www/site1"
    ssl: true
    with_letsencrypt: true
    
  - server_name: "www.site2.com"
    document_root: "/var/www/site2"
    ssl: true
    with_letsencrypt: true
```

### With Aliases

```yaml
apache_vhosts:
  - server_name: "www.example.com"
    document_root: "/var/www/example"
    ssl: true
    with_letsencrypt: true
    aliases:
      - "example.com"
      - "example.org"
      - "www.example.org"
```

### Custom Ports

```yaml
apache_listen_ports: [80, 8080]
apache_ssl_ports: [443, 8443]

apache_vhosts:
  - server_name: "app.example.com"
    document_root: "/var/www/app"
    port: 8080
    ssl: true
    ssl_port: 8443
```

### Custom Rewrite Rules (No Let's Encrypt)

```yaml
apache_vhosts:
  - server_name: "www.example.com"
    document_root: "/var/www/example"
    with_letsencrypt: false
    rewrite_rules:
      - "RewriteEngine On"
      - "RewriteRule ^old-url$ /new-url [R=301,L]"
```

### HTTP Only (No SSL)

```yaml
apache_vhosts:
  - server_name: "dev.example.com"
    document_root: "/var/www/dev"
    ssl: false
```

### Performance Tuning

```yaml
apache_timeout: 300
apache_keepalive: "On"
apache_max_keepalive_requests: 500
apache_keepalive_timeout: 10
```

### Custom Modules

```yaml
apache_modules_enabled:
  - rewrite
  - ssl
  - headers
  - proxy
  - proxy_http
  - proxy_wstunnel
  - http2
```

## 🔍 Troubleshooting Commands

### Check Configuration

```bash
# Test syntax
apache2ctl configtest

# View configuration with includes
apache2ctl -t -D DUMP_VHOSTS -D DUMP_MODULES

# Check specific file syntax
apache2 -t -f /etc/apache2/apache2.conf
```

### Check Logs

```bash
# Watch error log
tail -f /var/log/apache2/error.log

# Watch access log
tail -f /var/log/apache2/access.log

# Search for errors
grep -i error /var/log/apache2/error.log | tail -20

# View systemd journal
journalctl -u apache2 -f
```

### Check SSL

```bash
# Check certificate
openssl s_client -connect example.com:443 -servername example.com

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# List Let's Encrypt certificates
certbot certificates

# Test renewal
certbot renew --dry-run
```

### DNS Checks

```bash
# Check A record
dig example.com A +short

# Check from specific DNS server
dig @8.8.8.8 example.com A +short

# Check all records
dig example.com ANY

# Reverse DNS lookup
dig -x <IP_ADDRESS>
```

### Network Checks

```bash
# Check if port is open
nc -zv example.com 80
nc -zv example.com 443

# Check from internet
curl -I http://example.com
curl -I https://example.com

# Check with verbose output
curl -v https://example.com
```

## 🔐 Security Checks

### SSL Labs Test

```bash
# Online test
# Visit: https://www.ssllabs.com/ssltest/analyze.html?d=example.com
```

### Mozilla Observatory

```bash
# Online test
# Visit: https://observatory.mozilla.org/analyze/example.com
```

### Check Headers

```bash
# View all headers
curl -I https://example.com

# Check specific headers
curl -I https://example.com | grep -i "strict-transport"
curl -I https://example.com | grep -i "x-frame"
```

## 📁 Important File Locations

### Configuration Files

```
/etc/apache2/apache2.conf           # Main configuration
/etc/apache2/envvars                # Environment variables
/etc/apache2/ports.conf             # Port configuration
/etc/apache2/conf-available/        # Available configurations
/etc/apache2/conf-enabled/          # Enabled configurations
/etc/apache2/sites-available/       # Available virtual hosts
/etc/apache2/sites-enabled/         # Enabled virtual hosts
/etc/apache2/mods-available/        # Available modules
/etc/apache2/mods-enabled/          # Enabled modules
```

### Runtime Files

```
/var/run/apache2/                   # Runtime directory
/var/run/apache2/apache2.pid        # PID file
/var/lock/apache2/                  # Lock files
```

### Logs

```
/var/log/apache2/error.log          # Error log
/var/log/apache2/access.log         # Access log
/var/log/apache2/*_error.log        # Per-vhost error logs
/var/log/apache2/*_access.log       # Per-vhost access logs
```

### Let's Encrypt

```
/etc/letsencrypt/live/              # Current certificates (symlinks)
/etc/letsencrypt/archive/           # All certificates
/etc/letsencrypt/renewal/           # Renewal configuration
/var/log/letsencrypt/               # Certbot logs
```

### Website Content

```
/var/www/html/                      # Default document root
/var/www/*/                         # Custom document roots
```

## 🛠️ Common Issues & Solutions

### Issue: Apache Won't Start

```bash
# Check status
systemctl status apache2

# Check logs
journalctl -u apache2 -n 50

# Test configuration
apache2ctl configtest

# Common fixes:
# - Fix syntax errors in config
# - Check if port is already in use: lsof -i :80
# - Verify SSL certificates exist
```

### Issue: Site Not Loading

```bash
# Check if site is enabled
ls -la /etc/apache2/sites-enabled/

# Enable site
a2ensite example.com.conf
systemctl reload apache2

# Check virtual hosts
apache2ctl -S
```

### Issue: SSL Certificate Error

```bash
# Check certificate files
ls -la /etc/letsencrypt/live/example.com/

# Renew manually
certbot renew --force-renewal

# Check renewal config
cat /etc/letsencrypt/renewal/example.com.conf
```

### Issue: Permission Denied

```bash
# Check document root permissions
ls -la /var/www/

# Fix ownership
chown -R www-data:www-data /var/www/example

# Fix permissions
chmod -R 755 /var/www/example
```

## 📊 Variable Reference

### Essential Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_enabled` | `true` | Enable role |
| `apache_certbot_enabled` | `false` | Enable Let's Encrypt |
| `apache_certbot_email` | `""` | Email for certificates |
| `apache_certbot_challenge_method` | `"http"` | `"http"` or `"dns"` |

### HTTP-01 Challenge

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_certbot_strict_dns_check` | `true` | Fail if DNS missing |

### DNS-01 Challenge

| Variable | Default | Description |
|----------|---------|-------------|
| `apache_certbot_dns_provider` | `""` | e.g. `"cloudflare"` |
| `apache_certbot_dns_credentials_file` | `""` | Path to credentials |
| `apache_certbot_dns_credentials_content` | `""` | Credentials content |
| `apache_certbot_dns_propagation_seconds` | `60` | DNS wait time |

## 🎯 Best Practices

1. **Always backup before changes**: Create backups of `/etc/apache2/`
2. **Test in development first**: Use staging environment
3. **Use check mode**: Run with `--check` before applying
4. **Monitor logs**: Watch logs during deployment
5. **Version control**: Keep Ansible code in Git
6. **Document custom configs**: Add comments to your variables
7. **Use Ansible Vault**: Encrypt sensitive credentials
8. **Test rollback**: Verify rollback procedures work
9. **Keep certificates updated**: Monitor Let's Encrypt expiry
10. **Review security**: Regular security audits

## 📚 Documentation

- **Full Documentation**: `README.md`
- **File Structure**: `DIRECTORY_STRUCTURE.md`
- **This Guide**: `QUICK_REFERENCE.md`

## 🆘 Support

- **Apache Docs**: https://httpd.apache.org/docs/2.4/
- **Let's Encrypt**: https://letsencrypt.org/docs/
- **Certbot**: https://eff-certbot.readthedocs.io/
- **Ansible**: https://docs.ansible.com/

## 🏷️ Available Tags

| Tag | Purpose |
|-----|---------|
| `apache` | All Apache tasks |
| `installation` | Package installation |
| `configuration` | Config files |
| `ports` | Port configuration |
| `security` | Security settings |
| `modules` | Module management |
| `vhosts` | Virtual hosts |
| `validation` | Config testing |
| `certbot` | All Certbot tasks |
| `certificates` | Certificate requests |
| `renewal` | Auto-renewal |
| `http-01` | HTTP-01 specific |
| `dns-01` | DNS-01 specific |
| `preflight` | Pre-flight checks |
