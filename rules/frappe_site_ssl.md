# SSL Certificate Setup for Frappe Sites

This guide covers setting up SSL certificates using Let's Encrypt (Certbot) for Frappe sites.

## Prerequisites

**For Proxmox Containers:**
```bash
# Install snapd and required packages for Proxmox container
apt install snapd squashfuse fuse lxcfs -y
```

**For Regular Ubuntu/Debian Systems:**
```bash
# Install snapd
sudo apt install snapd -y
sudo snap install core
sudo snap refresh core

# Remove old certbot (if exists)
sudo apt -y remove certbot

# Install certbot via snap
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Verify installation
sudo certbot --version
```

## Setup Nginx

**Configure Nginx for Frappe:**
```bash
# Setup nginx configuration for Frappe
sudo bench setup nginx

# Test nginx configuration
sudo nginx -t

# Reload/restart nginx
sudo service nginx reload
sudo service nginx restart
# OR
sudo systemctl restart nginx

# Check nginx status
sudo systemctl status nginx
sudo systemctl start nginx  # If not running
```

## Obtain SSL Certificate

**For All Sites (Automatic):**
```bash
# Certbot will automatically detect all sites configured in nginx
sudo certbot --nginx
```

**For Specific Site:**
```bash
# Get certificate for specific domain(s)
sudo certbot --nginx -d example.com -d www.example.com

# Example for tbook.app:
sudo certbot --nginx -d tbook.app -d www.tbook.app
```

**Verify Certificates:**
```bash
# List all certificates
sudo certbot certificates
```

## Restart Services

**After SSL Setup:**
```bash
# Restart bench services
bench restart

# Restart nginx
sudo systemctl restart nginx

# Restart all supervisor services
sudo supervisorctl restart all
```

## Certificate Management

**Revoke Certificate:**
```bash
# Revoke a certificate (certificate will be invalidated)
sudo certbot revoke --cert-name medico.local
```

**Delete Certificate:**
```bash
# Delete certificate configuration
sudo certbot delete --cert-name frenchhome.tbook.app
sudo certbot delete --cert-name medico.local
```

## Auto-Renewal

**Certbot automatically sets up renewal. Test renewal:**
```bash
# Test renewal process
sudo certbot renew --dry-run
```

**Check renewal status:**
```bash
# View renewal timer
sudo systemctl status certbot.timer
```

## Troubleshooting

**Common Issues:**

1. **Nginx configuration errors:**
   ```bash
   sudo nginx -t  # Check configuration
   ```

2. **Port 80/443 not accessible:**
   ```bash
   # Check firewall
   sudo ufw status
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```

3. **DNS not resolving:**
   ```bash
   # Verify DNS A record points to server IP
   dig +short example.com A
   nslookup example.com
   ```

4. **Certificate renewal fails:**
   ```bash
   # Check certbot logs
   sudo tail -f /var/log/letsencrypt/letsencrypt.log
   ```

## Notes

- **DNS Propagation:** Ensure DNS A record points to server IP before requesting certificate
- **Port 80 Required:** Let's Encrypt requires port 80 to be open for domain validation
- **Auto-Renewal:** Certbot sets up automatic renewal via systemd timer
- **Certificate Location:** Certificates are stored in `/etc/letsencrypt/live/[domain]/`
- **Nginx Config:** Certbot automatically updates nginx configuration files

## Current Site Configuration

**tbook.app:**
- DNS A Record: `tbook.app` → `95.111.234.1`
- Site Directory: `/home/frappe/frappe-bench-15/sites/tbook.app`
- Symbolic Link: `95.111.234.1` → `tbook.app` (for direct IP access)

