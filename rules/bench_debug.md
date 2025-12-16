# Frappe Bench Debugging

This guide covers Frappe bench-specific troubleshooting after system-level checks pass.

**⚠️ Run [network_debug.md](network_debug.md) FIRST for system-level issues**

## Prerequisites Check

Before proceeding, ensure from network_debug.md:

- ✅ Required ports are listening (8000, 9000, 13000, 11000, 3306)
- ✅ Firewall allows connections
- ✅ System services are running (Redis, MariaDB)
- ✅ No critical system errors in logs

## Bench Directory Structure Analysis

### 1. Check Bench Configuration

**Configuration directory:**

```bash
cd /home/frappe/frappe-bench-15

# View bench config
cat config/common_site_config.json

# View Redis config
cat config/redis_cache.conf
cat config/redis_queue.conf
cat config/redis_socketio.conf

# View Supervisor config (if used)
cat config/supervisor.conf
```

**Key configuration items:**

```json
{
  "background_workers": 1,
  "gunicorn_workers": 4,
  "webserver_port": 8000,
  "socketio_port": 9000,
  "redis_cache": "redis://localhost:13000",
  "redis_queue": "redis://localhost:11000",
  "redis_socketio": "redis://localhost:13000"
}
```

### 2. Check Environment

**Python environment:**

```bash
# Activate virtual environment
cd /home/frappe/frappe-bench-15
source env/bin/activate

# Check Python version
python --version

# Check installed packages
pip list | grep frappe

# Verify frappe is installed
python -c "import frappe; print(frappe.__version__)"
```

**Environment issues:**

```bash
# If environment is broken, rebuild
cd /home/frappe/frappe-bench-15
bench setup env
bench pip install -e apps/frappe
bench pip install -e apps/erpnext
```

### 3. Check Bench Logs

**Log directory:**

```bash
cd /home/frappe/frappe-bench-15/logs

# List all log files
ls -lah

# Check recent logs
tail -f *.log
```

**Important log files:**

```bash
# Web server log
tail -f logs/web.log

# Socket.IO log
tail -f logs/socketio.log

# Worker logs
tail -f logs/worker.log

# Schedule log
tail -f logs/schedule.log

# Redis logs
tail -f logs/redis_cache.log
tail -f logs/redis_queue.log
```

**Search for errors:**

```bash
# Search all logs for errors
grep -i "error" logs/*.log

# Search for specific issues
grep -i "traceback" logs/*.log
grep -i "exception" logs/*.log
grep -i "failed" logs/*.log
```

### 4. Check Site Directory

**Site-specific logs and config:**

```bash
cd /home/frappe/frappe-bench-15/sites

# List all sites
ls -la

# Check current site
cat currentsite.txt

# View site config
cat erp.local/site_config.json

# Check site logs
tail -f erp.local/logs/web.log
tail -f erp.local/logs/error.log
tail -f erp.local/logs/scheduler.log
```

**Site database check:**

```bash
# Check if database exists
bench --site erp.local mariadb -e "SHOW DATABASES;"

# Check database connection
bench --site erp.local console
# In console:
frappe.db.get_list("User", limit=1)
exit()
```

### 5. Check Procfile

**Verify Procfile configuration:**

```bash
cat /home/frappe/frappe-bench-15/Procfile

# Check if all processes are defined
grep -E "web|socketio|worker|schedule" Procfile
```

**Expected processes:**

- `web` - Gunicorn web server
- `socketio` - Socket.IO server
- `worker_default` - Background worker (default queue)
- `worker_short` - Background worker (short queue)
- `worker_long` - Background worker (long queue)
- `schedule` - Scheduler for scheduled tasks

## Common Bench Issues

### Issue 1: Bench Not Starting

**Problem:** `bench start` fails

**Diagnostic steps:**

```bash
cd /home/frappe/frappe-bench-15

# Check bench status
bench status

# Try starting with verbose output
bench start --verbose

# Check for port conflicts
sudo lsof -i :8000 -i :9000

# Check Redis connectivity
redis-cli -p 13000 ping
redis-cli -p 11000 ping
```

**Solutions:**

```bash
# Clear cache and restart
bench clear-cache
bench restart

# If Redis is down
sudo systemctl restart redis

# If database is down
sudo systemctl restart mariadb

# Rebuild if needed
bench build
bench restart
```

### Issue 2: 404 Not Found Error

**Problem:** HTTP 404 when accessing site

**Diagnostic steps:**

```bash
# Check if site exists
bench --site erp.local list-apps

# Check site status
bench --site erp.local console
# In console:
frappe.local.site
exit()

# Check web server logs
tail -50 sites/erp.local/logs/web.log

# Check route configuration
bench --site erp.local console
# In console:
frappe.get_all("Web Route")
exit()
```

**Solutions:**

```bash
# Clear cache
bench --site erp.local clear-cache

# Rebuild website
bench --site erp.local clear-website-cache

# Reinstall app if broken
bench --site erp.local reinstall-app frappe

# Or migrate database
bench --site erp.local migrate

# Restart bench
bench restart
```

### Issue 3: Database Connection Errors

**Problem:** Cannot connect to database

**Diagnostic steps:**

```bash
# Check MariaDB status
sudo systemctl status mariadb

# Test database connection
bench --site erp.local mariadb

# Check database credentials
cat sites/erp.local/site_config.json | grep db_

# Test connection manually
mysql -u [db_user] -p[db_password] [db_name]
```

**Solutions:**

```bash
# Restart MariaDB
sudo systemctl restart mariadb

# Reset database password if needed
bench --site erp.local set-admin-password [new_password]

# Repair database
bench --site erp.local console
# In console:
frappe.db.sql("REPAIR TABLE `tabUser`")
exit()
```

### Issue 4: Redis Connection Errors

**Problem:** Redis connection failed

**Diagnostic steps:**

```bash
# Check Redis status
sudo systemctl status redis

# Test Redis connections
redis-cli -p 13000 ping  # Cache
redis-cli -p 11000 ping  # Queue

# Check Redis logs
tail -50 logs/redis_cache.log
tail -50 logs/redis_queue.log

# Check config
cat config/redis_cache.conf
cat config/redis_queue.conf
```

**Solutions:**

```bash
# Restart Redis
sudo systemctl restart redis

# Or restart specific Redis instances
redis-cli -p 13000 shutdown
redis-cli -p 11000 shutdown

# Restart bench (will restart Redis)
bench restart

# If config is wrong, reset
bench setup redis
```

### Issue 5: Permission Errors

**Problem:** Permission denied errors

**Diagnostic steps:**

```bash
# Check file permissions
ls -la /home/frappe/frappe-bench-15

# Check process user
ps aux | grep bench | grep -v grep

# Check site ownership
ls -la sites/erp.local
```

**Solutions:**

```bash
# Fix ownership (replace frappe with correct user)
sudo chown -R frappe:frappe /home/frappe/frappe-bench-15

# Fix permissions
chmod -R 755 /home/frappe/frappe-bench-15
chmod -R 755 sites

# Fix logs directory
chmod -R 755 logs
```

### Issue 6: Migration Error - Module **file** is None

**Problem:** `TypeError: expected str, bytes or os.PathLike object, not NoneType` during `bench migrate`

**Error location:** `frappe/model/sync.py` line 105

**Root cause:** A module listed in `modules.txt` exists as a namespace package (missing `__init__.py`) or doesn't exist properly, causing `frappe.get_module()` to return a module with `__file__` as `None`.

**Diagnostic steps:**

```bash
# Check which app is causing the error from the migration output
# Look for the app name before the error occurs

# Check modules.txt files in custom apps
find apps -name "modules.txt" -not -path "*/frappe/*" -not -path "*/erpnext/*" -not -path "*/hrms/*" -exec echo "=== {} ===" \; -exec cat {} \;

# For each custom app, verify module structure
cd /home/frappe/frappe-bench/apps
for app in */; do
  if [ -f "${app}${app%/}/modules.txt" ]; then
    echo "=== Checking ${app} ==="
    while read module; do
      module_path="${app}${app%/}/${module}"
      if [ -d "$module_path" ] && [ ! -f "$module_path/__init__.py" ]; then
        echo "MISSING __init__.py: $module_path"
      fi
    done < "${app}${app%/}/modules.txt"
  fi
done
```

**Solutions:**

```bash
# For each problematic module, create __init__.py file
cd /home/frappe/frappe-bench/apps/[app_name]/[app_name]
mkdir -p [module_name]
touch [module_name]/__init__.py

# Or remove the module from modules.txt if it's not needed
# Edit: apps/[app_name]/[app_name]/modules.txt
# Remove the problematic module line

# Clear cache and retry migration
bench --site [site_name] clear-cache
bench --site [site_name] migrate
```

**Prevention:**

- Always ensure modules listed in `modules.txt` have proper Python package structure with `__init__.py`
- When creating new modules, use `ModuleDef` DocType which automatically creates the structure
- Never manually add module names to `modules.txt` without creating the corresponding directory and `__init__.py`

## Bench Maintenance Commands

### Clear Cache and Rebuild

```bash
# Clear all caches
bench --site erp.local clear-cache

# Clear website cache
bench --site erp.local clear-website-cache

# Rebuild assets
bench build

# Rebuild all
bench build --force
```

### Database Maintenance

```bash
# Run migrations
bench --site erp.local migrate

# Backup database
bench --site erp.local backup

# Restore database
bench --site erp.local restore [backup-file]

# Optimize database
bench --site erp.local console
# In console:
frappe.db.sql("OPTIMIZE TABLE `tabUser`")
exit()
```

### Update and Upgrade

```bash
# Update bench
bench update --no-backup

# Upgrade specific app
bench update --app erpnext

# Reset app (dangerous!)
bench --site erp.local reinstall-app erpnext
```

## Advanced Diagnostics

### Check Bench Doctor

```bash
# Run bench doctor
bench doctor

# Check configuration
bench --site erp.local doctor
```

### Monitor Bench Processes

```bash
# Watch bench processes
watch -n 2 'ps aux | grep bench | grep -v grep'

# Monitor resources
top -p $(pgrep -d',' -f bench)

# Check queue status
bench --site erp.local console
# In console:
frappe.get_all("RQ Job", filters={"status": "failed"})
exit()
```

### Analyze Error Logs

```bash
# Count errors by type
grep -h "ERROR" logs/*.log | cut -d' ' -f6- | sort | uniq -c | sort -rn | head -20

# Find recent tracebacks
grep -B 5 -A 10 "Traceback" logs/*.log | tail -50

# Monitor live errors
tail -f logs/*.log | grep --color=auto -E "ERROR|CRITICAL|Exception"
```

## Configuration Files Reference

| File                    | Purpose              | Location                                        |
| ----------------------- | -------------------- | ----------------------------------------------- |
| common_site_config.json | Global bench config  | `/home/frappe/frappe-bench-15/sites/`           |
| site_config.json        | Site-specific config | `/home/frappe/frappe-bench-15/sites/erp.local/` |
| Procfile                | Process definitions  | `/home/frappe/frappe-bench-15/`                 |
| redis_cache.conf        | Redis cache config   | `/home/frappe/frappe-bench-15/config/`          |
| redis_queue.conf        | Redis queue config   | `/home/frappe/frappe-bench-15/config/`          |
| supervisor.conf         | Supervisor config    | `/home/frappe/frappe-bench-15/config/`          |

## Quick Troubleshooting Checklist

- [ ] Bench environment activated: `source env/bin/activate`
- [ ] Database connection works: `bench --site erp.local mariadb`
- [ ] Redis connections work: `redis-cli -p 13000 ping`
- [ ] Site exists and is active: `bench --site erp.local list-apps`
- [ ] No errors in logs: `grep ERROR logs/*.log`
- [ ] Migrations are current: `bench --site erp.local migrate`
- [ ] Cache is clear: `bench --site erp.local clear-cache`
- [ ] Assets are built: `bench build`
- [ ] Permissions are correct: `ls -la`
- [ ] Bench processes running: `ps aux | grep bench`
- [ ] All modules in modules.txt have **init**.py files (if migration fails with TypeError)

## When to Reinstall

If all else fails, consider reinstalling:

```bash
# Backup first!
bench --site erp.local backup --with-files

# Reinstall app
bench --site erp.local reinstall-app frappe

# Or create new site
bench new-site new-site-name
bench --site new-site-name install-app erpnext
bench --site new-site-name restore [backup-file]
```

---

**Rule Reference:** This file follows the Frappe bench debugging protocol.
