# 🔧 Troubleshooting Guide

> Common issues and solutions for the Mini SOC/XDR environment

---

## 📋 Quick Navigation

- [Service Issues](#service-issues)
  - [Services Won't Start](#services-wont-start)
  - [Services Keep Stopping](#services-keep-stopping)
  - [Services Running But Not Accessible](#services-running-but-not-accessible)
- [Network Issues](#network-issues)
  - [VMs Can't Communicate](#vms-cant-communicate)
  - [Can't Access Dashboards](#cant-access-dashboards)
  - [IP Addresses Changed](#ip-addresses-changed)
- [Wazuh Issues](#wazuh-issues)
  - [Agent Not Connecting](#agent-not-connecting)
  - [No Events Showing](#no-events-showing)
  - [Dashboard Won't Load](#wazuh-dashboard-wont-load)
- [Prometheus Issues](#prometheus-issues)
  - [Targets Showing DOWN](#targets-showing-down)
  - [No Metrics Data](#no-metrics-data)
  - [Queries Return Empty](#queries-return-empty)
- [Grafana Issues](#grafana-issues)
  - [Can't Log In](#cant-log-in-to-grafana)
  - [Dashboard Shows No Data](#dashboard-shows-no-data)
  - [Data Source Connection Failed](#data-source-connection-failed)
- [Velociraptor Issues](#velociraptor-issues)
  - [Client Not Showing](#client-not-showing)
  - [Client Can't Connect](#client-cant-connect)
  - [Permission Denied Errors](#velociraptor-permission-denied)
- [CrowdSec Issues](#crowdsec-issues)
- [Disk Space Issues](#disk-space-issues)
- [Performance Issues](#performance-issues)
- [Emergency Fixes](#emergency-fixes)

---

## Service Issues

### Services Won't Start

#### Symptom
```bash
sudo systemctl start [service]
# Returns: "Job for [service] failed"
```

#### Solution 1: Check Detailed Status
```bash
# Check full service status
sudo systemctl status [service] --no-pager

# Check recent logs
sudo journalctl -u [service] -n 50 --no-pager

# Check service-specific logs
sudo tail -50 /var/ossec/logs/ossec.log  # Wazuh
sudo tail -50 /var/log/velociraptor/velociraptor.log  # Velociraptor
```

**Look for specific error messages and proceed to relevant section below.**

---

#### Solution 2: Check Disk Space

**Services fail if disk is full!**
```bash
# Check disk usage
df -h

# If > 90% full, see "Disk Space Issues" section
```

---

#### Solution 3: Check RAM
```bash
# Check available memory
free -h

# If very low (< 100MB available), reboot VMs
```

---

#### Solution 4: Check Dependencies

**Some services depend on others:**

**Wazuh Dashboard** needs **Wazuh Indexer**:
```bash
sudo systemctl start wazuh-indexer
sleep 10
sudo systemctl start wazuh-dashboard
```

**Grafana** needs **Prometheus**:
```bash
sudo systemctl start prometheus
sleep 5
sudo systemctl start grafana-server
```

---

### Services Keep Stopping

#### Symptom
Service starts but stops after a few seconds or minutes.

#### Solution 1: Check for Crashes
```bash
# Check if service crashed
sudo systemctl status [service]

# Look for "code=exited, status=X"
# Check crash logs
sudo journalctl -u [service] --since "10 minutes ago"
```

---

#### Solution 2: Check Resource Limits

**Wazuh requires minimum RAM:**
```bash
# Check Wazuh memory usage
ps aux | grep wazuh | awk '{sum+=$6} END {print sum/1024 " MB"}'

# If Wazuh Manager keeps stopping, you may need more RAM
# Increase VM #1 RAM in VirtualBox settings
```

---

#### Solution 3: Check Configuration Errors
```bash
# Test Wazuh config
sudo /var/ossec/bin/wazuh-control check

# Test Prometheus config
sudo promtool check config /etc/prometheus/prometheus.yml

# Test Velociraptor config
sudo velociraptor --config /etc/velociraptor/server.config.yaml config show
```

---

### Services Running But Not Accessible

#### Symptom
`systemctl status` shows "active (running)" but can't access dashboard.

#### Solution: Check Listening Ports
```bash
# Check what's actually listening
sudo netstat -tlnp | grep -E '443|9090|3000|8889'

# Expected output:
# :443  - Wazuh Dashboard
# :9090 - Prometheus
# :3000 - Grafana
# :8889 - Velociraptor
```

**If port not listed:**
- Service may be listening on wrong interface
- Check configuration files for `bind_address` or `listen_address`

---

## Network Issues

### VMs Can't Communicate

#### Symptom
VM #2 can't ping VM #1, or vice versa.

#### Solution 1: Verify Network Configuration
```bash
# On both VMs, check IP addresses
ip addr show | grep "inet " | grep -v 127.0.0.1

# Check routing
ip route
```

**Both VMs should be on the same subnet!**
- ✅ Good: 192.168.53.24 and 192.168.53.92
- ❌ Bad: 192.168.53.24 and 192.168.56.92

---

#### Solution 2: Check VirtualBox Network Settings

**Both VMs must use same network configuration:**

1. **Shut down both VMs**
2. **In VirtualBox, for each VM:**
   - Select VM → Settings → Network
   - Adapter 1: **Attached to: Bridged Adapter**
   - Name: **Same physical network interface** (e.g., your WiFi or Ethernet)
   - Promiscuous Mode: Allow All
3. **Start VMs**
4. **Test ping again**

---

#### Solution 3: Check Firewall
```bash
# Check if firewall is blocking
sudo ufw status

# If active and blocking, you may need to allow traffic
sudo ufw allow from 192.168.53.0/24

# Or temporarily disable for testing
sudo ufw disable
```

---

### Can't Access Dashboards

#### Symptom
Browser can't reach dashboards from host machine.

#### Solution 1: Verify Services Are Listening

**On VM #1:**
```bash
# Check services
sudo systemctl status wazuh-dashboard
sudo systemctl status prometheus
sudo systemctl status grafana-server
sudo systemctl status velociraptor

# Check ports
sudo netstat -tlnp | grep -E '443|9090|3000|8889'
```

---

#### Solution 2: Check Host Machine Can Reach VM

**From your Windows/Mac host:**
```bash
# Ping VM #1
ping 192.168.53.24

# If can't ping, check VirtualBox network settings
```

---

#### Solution 3: Check Browser

**For HTTPS dashboards (Wazuh, Velociraptor):**
- Must accept self-signed certificate warning
- Click "Advanced" → "Proceed to [IP]"

**Try different browser:**
- Chrome/Edge usually best
- Firefox sometimes has stricter security

---

### IP Addresses Changed

#### Symptom
VMs got new IP addresses after reboot (DHCP reassigned).

#### Solution: Update All Configurations

**This is the most tedious issue! You need to update:**

**1. Prometheus Configuration (VM #1):**
```bash
sudo nano /etc/prometheus/prometheus.yml
# Update target IP under "production-server"
sudo systemctl restart prometheus
```

**2. Wazuh Agent Configuration (VM #2):**
```bash
sudo nano /var/ossec/etc/ossec.conf
# Update <address> under <server>
sudo systemctl restart wazuh-agent
```

**3. Velociraptor (if server IP changed):**
- This requires regenerating configurations
- Consider using static IPs to avoid this!

**Prevention: Use Static IPs**

**On each VM:**
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example configuration:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:  # Your interface name
      dhcp4: no
      addresses: [192.168.53.24/24]  # Your desired static IP
      gateway4: 192.168.53.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Apply:
```bash
sudo netplan apply
```

---

## Wazuh Issues

### Agent Not Connecting

#### Symptom
`agent_control -l` doesn't show agent, or shows "Never connected".

#### Solution 1: Check Manager IP Configuration

**On VM #2:**
```bash
# Check agent configuration
sudo grep -A 5 "<server>" /var/ossec/etc/ossec.conf
```

**Should show:**
```xml
<server>
  <address>192.168.53.24</address>  <!-- Your VM #1 IP -->
  <port>1514</port>
  <protocol>tcp</protocol>
</server>
```

**If shows `MANAGER_IP`:**
```bash
sudo nano /var/ossec/etc/ossec.conf
# Change MANAGER_IP to actual IP
sudo systemctl restart wazuh-agent
```

---

#### Solution 2: Verify Connectivity

**From VM #2:**
```bash
# Test connection to manager
ping -c 4 192.168.53.24

# Test Wazuh port
telnet 192.168.53.24 1514
# Should connect (Ctrl+] then 'quit' to exit)
```

---

#### Solution 3: Check Agent Status

**On VM #2:**
```bash
sudo systemctl status wazuh-agent --no-pager
sudo tail -30 /var/ossec/logs/ossec.log
```

**Look for errors like:**
- "Cannot connect to server"
- "Connection refused"
- "Invalid server address"

---

#### Solution 4: Restart Both Manager and Agent
```bash
# On VM #1
sudo systemctl restart wazuh-manager

# Wait 30 seconds

# On VM #2
sudo systemctl restart wazuh-agent

# Wait 30 seconds

# On VM #1, check again
sudo /var/ossec/bin/agent_control -l
```

---

### No Events Showing

#### Symptom
Agent connected but no events appear in dashboard.

#### Solution 1: Check Agent Status in Dashboard

1. **Wazuh Dashboard → Server Management → Endpoints Summary**
2. **Click on agent name**
3. **Check "Last keep alive"** - should be recent

---

#### Solution 2: Generate Test Events

**On VM #2:**
```bash
# Generate authentication events
ssh fakeuser@localhost
# Wrong password 5 times

# Wait 30-60 seconds for processing
```

**Check Dashboard:**
- Menu → Threat Hunting → Events
- Time: Last 15 minutes
- Search: `agent.name: "hazimprod"`

---

#### Solution 3: Check Agent Logs Are Being Collected

**On VM #2:**
```bash
# Check what logs agent is monitoring
sudo grep -A 5 "<localfile>" /var/ossec/etc/ossec.conf | head -20

# Verify log files exist
ls -lh /var/log/auth.log
ls -lh /var/log/syslog
```

---

### Wazuh Dashboard Won't Load

#### Symptom
Browser can't reach `https://192.168.53.24`

#### Solution 1: Check All Wazuh Components
```bash
# Check all three services
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```

**All three must be running!**

---

#### Solution 2: Start Services in Correct Order
```bash
# Stop all
sudo systemctl stop wazuh-dashboard
sudo systemctl stop wazuh-indexer
sudo systemctl stop wazuh-manager

# Start in order
sudo systemctl start wazuh-manager
sleep 10
sudo systemctl start wazuh-indexer
sleep 30  # Indexer needs time to initialize
sudo systemctl start wazuh-dashboard
sleep 10

# Check status
sudo systemctl status wazuh-dashboard
```

---

#### Solution 3: Check Dashboard Logs
```bash
# Check dashboard logs
sudo journalctl -u wazuh-dashboard -n 50

# Check if it's listening
sudo netstat -tlnp | grep 443
```

---

## Prometheus Issues

### Targets Showing DOWN

#### Symptom
`http://192.168.53.24:9090/targets` shows targets as DOWN (red).

#### Solution 1: Check Node Exporter on VM #2
```bash
# On VM #2
sudo systemctl status node_exporter --no-pager

# Test locally
curl http://localhost:9100/metrics
```

---

#### Solution 2: Verify Network Connectivity
```bash
# From VM #1, test connection to VM #2
curl http://192.168.53.92:9100/metrics

# Should return metrics data
```

**If fails:**
- Check network connectivity (ping)
- Check firewall on VM #2
- Verify IP address in Prometheus config

---

#### Solution 3: Check Prometheus Configuration
```bash
# On VM #1
cat /etc/prometheus/prometheus.yml

# Verify target IP is correct under "production-server"
# If wrong, edit and restart:
sudo nano /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

---

### No Metrics Data

#### Symptom
Queries return no data or "No data points".

#### Solution 1: Check Time Range

**In Prometheus UI:**
- Look at time selector (top right)
- Try "Last 5 minutes" or "Last 1 hour"
- Make sure it's not set to past dates

---

#### Solution 2: Verify Target Is UP
```bash
# Check targets page
# http://192.168.53.24:9090/targets
# Should show UP (green)
```

---

#### Solution 3: Test Simple Query

**Try basic query first:**
```
up
```

**Should return:**
```
up{job="prometheus"} = 1
up{job="production-server"} = 1
```

**If this works but other queries don't:**
- Your query syntax might be wrong
- Metric name might be incorrect
- Try autocomplete: Start typing `node_` and see suggestions

---

### Queries Return Empty

#### Symptom
Query executes but shows "Empty query result".

#### Solution: Check Metric Names
```bash
# See all available metrics
curl http://192.168.53.24:9090/api/v1/label/__name__/values | grep node_

# Or in Prometheus UI, type "node_" and see autocomplete suggestions
```

**Common mistakes:**
- Wrong job name: Use `job="production-server"` not `job="vm2"`
- Typos in metric names
- Wrong time range

---

## Grafana Issues

### Can't Log In to Grafana

#### Symptom
Login page appears but credentials don't work.

#### Solution 1: Reset Admin Password

**On VM #1:**
```bash
# Reset admin password
sudo grafana-cli admin reset-admin-password newpassword123

# Or reset to default
sudo grafana-cli admin reset-admin-password admin

# Restart Grafana
sudo systemctl restart grafana-server
```

---

#### Solution 2: Check Grafana Status
```bash
sudo systemctl status grafana-server --no-pager
sudo journalctl -u grafana-server -n 30
```

---

### Dashboard Shows No Data

#### Symptom
Dashboard loads but all panels show "No data".

#### Solution 1: Check Time Range

**Top right corner:**
- Click time range selector
- Select "Last 5 minutes" or "Last 1 hour"
- Click "Refresh" icon

---

#### Solution 2: Check Data Source

**In dashboard:**
1. Click any panel title → **Edit**
2. Look at **Query inspector** (bottom)
3. Check if query returns data
4. Check data source is "Prometheus"

---

#### Solution 3: Check Variables

**Some dashboards have variables:**
- Look for dropdowns at top of dashboard
- Try selecting different options
- Some dashboards need hostname/instance selected

---

### Data Source Connection Failed

#### Symptom
Grafana says "Data source connection failed" or "Bad Gateway".

#### Solution: Reconfigure Prometheus Data Source

1. **☰ menu → Connections → Data sources**
2. **Click "Prometheus"**
3. **Verify URL:** `http://localhost:9090`
4. **Scroll down → "Save & Test"**
5. **Should see:** ✅ "Data source is working"

**If still fails:**
```bash
# Check Prometheus is running
sudo systemctl status prometheus

# Check Prometheus is accessible
curl http://localhost:9090/api/v1/status/config
```

---

## Velociraptor Issues

### Client Not Showing

#### Symptom
Velociraptor GUI doesn't show VM #2 client.

#### Solution 1: Check Client Service

**On VM #2:**
```bash
sudo systemctl status velociraptor-client --no-pager
sudo journalctl -u velociraptor-client -n 30
```

---

#### Solution 2: Verify Connectivity

**From VM #2:**
```bash
# Test connection to server
ping -c 4 192.168.53.24

# Test Velociraptor port
telnet 192.168.53.24 8000
```

---

#### Solution 3: Refresh GUI

**In Velociraptor web interface:**
- Press **F5** to refresh
- Click **"Show All"** again
- Wait 10-20 seconds
- Check **"Offline clients"** tab

---

### Client Can't Connect

#### Symptom
Client service running but shows connection errors in logs.

#### Solution 1: Check Server Configuration

**On VM #1:**
```bash
# Verify server is listening
sudo netstat -tlnp | grep 8000

# Check server logs
sudo journalctl -u velociraptor -n 50
```

---

#### Solution 2: Check Client Configuration

**On VM #2:**
```bash
# Verify client config exists
ls -lh /etc/velociraptor/client.config.yaml

# Check config has correct server IP
sudo grep -i "server_urls" /etc/velociraptor/client.config.yaml
```

**Should show:**
```yaml
server_urls:
  - https://192.168.53.24:8000/
```

---

### Velociraptor Permission Denied

#### Symptom
`Permission denied` errors when starting client.

#### Solution: Fix Permissions
```bash
# On VM #2
sudo chown velociraptor:velociraptor /etc/velociraptor/client.config.yaml
sudo touch /etc/velociraptor.writeback.yaml
sudo chown velociraptor:velociraptor /etc/velociraptor.writeback.yaml

# Create data directory
sudo mkdir -p /var/lib/velociraptor-client
sudo chown velociraptor:velociraptor /var/lib/velociraptor-client

# Restart client
sudo systemctl restart velociraptor-client
```

---

## CrowdSec Issues

### CrowdSec Not Running
```bash
# Check status
sudo systemctl status crowdsec

# Check logs
sudo tail -50 /var/log/crowdsec.log

# Restart
sudo systemctl restart crowdsec
```

---

### No Metrics Showing
```bash
# Verify CrowdSec is actually monitoring
sudo cscli metrics

# Check if log files exist
ls -lh /var/log/auth.log
ls -lh /var/log/syslog

# Check acquisitions
sudo cscli collections list
sudo cscli scenarios list
```

---

## Disk Space Issues

### Disk Full Error

#### Symptom
```
No space left on device
```

#### Solution 1: Check Disk Usage
```bash
# Check overall usage
df -h

# Find large directories
sudo du -h --max-depth=1 / 2>/dev/null | sort -h | tail -20

# Check specific locations
sudo du -sh /var/ossec/queue/*
sudo du -sh /var/lib/prometheus/*
sudo du -sh /var/log/*
```

---

#### Solution 2: Clean Wazuh Queue (Most Common!)

**Wazuh vulnerability detector cache can grow huge:**
```bash
# Stop Wazuh
sudo systemctl stop wazuh-manager

# Clean vulnerability detector cache
sudo rm -rf /var/ossec/queue/vd_updater/*

# Clean other queue directories
sudo rm -rf /var/ossec/queue/diff/*
sudo rm -rf /var/ossec/queue/fim/db/*

# Start Wazuh
sudo systemctl start wazuh-manager

# Check space
df -h
```

---

#### Solution 3: Clean Journal Logs
```bash
# Check journal size
sudo journalctl --disk-usage

# Clean old journals
sudo journalctl --vacuum-size=100M

# Or keep last 2 days only
sudo journalctl --vacuum-time=2d
```

---

#### Solution 4: Clean APT Cache
```bash
sudo apt clean
sudo apt autoclean
sudo apt autoremove
```

---

#### Solution 5: Expand Disk (If Necessary)

**In VirtualBox:**
1. **Shut down VM**
2. **File → Virtual Media Manager**
3. **Select VM disk → Increase size**
4. **Start VM**
5. **Resize partition:**
```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h
```

---

## Performance Issues

### VMs Running Slow

#### Solution 1: Check RAM Usage
```bash
# Check memory
free -h

# Check swap usage
swapon --show

# Check top processes
top
# Press 'M' to sort by memory
```

**If RAM is exhausted:**
- Close one VM temporarily
- Increase RAM in VirtualBox settings
- Stop unnecessary services

---

#### Solution 2: Reduce Services

**On VM #1, you can temporarily stop Grafana:**
```bash
# Grafana is nice but not essential
sudo systemctl stop grafana-server

# Use Prometheus UI instead
```

---

#### Solution 3: Check Host Machine

**On your Windows/Mac:**
- Close unnecessary applications
- Close browser tabs
- Check Task Manager/Activity Monitor
- Make sure host has available RAM

---

### Host Machine Freezing

**If your laptop freezes with both VMs running:**

**Short-term fix:**
- Shut down one VM
- Work on one VM at a time
- Use screenshots for demos

**Long-term fix:**
- Reduce VM #1 RAM to 2GB
- Reduce VM #2 RAM to 1GB
- Adjust VM CPU allocation
- Consider upgrading host RAM

---

## Emergency Fixes

### Nothing Works - Nuclear Option

**When everything is broken:**
```bash
# On each VM:

# 1. Reboot
sudo reboot

# 2. After reboot, start services
~/start-soc.sh  # VM #1
~/start-agents.sh  # VM #2

# 3. Check status
~/check-soc-status.sh  # VM #1
~/check-agents-status.sh  # VM #2

# 4. If still broken, check logs
sudo journalctl -xe
```

---

### Services Won't Start After Reboot

**Common cause: Disk space filled during shutdown**
```bash
# Check disk space FIRST
df -h

# If > 90% full, clean Wazuh queue
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater/*
sudo systemctl start wazuh-manager
```

---

### Can't SSH into VM

**If VM is running but can't SSH:**

1. **Use VirtualBox console:**
   - Right-click VM → Show
   - Log in directly
   
2. **Check SSH service:**
```bash
sudo systemctl status ssh
sudo systemctl start ssh
```

3. **Check network:**
```bash
ip addr show
# Verify IP address
```

---

### Dashboard Shows 502 Bad Gateway

**Wazuh Indexer not ready:**
```bash
# Indexer needs time to start
sudo systemctl restart wazuh-indexer
# Wait 30 seconds

sudo systemctl restart wazuh-dashboard
# Wait 15 seconds

# Try accessing again
```

---

### Complete Reset (Last Resort)

**If you need to start over:**
```bash
# Backup your scripts first!
cp ~/start-soc.sh ~/start-soc.sh.backup
cp ~/start-agents.sh ~/start-agents.sh.backup

# Then reinstall problematic service
# See SETUP.md for installation instructions
```

**Or restore from VirtualBox snapshot if you created one!**

---

## Diagnostic Commands Cheat Sheet

**Copy these for quick troubleshooting:**
```bash
# System resources
df -h                    # Disk space
free -h                  # RAM
top                      # CPU/Memory usage
htop                     # Better process viewer

# Network
ip addr show             # IP addresses
ping 192.168.53.24       # Test connectivity
telnet IP PORT           # Test port connectivity
sudo netstat -tlnp       # Listening ports

# Services
sudo systemctl status SERVICE           # Service status
sudo journalctl -u SERVICE -n 50       # Recent logs
sudo systemctl restart SERVICE          # Restart service

# Wazuh specific
sudo /var/ossec/bin/agent_control -l   # List agents
sudo tail -f /var/ossec/logs/ossec.log # Watch logs live

# Prometheus
curl http://localhost:9090/api/v1/targets  # Check targets

# Quick test all services
~/check-soc-status.sh    # VM #1
~/check-agents-status.sh # VM #2
```

---

## Getting Help

**If you're still stuck:**

1. **Check logs carefully** - Error messages usually tell you what's wrong
2. **Search the error message** - Google it!
3. **Check official documentation:**
   - [Wazuh Docs](https://documentation.wazuh.com/)
   - [Prometheus Docs](https://prometheus.io/docs/)
   - [Grafana Docs](https://grafana.com/docs/)
4. **AI it! lol** 

---

## Prevention Tips

**Avoid issues before they happen:**

✅ **Monitor disk space regularly:** `df -h`  
✅ **Clean Wazuh queue monthly:** `sudo rm -rf /var/ossec/queue/vd_updater/*`  
✅ **Take VirtualBox snapshots** before major changes  
✅ **Document your specific IPs** in case they change  
✅ **Test after every change**  
✅ **Keep notes** of what you fixed  

---

## Related Documentation

- [Installation Guide](SETUP.md)
- [Demo Guide](DEMOS.md)
- [Command Reference](COMMANDS.md)
- [README](README.md)

---

**Built with 💙 for Cybersecurity Malaysia Internship**

*Remember: Most issues are fixable with a restart and checking logs!*
