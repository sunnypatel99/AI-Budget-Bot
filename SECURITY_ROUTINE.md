# Security Check Routine for Self-Hosted n8n

A comprehensive security maintenance schedule for self-hosted n8n instances on Raspberry Pi or other Linux systems. This routine helps ensure your automation remains secure, up-to-date, and running smoothly.

## 📋 Overview

Running a self-hosted n8n instance requires regular security maintenance to protect your workflows, credentials, and personal data. This guide provides a systematic approach to monitoring and maintaining your deployment.

**Time Investment:**
- Daily: 2 minutes
- Weekly: 5-10 minutes  
- Monthly: 15-20 minutes
- Quarterly: 30 minutes

**Total monthly commitment: ~1 hour** (protects hours of work and sensitive data)

---

## 🔄 Daily Checks (2 minutes)

### Quick Health Verification

Run these commands to ensure your system is operating normally:

```bash
# 1. Verify n8n container is running
docker ps | grep n8n
# Expected: STATUS shows "Up X hours/days"

# 2. Quick log check for errors
docker logs --tail 20 n8n
# Look for: No error messages or authentication failures

# 3. Check disk space
df -h | grep -E 'Filesystem|/$'
# Expected: Usage below 80%
```

### What to Look For

**✅ Good Signs:**
- Container status: "Up"
- No error messages in logs
- Disk usage below 80%

**⚠️ Warning Signs:**
- Container restarting repeatedly
- Authentication errors in logs
- Disk usage above 80%

---

## 📅 Weekly Checks (5-10 minutes)

### 1. System Updates

Keep your OS and packages up to date:

```bash
# Update package list
sudo apt update

# Check for available updates
sudo apt list --upgradable

# Install updates (requires confirmation)
sudo apt upgrade -y

# Check if reboot is needed
[ -f /var/run/reboot-required ] && echo "Reboot required" || echo "No reboot needed"
```

### 2. Review n8n Execution History

**Via n8n Web Interface:**

1. Open n8n: `http://localhost:5678` (via Raspberry Pi Connect or local network)
2. Navigate to **Executions** tab
3. Review last 7 days of executions

**Look for:**
- ⚠️ Failed workflows
- ⚠️ Repeated errors
- ⚠️ Unusual execution patterns
- ⚠️ API authentication failures

### 3. Verify Firewall Status

```bash
# Check firewall is active
sudo ufw status

# Expected output:
# Status: active
# Default: deny (incoming), allow (outgoing)
```

**What to verify:**
- ✅ Status: active
- ✅ Default deny incoming
- ✅ No unexpected allow rules

### 4. Check for Unauthorized Access Attempts

```bash
# Review failed login attempts (last 20)
sudo grep "Failed password" /var/log/auth.log | tail -n 20

# Check for multiple attempts from same IP
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10
```

**Red flags:**
- ❌ Multiple failed attempts from external IPs
- ❌ Attempts on unusual accounts
- ❌ Repeated attempts in short timeframe

### 5. Verify n8n Network Binding

```bash
# Check what ports are listening
sudo netstat -tlnp | grep 5678

# Expected: 0.0.0.0:5678 or :::5678 (local network only)
# NOT: External IP addresses
```

### 6. Docker Health Check

```bash
# View Docker container stats
docker stats n8n --no-stream

# Check Docker disk usage
docker system df

# View n8n container details
docker inspect n8n | grep -A 5 "State"
```

---

## 🗓️ Monthly Checks (15-20 minutes)

### 1. Backup Workflows

**Create timestamped backup:**

```bash
# Create backup directory if it doesn't exist
mkdir -p ~/n8n-backups

# Create compressed backup with date
tar -czf ~/n8n-backups/n8n-backup-$(date +%Y%m%d).tar.gz ~/.n8n

# Verify backup was created
ls -lh ~/n8n-backups/

# Check backup size
du -sh ~/n8n-backups/n8n-backup-$(date +%Y%m%d).tar.gz
```

**Download backup to safe location:**

Via Raspberry Pi Connect:
1. Connect to Pi via browser
2. Open File Manager
3. Navigate to `~/n8n-backups/`
4. Download latest backup to your computer
5. Store in cloud storage (Google Drive, Dropbox) or external drive

**Backup retention:**
```bash
# Keep only last 4 weekly backups (saves space)
cd ~/n8n-backups
ls -t | tail -n +5 | xargs rm -f

# Or keep all monthly backups for 1 year
# Delete backups older than 365 days
find ~/n8n-backups -name "*.tar.gz" -mtime +365 -delete
```

### 2. Update n8n Docker Image

**Pull latest version:**

```bash
# Pull the latest n8n image
docker pull docker.n8n.io/n8nio/n8n

# Check if update available
# Compare IMAGE ID of running container vs pulled image
docker images | grep n8n
docker ps | grep n8n
```

**Update n8n container:**

```bash
# Stop current container
docker stop n8n

# Remove old container (data persists in ~/.n8n volume)
docker rm n8n

# Start new container with latest image
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -e N8N_SECURE_COOKIE=false \
  -e N8N_HOST=localhost \
  -e N8N_PORT=5678 \
  -e N8N_PROTOCOL=http \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n

# Verify new container is running
docker ps | grep n8n

# Check logs for successful start
docker logs n8n | tail -n 30
```

### 3. Review API Usage & Costs

**Claude API (Anthropic):**

Visit: https://console.anthropic.com/settings/billing

**Check:**
- ✅ Monthly usage (should be under $10 if limit set)
- ✅ Usage trends (sudden spikes may indicate issues)
- ✅ Spending limit still active
- ⚠️ Any anomalies or unexpected charges

**Airtable:**

Visit: https://airtable.com/account

**Check:**
- ✅ API calls remaining (Free tier: 1,000/month)
- ✅ Storage usage
- ⚠️ Approaching any limits

**Gmail API:**

Visit: https://console.cloud.google.com/apis/dashboard

**Check:**
- ✅ Gmail API usage (should be well within free quota)
- ✅ No quota errors
- ⚠️ Unusual usage patterns

### 4. Review Connected Account Permissions

**Google Account:**

Visit: https://myaccount.google.com/permissions

**Verify:**
- ✅ n8n OAuth app still shows as connected
- ✅ "Last accessed" date is recent
- ❌ No unfamiliar apps with access
- ✅ Only necessary permissions granted

**Airtable Tokens:**

Visit: https://airtable.com/create/tokens

**Verify:**
- ✅ n8n token is active
- ✅ Scopes are still appropriate
- ✅ Last used date is recent
- ❌ No unused or old tokens

### 5. Clean Up Docker Resources

```bash
# Show current Docker disk usage
docker system df

# Remove unused images, containers, networks (CAUTION: will ask for confirmation)
docker system prune -a

# Alternative: Just remove unused containers
docker container prune

# Remove unused images only
docker image prune -a
```

**WARNING:** The `-a` flag removes ALL unused images. Make sure you want to do this.

### 6. Check n8n Data Size

```bash
# Check total n8n data directory size
du -sh ~/.n8n

# Break down by subdirectory
du -h --max-depth=1 ~/.n8n | sort -hr

# Check execution data size specifically
du -sh ~/.n8n/.cache/
```

**If execution data is too large:**

Via n8n Web Interface:
1. Go to **Settings** → **Execution Data**
2. Set "Save execution data" to: **Save only on error**
3. Manually clear old successful executions
4. Or set retention period (e.g., 7 days)

### 7. Verify Raspberry Pi Connect Security

Visit: https://connect.raspberrypi.com

**Check:**
- ✅ Only your devices are listed
- ✅ Last accessed times are accurate
- ❌ No unfamiliar devices
- ✅ 2FA is enabled on Raspberry Pi ID

### 8. Test All Workflows

**Manually execute each workflow:**

1. **Transaction Processing Workflow:**
   - Click "Execute Workflow" in n8n
   - Verify: Gmail connection works
   - Verify: Claude API responds
   - Verify: Airtable writes succeed
   - Check execution log for errors

2. **Weekly Report Workflow:**
   - Execute manually
   - Verify: Discord webhook delivers
   - Verify: Calculations are accurate
   - Check report quality

3. **Monthly Report Workflow:**
   - Execute manually
   - Verify: Comprehensive analysis
   - Verify: Discord delivery
   - Check for any errors

---

## 📆 Quarterly Checks (30 minutes)

### 1. Full Security Audit

**System Security:**

```bash
# Check for security updates
sudo apt update
sudo apt list --upgradable | grep -i security

# Review all listening ports
sudo netstat -tlnp

# Expected: Only essential services (Docker, etc.)
# NOT: Unexpected open ports

# Check running processes
ps aux | grep -E 'docker|n8n|node'

# Review system logs for errors
sudo journalctl -p 3 -xb | tail -n 50
# (Shows priority 3 errors from current boot)
```

**Docker Security:**

```bash
# Verify Docker daemon is secure
docker info | grep -i security

# Check container security settings
docker inspect n8n | grep -i security

# Expected: SecurityOpt: null (uses Docker defaults)
```

**Network Security:**

```bash
# Verify no port forwarding to n8n
# (Check your router admin panel - can't be done via command line)

# Verify firewall rules haven't changed
sudo ufw status numbered

# Check for any unusual network connections
sudo netstat -ant | grep ESTABLISHED
```

### 2. Credential & Password Rotation (Optional but Recommended)

**Consider rotating:**

- [ ] Raspberry Pi user password
- [ ] Airtable Personal Access Token
- [ ] Anthropic API key (if compromised or suspected)
- [ ] Discord webhook URL (if needed)

**Gmail OAuth:**
- No need to rotate unless you suspect compromise
- Revoke and re-authorize if needed via Google Account permissions

### 3. Review & Update Documentation

**Create/update your personal notes:**

```bash
# Create maintenance log file
nano ~/n8n-maintenance.log

# Example entry:
# 2025-01-15: Monthly check - all systems normal, updated n8n to v1.x.x
# 2025-02-01: Quarterly audit - rotated Airtable token, no issues found
# 2025-02-15: Weekly check - fixed failed workflow (Gmail OAuth expired)
```

**Document:**
- [ ] Any workflow changes made
- [ ] Custom configurations
- [ ] Lessons learned from issues
- [ ] Backup locations and procedures

### 4. Disaster Recovery Test

**Verify you can recover from failure:**

```bash
# Test 1: Can you restore from backup?
# 1. Create test directory
mkdir -p ~/n8n-test-restore

# 2. Extract latest backup
tar -xzf ~/n8n-backups/n8n-backup-*.tar.gz -C ~/n8n-test-restore

# 3. Verify contents
ls -la ~/n8n-test-restore/.n8n/

# 4. Clean up test
rm -rf ~/n8n-test-restore
```

**Checklist:**
- [ ] Backups are accessible and not corrupted
- [ ] You know how to restore n8n from backup
- [ ] Workflow exports are saved separately
- [ ] API credentials are documented securely (password manager)

### 5. Review n8n Changelog & Updates

**Check for important updates:**

Visit: https://n8n.io/blog or https://github.com/n8n-io/n8n/releases

**Look for:**
- Security patches
- Major version updates
- Deprecated features affecting your workflows
- New features that could improve your automation

**Update strategy:**
- Security patches: Update within 1 week
- Major versions: Test in staging first (or backup before updating)
- Feature updates: Optional, based on need

---

## 🚨 Red Flag Checklist

**Take immediate action if you observe:**

### Critical Issues (Fix Immediately)

❌ **Container repeatedly restarting**
```bash
# Check status
docker ps -a | grep n8n

# If restarting, check logs
docker logs n8n --tail 100

# Common fixes:
# - Check disk space (df -h)
# - Verify permissions (chmod 700 ~/.n8n)
# - Check for corrupted data
```

❌ **Unexpected API charges**
- Check Claude API billing immediately
- Review Airtable usage
- Check for runaway workflows

❌ **Multiple failed login attempts**
```bash
# Check auth logs
sudo grep "Failed password" /var/log/auth.log | tail -n 50

# If attacks detected:
# - Verify SSH is disabled: sudo systemctl status ssh
# - Check firewall: sudo ufw status
# - Consider changing password
```

❌ **Disk space below 10%**
```bash
# Check usage
df -h

# Clean up:
docker system prune -a
rm -rf ~/.n8n/.cache/executions/*
# Clear old backups
```

❌ **Workflows failing repeatedly**
- Check Executions tab in n8n
- Review error messages
- Common causes:
  - API key expiration
  - OAuth token expired (re-authorize)
  - Service API changes
  - Network connectivity issues

### Warning Signs (Investigate Soon)

⚠️ **Unusual network activity**
```bash
# Check bandwidth (install vnstat first)
sudo apt install vnstat
vnstat -d
```

⚠️ **Container using excessive resources**
```bash
docker stats n8n --no-stream

# If CPU/Memory unusually high:
# - Check for workflow loops
# - Review recent workflow changes
# - Consider restarting container
```

⚠️ **Slow workflow execution**
- Check system load: `top` or `htop`
- Review Docker stats
- Check internet connection speed

---

## 📋 Quick Reference Checklist

### Copy this to your maintenance routine:

**WEEKLY (Every Sunday):**
```
□ Check n8n container running (docker ps)
□ Review execution logs for errors
□ Update system packages (apt update/upgrade)
□ Check firewall status (ufw status)
□ Verify disk space (df -h)
□ Review failed auth attempts
```

**MONTHLY (1st of month):**
```
□ Backup workflows (tar command)
□ Download backup to safe location
□ Update n8n Docker image
□ Review Claude API costs
□ Review Airtable usage
□ Check Google OAuth permissions
□ Clean up Docker (docker system prune)
□ Test all workflows manually
□ Check n8n data size
```

**QUARTERLY (Jan/Apr/Jul/Oct):**
```
□ Full security audit
□ Review all listening ports
□ Consider credential rotation
□ Test disaster recovery
□ Review n8n changelog
□ Update documentation
□ Verify 2FA on all accounts
□ Check router (no port forwarding)
```

---

## 🔧 Automation Ideas

### Create a Weekly Check Script

Save this as `~/check-n8n.sh`:

```bash
#!/bin/bash

echo "================================"
echo "n8n Weekly Security Check"
echo "Date: $(date)"
echo "================================"
echo ""

echo "=== Container Status ==="
docker ps | grep n8n
echo ""

echo "=== Disk Space ==="
df -h | grep -E 'Filesystem|/$'
echo ""

echo "=== Recent Errors (Last 10 lines) ==="
docker logs n8n --tail 10 | grep -i error || echo "No errors found"
echo ""

echo "=== Firewall Status ==="
sudo ufw status | head -5
echo ""

echo "=== Failed Login Attempts (Last 5) ==="
sudo grep "Failed password" /var/log/auth.log | tail -n 5 || echo "No failed attempts"
echo ""

echo "=== Network Binding ==="
sudo netstat -tlnp | grep 5678
echo ""

echo "================================"
echo "Check complete!"
echo "================================"
```

**Make it executable:**
```bash
chmod +x ~/check-n8n.sh
```

**Run weekly:**
```bash
./check-n8n.sh
```

## 🛟 Troubleshooting Common Issues

### Container Won't Start

```bash
# Check logs for error details
docker logs n8n

# Common fixes:
# 1. Port conflict
sudo netstat -tlnp | grep 5678
# Kill process using port or change n8n port

# 2. Permission issues
chmod 700 ~/.n8n
chown -R $USER:$USER ~/.n8n

# 3. Corrupted data (backup first!)
mv ~/.n8n ~/.n8n.backup
mkdir ~/.n8n
# Restore from backup
```

### OAuth Not Working

**Gmail OAuth fails:**
- Must authenticate on Pi directly (via Pi Connect)
- Check redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`
- Verify Google OAuth app in "Testing" mode
- Confirm your email is added as test user

### High Memory Usage

```bash
# Check current usage
docker stats n8n --no-stream

# Restart container (clears cache)
docker restart n8n

# If persistent:
# - Review workflow efficiency
# - Check for infinite loops
# - Limit execution data retention
```

---

## 📝 Maintenance Log Template

**Keep track of your maintenance:**

```
# n8n Maintenance Log

## 2025-01-05 - Weekly Check
- Status: All systems normal
- Updated OS packages
- Disk usage: 45%
- No failed login attempts

## 2025-02-01 - Monthly Check
- Backup completed and downloaded
- Updated n8n to v1.23.0
- Claude API usage: $1.50 (well under limit)
- All workflows tested successfully

## 2025-04-01 - Quarterly Audit
- Full security audit completed
- Rotated Airtable PAT
- Tested disaster recovery
- No security issues found
```

---

## ✅ Summary

**Regular maintenance keeps your self-hosted n8n:**
- ✅ Secure from threats
- ✅ Running efficiently
- ✅ Up to date with patches
- ✅ Protected with backups
- ✅ Monitored for issues

---

**Remember:** Security is not a one-time setup, it's an ongoing process. Consistent maintenance catches issues early and keeps your automation running smoothly and securely.

**Questions or improvements to this routine?** Open an issue or submit a pull request!

---

**Last Updated:** December 2024  
**Version:** 1.0  
**License:** MIT
