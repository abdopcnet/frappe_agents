# Network & Server Debugging

This guide covers system-level network and server troubleshooting for Frappe installations.

**⚠️ Run this FIRST before checking Frappe-specific issues (see bench_debug.md)**

## Quick Diagnostic Commands

### 1. Check Running Ports

**List all listening ports:**

```bash
# All listening ports
sudo netstat -tulpn | grep LISTEN

# OR using ss (modern alternative)
sudo ss -tulpn | grep LISTEN

# OR using lsof
sudo lsof -i -P -n | grep LISTEN
```

**Check specific Frappe ports:**

```bash
# Web server (8000)
sudo netstat -tulpn | grep :8000

# Socket.IO (9000)
sudo netstat -tulpn | grep :9000

# Redis Cache (13000)
sudo netstat -tulpn | grep :13000

# Redis Queue (11000)
sudo netstat -tulpn | grep :11000

# MariaDB/MySQL (3306)
sudo netstat -tulpn | grep :3306
```

### 2. Check Service Status

**Check if services are running:**

```bash
# Check bench processes
ps aux | grep bench

# Check Redis
sudo systemctl status redis
ps aux | grep redis

# Check MariaDB/MySQL
sudo systemctl status mariadb
# OR
sudo systemctl status mysql

# Check Nginx (if used)
sudo systemctl status nginx
```

### 3. Check Network Connectivity

**Test local connectivity:**

```bash
# Test localhost
curl http://localhost:8000

# Test specific IP
curl http://95.111.234.1/

# Check if port is accessible
telnet localhost 8000
# OR
nc -zv localhost 8000
```

**Check network interface:**

```bash
# List network interfaces
ip addr show

# Check if IP is bound
ip addr | grep "95.111.234.1"

# Check routing
ip route show
```

### 4. Check Firewall Rules

**Ubuntu/Debian (ufw):**

```bash
# Check firewall status
sudo ufw status verbose

# Check if port 8000 is allowed
sudo ufw status | grep 8000

# Allow port if blocked
sudo ufw allow 8000/tcp
```

**CentOS/RHEL (firewalld):**

```bash
# Check firewall status
sudo firewall-cmd --state

# List open ports
sudo firewall-cmd --list-ports

# Open port if needed
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload
```

**iptables:**

```bash
# List firewall rules
sudo iptables -L -n -v

# Check if port is blocked
sudo iptables -L INPUT -n -v | grep 8000
```

## System Logs Analysis

### 1. Check System Logs

**General system logs:**

```bash
# Recent system errors
sudo tail -f /var/log/syslog
# OR on CentOS/RHEL
sudo tail -f /var/log/messages

# Authentication logs
sudo tail -f /var/log/auth.log

# Kernel logs
dmesg | tail -50
```

### 2. Check Web Server Logs

**Nginx logs:**

```bash
# Access log
sudo tail -f /var/log/nginx/access.log

# Error log
sudo tail -f /var/log/nginx/error.log
```

**Apache logs:**

```bash
# Access log
sudo tail -f /var/log/apache2/access.log

# Error log
sudo tail -f /var/log/apache2/error.log
```

### 3. Check Service Logs

**MariaDB/MySQL logs:**

```bash
# Error log
sudo tail -f /var/log/mysql/error.log
# OR
sudo tail -f /var/log/mariadb/mariadb.log

# Check if MySQL is running
sudo systemctl status mariadb
```

**Redis logs:**

```bash
# Redis log
sudo tail -f /var/log/redis/redis-server.log

# Check Redis connectivity
redis-cli ping
```

## Common Network Issues

### Issue 1: Port Already in Use

**Problem:** `Address already in use` error

**Solution:**

```bash
# Find process using port 8000
sudo lsof -i :8000

# Kill the process (replace PID)
sudo kill -9 <PID>

# Or kill all Python processes (careful!)
pkill -9 python
```

### Issue 2: Cannot Bind to IP Address

**Problem:** `Cannot assign requested address`

**Solution:**

```bash
# Check if IP exists on system
ip addr show | grep "95.111.234.1"

# If IP doesn't exist, check bench config
cat /home/frappe/frappe-bench-15/sites/common_site_config.json

# Edit config if needed
nano /home/frappe/frappe-bench-15/sites/common_site_config.json
```

### Issue 3: Firewall Blocking Connections

**Problem:** Connection refused or timeout

**Solution:**

```bash
# Temporarily disable firewall (for testing only!)
sudo ufw disable

# Test connection
curl http://localhost:8000

# If it works, add firewall rule
sudo ufw allow 8000/tcp
sudo ufw enable
```

### Issue 4: Service Not Running

**Problem:** 404 errors, connection refused

**Solution:**

```bash
# Check if bench is running
ps aux | grep bench

# If not running, start bench
cd /home/frappe/frappe-bench-15
bench start

# Or use supervisor (if configured)
sudo supervisorctl status all
sudo supervisorctl start all
```

### Issue 5: 404 Error When Accessing Site via IP Address

**Problem:** Getting 404 NOT FOUND when accessing site via IP address (e.g., `http://95.111.234.1/`)

**Root Cause:** Frappe matches the HTTP Host header to a site directory name. When accessing via IP address, Frappe looks for a site directory named after the IP, but the actual site has a different name (e.g., `tbook.app`).

**Solution:** Create a symbolic link from the IP address to the actual site directory:

```bash
# Navigate to sites directory
cd /home/frappe/frappe-bench-15/sites

# Create symbolic link: IP -> actual site directory
ln -s tbook.app 95.111.234.1

# Set correct ownership (match site directory owner)
chown frappe:frappe 95.111.234.1

# Verify the link
ls -la | grep "95.111.234.1"
# Should show: lrwxrwxrwx ... 95.111.234.1 -> tbook.app
```

**How it works:**

1. When accessing `http://95.111.234.1/`, Frappe extracts hostname `95.111.234.1`
2. Frappe looks for site directory `/home/frappe/frappe-bench-15/sites/95.111.234.1`
3. Finds symbolic link pointing to `tbook.app`
4. Loads site configuration from `tbook.app/site_config.json`
5. Site is accessible via IP address

**Note:** This solution does NOT require modifying core Frappe code. It's a filesystem-level solution that works with Frappe's existing site resolution mechanism.

## Network Configuration Check

### Check Bench Site Configuration

**View configuration:**

```bash
# Common site config
cat /home/frappe/frappe-bench-15/sites/common_site_config.json

# Specific site config
cat /home/frappe/frappe-bench-15/sites/erp.local/site_config.json
```

**Common configuration issues:**

```json
{
  "webserver_port": 8000,
  "socketio_port": 9000,
  "serve_default_site": true,
  "default_site": "erp.local"
}
```

### Check Procfile

**View Procfile:**

```bash
cat /home/frappe/frappe-bench-15/Procfile
```

**Expected content:**

```
web: bench serve --port 8000
socketio: /usr/bin/node apps/frappe/socketio.js
watch: bench watch
schedule: bench schedule
worker_short: bench worker --queue short
worker_long: bench worker --queue long
worker_default: bench worker --queue default
```

## DNS and Host Resolution

### Check DNS Resolution

**Test DNS:**

```bash
# Check hostname resolution
nslookup erp.local

# Check /etc/hosts file
cat /etc/hosts | grep erp.local

# Add to hosts if missing
echo "127.0.0.1 erp.local" | sudo tee -a /etc/hosts
```

### Check Site Configuration

**Verify site exists:**

```bash
# List all sites
ls -la /home/frappe/frappe-bench-15/sites/

# Check if site is active
cat /home/frappe/frappe-bench-15/sites/currentsite.txt
```

## Performance Checks

### Check System Resources

**CPU and Memory:**

```bash
# System resources
top
# OR
htop

# Memory usage
free -h

# Disk space
df -h
```

**Process-specific:**

```bash
# Check Python processes
ps aux | grep python | grep -v grep

# Check resource usage by process
top -p $(pgrep -d',' -f bench)
```

## Next Steps

If network/system checks are OK but still having issues:

1. ✅ All required ports are listening
2. ✅ Firewall allows connections
3. ✅ Services are running
4. ✅ No system errors in logs

**→ Proceed to [bench_debug.md](bench_debug.md) for Frappe-specific troubleshooting**

## Quick Reference

| Issue              | Check             | Command                                 |
| ------------------ | ----------------- | --------------------------------------- |
| Port not listening | Running processes | `sudo netstat -tulpn \| grep 8000`      |
| Service down       | Service status    | `sudo systemctl status mariadb`         |
| Firewall blocking  | Firewall rules    | `sudo ufw status`                       |
| Connection refused | Service running   | `ps aux \| grep bench`                  |
| 404 Error          | Web server logs   | `sudo tail -f /var/log/nginx/error.log` |

---

**Rule Reference:** This file follows the network debugging protocol for Frappe installations.
