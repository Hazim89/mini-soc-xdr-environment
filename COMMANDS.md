# ⚡ Command Reference

> Quick command cheat sheet for daily operations

---

## 📋 Quick Navigation

- [VM Identification](#vm-identification)
- [Startup & Shutdown](#startup--shutdown)
- [Status Checks](#status-checks)
- [Service Management](#service-management)
- [Wazuh Commands](#wazuh-commands)
- [Prometheus Queries](#prometheus-queries)
- [Grafana Commands](#grafana-commands)
- [Velociraptor Commands](#velociraptor-commands)
- [CrowdSec Commands](#crowdsec-commands)
- [Network & System](#network--system)
- [Troubleshooting Quick Fixes](#troubleshooting-quick-fixes)

---

## VM Identification

**Know which VM you're on:**
```bash
# Check hostname
hostname

# Check IP address
ip addr show | grep "inet " | grep -v 127.0.0.1

# Check RAM
free -h

# Check disk space
df -h
```

**Expected values:**
- **VM #1:** `hazim` / ~2.5GB RAM / ~28GB disk
- **VM #2:** `hazimprod` / ~1.5GB RAM / ~13GB disk

---

## Startup & Shutdown

### Quick Start - VM #1 (SOC Server)
```bash
# Start all SOC services
~/start-soc.sh

# Check all services
~/check-soc-status.sh
```

---

### Quick Start - VM #2 (Production Server)
```bash
# Start all agents
~/start-agents.sh

# Check all agents
~/check-agents-status.sh
```

---

### Manual Startup - VM #1
```bash
# Start services in order
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
sudo systemctl start prometheus
sudo systemctl start grafana-server
sudo systemctl start velociraptor
```

---

### Manual Startup - VM #2
```bash
# Start all agents
sudo systemctl start wazuh-agent
sudo systemctl start node_exporter
sudo systemctl start crowdsec
sudo systemctl start velociraptor-client
```

---

### Shutdown Commands
```bash
# Graceful shutdown
sudo shutdown -h now

# Reboot
sudo reboot

# Shutdown after 5 minutes
sudo shutdown +5

# Cancel scheduled shutdown
sudo shutdown -c
```

**Recommended shutdown order:**
1. VM #2 first (agents)
2. VM #1 second (servers)

---

## Status Checks

### Quick Status - All Services

**VM #1:**
```bash
# All SOC services
~/check-soc-status.sh
```

**VM #2:**
```bash
# All agents
~/check-agents-status.sh
```

---

### Individual Service Status
```bash
# Check any service
sudo systemctl status SERVICE_NAME --no-pager

# Examples:
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status prometheus --no-pager
sudo systemctl status grafana-server --no-pager
sudo systemctl status velociraptor --no-pager
```

---

### Check if Service is Running
```bash
# Quick check (just Active line)
sudo systemctl is-active SERVICE_NAME

# Returns: active, inactive, or failed
```

---

### Check All Services at Once - VM #1
```bash
for service in wazuh-manager wazuh-indexer wazuh-dashboard prometheus grafana-server velociraptor; do
  echo -n "$service: "
  sudo systemctl is-active $service
done
```

---

### Check All Services at Once - VM #2
```bash
for service in wazuh-agent node_exporter crowdsec velociraptor-client; do
  echo -n "$service: "
  sudo systemctl is-active $service
done
```

---

## Service Management

### Start/Stop/Restart Services
```bash
# Start
sudo systemctl start SERVICE_NAME

# Stop
sudo systemctl stop SERVICE_NAME

# Restart
sudo systemctl restart SERVICE_NAME

# Reload config (if supported)
sudo systemctl reload SERVICE_NAME
```

---

### Enable/Disable Auto-Start
```bash
# Enable (start on boot)
sudo systemctl enable SERVICE_NAME

# Disable
sudo systemctl disable SERVICE_NAME

# Check if enabled
sudo systemctl is-enabled SERVICE_NAME
```

---

### View Service Logs
```bash
# Recent logs (last 50 lines)
sudo journalctl -u SERVICE_NAME -n 50 --no-pager

# Follow logs live
sudo journalctl -u SERVICE_NAME -f

# Logs from last 10 minutes
sudo journalctl -u SERVICE_NAME --since "10 minutes ago"

# Logs from today
sudo journalctl -u SERVICE_NAME --since today
```

---

## Wazuh Commands

### Agent Management (VM #1)
```bash
# List all agents
sudo /var/ossec/bin/agent_control -l

# Show agent info
sudo /var/ossec/bin/agent_control -i AGENT_ID

# Example:
sudo /var/ossec/bin/agent_control -i 001

# Restart manager
sudo systemctl restart wazuh-manager
```

---

### Agent Operations (VM #2)
```bash
# Check agent status
sudo systemctl status wazuh-agent --no-pager

# Restart agent
sudo systemctl restart wazuh-agent

# View agent logs
sudo tail -50 /var/ossec/logs/ossec.log

# Follow logs live
sudo tail -f /var/ossec/logs/ossec.log

# Check agent configuration
sudo cat /var/ossec/etc/ossec.conf | grep -A 5 "<server>"
```

---

### Wazuh Troubleshooting
```bash
# Check configuration
sudo /var/ossec/bin/wazuh-control check

# Check service status (all components)
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager

# Check disk space (common issue!)
df -h

# Clean Wazuh queue if disk full
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater/*
sudo systemctl start wazuh-manager
```

---

### Access Wazuh Dashboard
```
https://192.168.53.24

Username: admin
Password: yT2qI0O7smhOyf6r1TU+2rOkj734M26
```

---

## Prometheus Queries

### Access Prometheus
```
http://192.168.53.24:9090
```

---

### Check Targets
```
# Direct URL
http://192.168.53.24:9090/targets

# Or: Status → Targets
```

---

### Common PromQL Queries

**CPU Usage:**
```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle",job="production-server"}[5m])) * 100)
```

**Memory Usage:**
```promql
100 * (1 - ((node_memory_MemAvailable_bytes{job="production-server"}) / (node_memory_MemTotal_bytes{job="production-server"})))
```

**Disk Usage:**
```promql
100 - ((node_filesystem_avail_bytes{job="production-server",mountpoint="/"} * 100) / node_filesystem_size_bytes{job="production-server",mountpoint="/"})
```

**System CPU (for attacks):**
```promql
rate(node_cpu_seconds_total{job="production-server",mode="system"}[2m])
```

**Network Connections:**
```promql
node_netstat_Tcp_CurrEstab{job="production-server"}
```

**Check if targets are up:**
```promql
up
```

**Uptime:**
```promql
node_time_seconds{job="production-server"} - node_boot_time_seconds{job="production-server"}
```

---

### Prometheus Service Commands (VM #1)
```bash
# Restart Prometheus
sudo systemctl restart prometheus

# Check config syntax
sudo promtool check config /etc/prometheus/prometheus.yml

# View current config
cat /etc/prometheus/prometheus.yml

# Edit config
sudo nano /etc/prometheus/prometheus.yml

# Test query from command line
curl 'http://localhost:9090/api/v1/query?query=up'
```

---

## Grafana Commands

### Access Grafana
```
http://192.168.53.24:3000

Username: admin
Password: 727599
```

---

### Grafana Service Commands (VM #1)
```bash
# Restart Grafana
sudo systemctl restart grafana-server

# Check status
sudo systemctl status grafana-server --no-pager

# View logs
sudo journalctl -u grafana-server -n 50

# Reset admin password
sudo grafana-cli admin reset-admin-password newpassword123

# Check listening port
sudo netstat -tlnp | grep 3000
```

---

### Import Dashboard

**In Grafana UI:**
1. ☰ menu → Dashboards → Import
2. Enter dashboard ID: `1860`
3. Load → Select Prometheus data source → Import

**Or via API:**
```bash
curl -X POST http://admin:PASSWORD@localhost:3000/api/dashboards/import \
  -H "Content-Type: application/json" \
  -d '{"dashboard":{"id":1860},"overwrite":true}'
```

---

## Velociraptor Commands

### Access Velociraptor
```
https://192.168.53.24:8889

Username: admin
Password: [Your Velociraptor password]
```

---

### Velociraptor Server Commands (VM #1)
```bash
# Restart server
sudo systemctl restart velociraptor

# Check status
sudo systemctl status velociraptor --no-pager

# View logs
sudo journalctl -u velociraptor -n 50

# Check listening ports
sudo netstat -tlnp | grep velociraptor

# Check configuration
sudo cat /etc/velociraptor/server.config.yaml | grep -E "bind_address|bind_port|public_url"

# Add new admin user
sudo velociraptor --config /etc/velociraptor/server.config.yaml user add USERNAME --role administrator
```

---

### Velociraptor Client Commands (VM #2)
```bash
# Restart client
sudo systemctl restart velociraptor-client

# Check status
sudo systemctl status velociraptor-client --no-pager

# View logs
sudo journalctl -u velociraptor-client -n 50

# Check client can reach server
telnet 192.168.53.24 8000

# View client configuration
sudo cat /etc/velociraptor/client.config.yaml | grep server_urls
```

---

### Common Artifacts to Collect

**In Velociraptor GUI:**
- `Generic.Client.Info` - System information
- `Generic.System.Pstree` - Process tree
- `Linux.Collection.AuthLogs` - Authentication logs
- `Linux.Sys.LastUserLogin` - Login history
- `Linux.Network.Netstat` - Network connections
- `Linux.Sys.Users` - User accounts

---

## CrowdSec Commands

### CrowdSec Status (VM #2)
```bash
# Check status
sudo systemctl status crowdsec --no-pager

# Restart
sudo systemctl restart crowdsec

# View metrics
sudo cscli metrics

# View logs
sudo tail -50 /var/log/crowdsec.log
```

---

### View Decisions (Bans)
```bash
# List all active decisions
sudo cscli decisions list

# List decisions for specific IP
sudo cscli decisions list --ip 192.168.1.100

# Delete decision (unban)
sudo cscli decisions delete --ip 192.168.1.100
```

---

### View Alerts
```bash
# List recent alerts
sudo cscli alerts list

# List last 20 alerts
sudo cscli alerts list | head -20

# View alert details
sudo cscli alerts inspect ALERT_ID
```

---

### Scenarios & Collections
```bash
# List installed scenarios
sudo cscli scenarios list

# List installed collections
sudo cscli collections list

# Update hub
sudo cscli hub update

# Upgrade installed items
sudo cscli hub upgrade

# List available scenarios
sudo cscli scenarios list -a
```

---

### CrowdSec Configuration
```bash
# View acquisition config (what logs are monitored)
sudo cat /etc/crowdsec/acquis.yaml

# View main config
sudo cat /etc/crowdsec/config.yaml

# Test configuration
sudo cscli config validate
```

---

## Network & System

### Check IP Address
```bash
# Show all IPs
ip addr show

# Show only main IP
ip addr show | grep "inet " | grep -v 127.0.0.1

# Alternative
hostname -I
```

---

### Network Connectivity
```bash
# Ping other VM
ping -c 4 192.168.53.24   # Ping VM #1 from VM #2
ping -c 4 192.168.53.92   # Ping VM #2 from VM #1

# Test specific port
telnet 192.168.53.24 9090    # Test Prometheus
telnet 192.168.53.92 9100    # Test Node Exporter

# Check listening ports
sudo netstat -tlnp

# Check specific port
sudo netstat -tlnp | grep 9090

# Alternative (ss command)
sudo ss -tlnp
```

---

### DNS & Routing
```bash
# Check routing table
ip route

# Check DNS resolution
nslookup google.com

# Test internet connectivity
ping -c 4 8.8.8.8
```

---

### System Resources
```bash
# Check RAM usage
free -h

# Check disk space
df -h

# Check disk space with inodes
df -ih

# Show top processes by memory
ps aux --sort=-%mem | head -10

# Show top processes by CPU
ps aux --sort=-%cpu | head -10

# Interactive process viewer
top
# Press 'M' to sort by memory
# Press 'P' to sort by CPU
# Press 'q' to quit

# Better process viewer (if installed)
htop
```

---

### Find Large Files/Directories
```bash
# Find largest directories in /var
sudo du -h --max-depth=1 /var | sort -h | tail -10

# Find files larger than 100MB
sudo find / -type f -size +100M 2>/dev/null

# Check specific Wazuh queue (common issue!)
sudo du -sh /var/ossec/queue/*
```

---

### System Information
```bash
# OS version
cat /etc/os-release

# Kernel version
uname -r

# System uptime
uptime

# CPU information
lscpu

# Memory information
free -h

# Disk information
lsblk
```

---

## Troubleshooting Quick Fixes

### Service Won't Start
```bash
# Check status and logs
sudo systemctl status SERVICE_NAME --no-pager
sudo journalctl -u SERVICE_NAME -n 50 --no-pager

# Check disk space (most common issue!)
df -h

# Restart service
sudo systemctl restart SERVICE_NAME
```

---

### Agent Not Connecting
```bash
# On VM #2 - Check agent config
sudo grep -A 5 "<server>" /var/ossec/etc/ossec.conf

# Test connectivity
ping -c 4 192.168.53.24
telnet 192.168.53.24 1514

# Restart both
# On VM #1:
sudo systemctl restart wazuh-manager

# On VM #2:
sudo systemctl restart wazuh-agent
```

---

### Disk Space Full
```bash
# Check what's using space
df -h
sudo du -h --max-depth=1 / 2>/dev/null | sort -h | tail -20

# Clean Wazuh queue (COMMON FIX!)
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater/*
sudo systemctl start wazuh-manager

# Clean journal logs
sudo journalctl --vacuum-size=100M

# Clean APT cache
sudo apt clean
```

---

### Prometheus Target Down
```bash
# On VM #2 - Check Node Exporter
sudo systemctl status node_exporter
curl http://localhost:9100/metrics

# From VM #1 - Test connection
curl http://192.168.53.92:9100/metrics

# Check Prometheus config
cat /etc/prometheus/prometheus.yml

# Restart Prometheus
sudo systemctl restart prometheus
```

---

### Dashboard Won't Load
```bash
# Wazuh - Restart all components in order
sudo systemctl stop wazuh-dashboard
sudo systemctl restart wazuh-indexer
sleep 30
sudo systemctl restart wazuh-dashboard

# Grafana - Reset password
sudo grafana-cli admin reset-admin-password admin
sudo systemctl restart grafana-server

# Check if listening
sudo netstat -tlnp | grep -E "443|3000|9090|8889"
```

---

### Complete System Reset
```bash
# Nuclear option - reboot everything
sudo reboot

# After reboot:
~/start-soc.sh      # VM #1
~/start-agents.sh   # VM #2

# Check everything
~/check-soc-status.sh    # VM #1
~/check-agents-status.sh # VM #2
```

---

## Demo Commands

### Generate SSH Brute Force Attack
```bash
# On VM #2 - Failed logins
for i in {1..10}; do 
  echo "Attack attempt $i"
  ssh attacker$i@localhost
  sleep 1
done

# Press Ctrl+C after 5-10 attempts
```

---

### Generate CPU Load
```bash
# On VM #2 - Install stress tool
sudo apt install stress-ng -y

# Generate load for 30 seconds
stress-ng --cpu 2 --timeout 30s
```

---

### Test File Integrity Monitoring
```bash
# On VM #2 - Modify monitored file
echo "test change for demo" | sudo tee -a /etc/passwd

# Wait 2-3 minutes, then check Wazuh Dashboard
```

---

## Useful One-Liners
```bash
# Check all service status in one go (VM #1)
echo "Wazuh Manager:" && sudo systemctl is-active wazuh-manager && \
echo "Prometheus:" && sudo systemctl is-active prometheus && \
echo "Grafana:" && sudo systemctl is-active grafana-server && \
echo "Velociraptor:" && sudo systemctl is-active velociraptor

# Check all agents status (VM #2)
echo "Wazuh Agent:" && sudo systemctl is-active wazuh-agent && \
echo "Node Exporter:" && sudo systemctl is-active node_exporter && \
echo "CrowdSec:" && sudo systemctl is-active crowdsec && \
echo "Velociraptor Client:" && sudo systemctl is-active velociraptor-client

# Show all listening ports
sudo netstat -tlnp | grep LISTEN

# Find process using specific port
sudo lsof -i :9090

# Watch disk space (updates every 2 seconds)
watch -n 2 df -h

# Monitor logs live (all services)
sudo journalctl -f

# Get public IP
curl ifconfig.me

# Check total system uptime
uptime -p
```

---

## Quick Access URLs

**Copy-paste ready!**
```
# Wazuh Dashboard
https://192.168.53.24

# Prometheus
http://192.168.53.24:9090

# Prometheus Targets
http://192.168.53.24:9090/targets

# Grafana
http://192.168.53.24:3000

# Velociraptor
https://192.168.53.24:8889
```

---

## Pre-Demo Checklist

**Before presenting, run these:**
```bash
# VM #1 - Start and verify
~/start-soc.sh
sleep 30
~/check-soc-status.sh

# VM #2 - Start and verify
~/start-agents.sh
sleep 10
~/check-agents-status.sh

# Check connectivity
ping -c 2 192.168.53.24   # From VM #2
ping -c 2 192.168.53.92   # From VM #1

# Check disk space
df -h

# Open all dashboards in browser
# - Wazuh, Prometheus, Grafana, Velociraptor
```

---

## Pro Tips

**Time-saving aliases** (add to `~/.bashrc`):
```bash
# Quick status checks
alias soc-status='~/check-soc-status.sh'
alias agent-status='~/check-agents-status.sh'

# Quick service management
alias wazuh-restart='sudo systemctl restart wazuh-manager'
alias prom-restart='sudo systemctl restart prometheus'

# Quick disk check
alias diskspace='df -h | grep -E "Filesystem|/dev/mapper"'

# Quick log viewing
alias wazuh-logs='sudo tail -50 /var/ossec/logs/ossec.log'
alias prom-logs='sudo journalctl -u prometheus -n 50 --no-pager'

# Apply aliases
source ~/.bashrc
```

---

## Related Documentation

- [Installation Guide](SETUP.md)
- [Demo Guide](DEMOS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
- [README](README.md)

---

**Built with 💙 for Cybersecurity Malaysia Internship**

