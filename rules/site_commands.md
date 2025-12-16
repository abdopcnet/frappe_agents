# Site Commands Reference

## Site Management

**Create/Remove:**

```bash
bench new-site [site_name]
bench drop-site [site_name]
bench --site [site] list-apps
```

## App Management

**Install/Uninstall:**

```bash
bench get-app [app_name]
bench --site [site] install-app [app_name]
bench --site [site] uninstall-app [app_name]
```

**Real Apps in This Bench:**

- frappe, erpnext, hrms, lms, payments, hrms_custom, export_custom_fields

## Database Operations

**Backup:**

```bash
bench --site [site] backup
bench --site [site] backup --with-files
```

**Migrate:**

```bash
bench --site [site] migrate
bench --site [site] migrate --app [app_name]
```

**Console:**

```bash
bench --site [site] console
>>> import frappe
>>> frappe.get_installed_apps()
```

**MariaDB:**

```bash
bench --site [site] mariadb
bench --site [site] mariadb -e "SELECT * FROM \`tabUser\` LIMIT 10;"
```

## Development Commands

**Build:**

```bash
bench build
bench build --app [app_name]
bench build --production
```

**Restart:**

```bash
bench restart
```

**Cache:**

```bash
bench --site [site] clear-cache
bench --site [site] clear-website-cache
```

**Watch:**

```bash
bench watch  # Auto-rebuild on changes
```

**Reload:**

```bash
bench --site [site] reload-doctype "DocType Name"
```

## Configuration

**Set Config:**

```bash
bench --site [site] set-config developer_mode 1
bench --site [site] set-config maintenance_mode 1
bench --site [site] set-config socketio_port 9000
```

**DNS Multitenant Mode:**

```bash
# Single site mode - allows IP access
bench config dns_multitenant off
# Sets serve_default_site=true, uses default_site for all requests

# Multi-site mode - requires hostname matching
bench config dns_multitenant on
# Each request hostname must match a site directory name
```

**How Frappe Determines Site:**

1. X-Frappe-Site-Name header (highest priority)
2. HTTP Host header → matches site directory
3. If dns_multitenant=off: uses default_site

**Accessing Site via IP Address:**
When accessing a site via IP address (e.g., `http://95.111.234.1/`), Frappe looks for a site directory matching the IP. If your site has a different name, create a symbolic link:

```bash
cd /home/frappe/frappe-bench-15/sites
ln -s [actual_site_name] [ip_address]
chown frappe:frappe [ip_address]
```

Example:

```bash
ln -s tbook.app 95.111.234.1
chown frappe:frappe 95.111.234.1
```

**Current Config (common_site_config.json):**

- developer_mode: 1
- socketio_port: 9000
- webserver_port: 8000
- default_site: tbook.app
- dns_multitenant: false
- serve_default_site: true

## Production

**Setup:**

```bash
sudo bench setup production [user]
sudo bench setup nginx
sudo bench setup lets-encrypt [site]
```

**Update:**

```bash
bench update
bench update --app [app_name]
bench update --pull
```

## Batch Operations

**All Sites:**

```bash
bench --site all backup
bench --site all migrate
bench --site all clear-cache
```

## Troubleshooting

**Logs:**

```bash
tail -f sites/[site]/logs/error.log
tail -f sites/[site]/logs/access.log
```

**Permissions:**

```bash
sudo chown -R frappe:frappe /home/frappe/frappe-bench
```

**Ports:**

```bash
sudo lsof -i :8000  # Web server
sudo lsof -i :9000  # Socket.IO
```
