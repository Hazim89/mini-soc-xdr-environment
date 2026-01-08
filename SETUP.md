# 🔧 Installation & Setup Guide

> Complete step-by-step guide to build your own Mini SOC/XDR environment from scratch

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [System Architecture](#system-architecture)
- [VM Preparation](#vm-preparation)
- [VM #1: SOC Server Setup](#vm-1-soc-server-setup)
  - [Wazuh Installation](#1-wazuh-installation)
  - [Prometheus Installation](#2-prometheus-installation)
  - [Grafana Installation](#3-grafana-installation)
  - [Velociraptor Server Installation](#4-velociraptor-server-installation)
- [VM #2: Production Server Setup](#vm-2-production-server-setup)
  - [Wazuh Agent Installation](#1-wazuh-agent-installation)
  - [Node Exporter Installation](#2-node-exporter-installation)
  - [CrowdSec Installation](#3-crowdsec-installation)
  - [Velociraptor Client Installation](#4-velociraptor-client-installation)
- [Configuration](#configuration)
- [Verification](#verification)
- [Startup Scripts](#startup-scripts)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Hardware Requirements

**Host Machine:**
- **CPU:** Quad-core or better
- **RAM:** 8GB minimum (16GB recommended)
- **Disk:** 50GB+ free space
- **OS:** Windows, macOS, or Linux

**Note:** This setup was successfully deployed on an 8GB laptop through careful resource optimization.

**Note:** Laptop with higher specifications can accommodate higher VM settings.

### Software Requirements

- **VirtualBox** 7.0+ (or VMware Workstation)
- **Ubuntu Server 24.04 LTS** ISO
- **Internet connection** for both VMs
- **Basic Linux command-line knowledge**

### Network Requirements

- Bridged networking capability
- DHCP available on host network
- No firewall blocking VM-to-VM communication

---

## System Architecture

### Two-VM Setup
```
┌─────────────────────────────────────────────────────────────┐
│                         HOST MACHINE                        │
│                        (8GB RAM Laptop)                     │
│                                                             │
│  ┌────────────────────────┐  ┌──────────────────────────┐   │
│  │   VM #1: SOC Server    │  │  VM #2: Production       │   │
│  │                        │  │       Server             │   │
│  │  • Wazuh Manager       │◄─┤  • Wazuh Agent           │   │
│  │  • Prometheus          │◄─┤  • Node Exporter         │   │
│  │  • Grafana             │  │  • CrowdSec              │   │
│  │  • Velociraptor Server │◄─┤  • Velociraptor Client   │   │
│  │                        │  │                          │   │
│  │  RAM: 2.5GB            │  │  RAM: 1.5GB              │   │
│  │  Disk: 28GB            │  │  Disk: 13GB              │   │
│  │  IP: 192.168.53.24     │  │  IP: 192.168.53.92       │   │
│  └────────────────────────┘  └──────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## VM Preparation

### VM #1: SOC Server Specifications

**Create new VM in VirtualBox:**

1. **Name:** `SOC-Server` (or similar)
2. **Type:** Linux
3. **Version:** Ubuntu (64-bit)
4. **Memory:** 2560 MB (2.5GB)
5. **Hard Disk:** 
   - Create virtual hard disk (VDI)
   - Dynamically allocated
   - **30GB** (we'll use ~28GB)
6. **CPU:** 2 cores
7. **Network:** 
   - Adapter 1: Bridged Adapter
   - Select your active network interface

**Install Ubuntu 24.04 LTS:**
- Download from: https://ubuntu.com/download/server
- Mount ISO and install
- Choose minimal installation
- Enable OpenSSH server during installation
- Create user (e.g., `hazim89`)

---

### VM #2: Production Server Specifications

**Create new VM in VirtualBox:**

1. **Name:** `Production-Server`
2. **Type:** Linux
3. **Version:** Ubuntu (64-bit)
4. **Memory:** 1536 MB (1.5GB)
5. **Hard Disk:** 
   - Create virtual hard disk (VDI)
   - Dynamically allocated
   - **15GB**
6. **CPU:** 1-2 cores
7. **Network:** 
   - Adapter 1: Bridged Adapter
   - **Must be on same network as VM #1**

**Install Ubuntu 24.04 LTS** (same as VM #1)

---

### Network Verification

**After both VMs are installed:**

**On VM #1:**
```bash
ip addr show | grep "inet " | grep -v 127.0.0.1
# Note the IP address (e.g., 192.168.53.24)
```

**On VM #2:**
```bash
ip addr show | grep "inet " | grep -v 127.0.0.1
# Note the IP address (e.g., 192.168.53.92)
# Must be on same subnet as VM #1!
```

**Test connectivity:**
```bash
# From VM #2, ping VM #1
ping -c 4 192.168.53.24  # Use your VM #1 IP
```

**If VMs can't ping each other:**
- Check both are on Bridged Adapter
- Check same network interface selected
- Restart VMs

---

## VM #1: SOC Server Setup

### System Preparation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install basic tools
sudo apt install -y curl wget vim net-tools

# Check available resources
free -h
df -h
```

---

### 1. Wazuh Installation

**Wazuh requires significant disk space. We'll install natively (not Docker).**

**Note:**  Because I am using a 8GB RAM, Laptop with >8GB RAM can run docker no problem.

#### Step 1: Download Wazuh Install Script
```bash
cd /tmp
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
```

#### Step 2: Install Wazuh (All-in-One)
```bash
# -a = install all components (Manager, Indexer, Dashboard)
# -i = ignore RAM requirement (we have 2.5GB, it wants 4GB)
sudo bash ./wazuh-install.sh -a -i
```

**Installation takes 10-15 minutes.**

#### Step 3: Save Credentials
```bash
# Credentials are saved here
sudo tar -xvf /root/wazuh-install-files.tar -C /tmp
cat /tmp/wazuh-install-files/wazuh-passwords.txt
```

**Save these credentials securely!**

#### Step 4: Verify Installation
```bash
# Check services
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

# All should show: active (running)
```

#### Step 5: Access Dashboard

Open browser: `https://[VM1-IP]`
- Example: `https://192.168.53.24`
- Accept SSL warning (self-signed certificate)
- Login with admin credentials

**✅ Wazuh Dashboard should load!**

---

### 2. Prometheus Installation

#### Step 1: Create Prometheus User
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

#### Step 2: Download Prometheus
```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-2.48.0.linux-amd64.tar.gz
tar -xvf prometheus-2.48.0.linux-amd64.tar.gz
cd prometheus-2.48.0.linux-amd64
```

#### Step 3: Install Binaries
```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

#### Step 4: Create Directories
```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

#### Step 5: Copy Configuration Files
```bash
sudo cp -r consoles /etc/prometheus/
sudo cp -r console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

#### Step 6: Create Configuration File
```bash
sudo nano /etc/prometheus/prometheus.yml
```

**Paste this configuration:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'production-server'
    static_configs:
      - targets: ['192.168.53.92:9100']  # Change to your VM #2 IP
```

**Save and exit** (Ctrl+X, Y, Enter)
```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

#### Step 7: Create Systemd Service
```bash
sudo nano /etc/systemd/system/prometheus.service
```

**Paste:**
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

#### Step 8: Start Prometheus
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

#### Step 9: Verify

Open browser: `http://[VM1-IP]:9090`
- Example: `http://192.168.53.24:9090`

**✅ Prometheus UI should load!**

---

### 3. Grafana Installation

#### Step 1: Add Grafana Repository
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

#### Step 2: Install Grafana
```bash
sudo apt update
sudo apt install -y grafana
```

#### Step 3: Start Grafana
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

#### Step 4: Access Grafana

Open browser: `http://[VM1-IP]:3000`
- Default login: `admin` / `admin`
- **Change password when prompted**

#### Step 5: Add Prometheus Data Source

1. Click **☰ menu** → **Connections** → **Data sources**
2. Click **"Add data source"**
3. Select **"Prometheus"**
4. **URL:** `http://localhost:9090`
5. Click **"Save & Test"**
6. Should see: ✅ **"Data source is working"**

**✅ Grafana configured!**

---

### 4. Velociraptor Server Installation

#### Step 1: Download Velociraptor
```bash
cd /tmp
wget https://github.com/Velocidex/velociraptor/releases/download/v0.7.1/velociraptor-v0.7.1-linux-amd64
sudo mv velociraptor-v0.7.1-linux-amd64 /usr/local/bin/velociraptor
sudo chmod +x /usr/local/bin/velociraptor
```

#### Step 2: Generate Configuration
```bash
sudo velociraptor config generate -i
```

**Interactive prompts:**
- **OS:** Linux
- **Deployment type:** Self Signed SSL
- **Path for datastore:** `/opt/velociraptor/`
- **Public DNS name:** [VM1-IP] (e.g., 192.168.53.24)
- **Frontend port:** 8000
- **GUI port:** 8889
- **GUI bind address:** 0.0.0.0 (to allow network access)
- **Logs directory:** `/var/log/velociraptor/`
- **Create admin user:** Yes
- **Username:** admin
- **Password:** [Choose strong password]

**Configs saved to:**
- `/etc/velociraptor/server.config.yaml`
- `/etc/velociraptor/client.config.yaml`

#### Step 3: Create Velociraptor User
```bash
sudo useradd --no-create-home --shell /bin/false velociraptor
```

#### Step 4: Set Permissions
```bash
sudo mkdir -p /opt/velociraptor
sudo mkdir -p /var/log/velociraptor
sudo mkdir -p /etc/velociraptor
sudo chown -R velociraptor:velociraptor /opt/velociraptor
sudo chown -R velociraptor:velociraptor /var/log/velociraptor
sudo chown velociraptor:velociraptor /etc/velociraptor/*.yaml
```

#### Step 5: Create Systemd Service
```bash
sudo nano /etc/systemd/system/velociraptor.service
```

**Paste:**
```ini
[Unit]
Description=Velociraptor Server
After=network.target

[Service]
Type=simple
User=velociraptor
Group=velociraptor
ExecStart=/usr/local/bin/velociraptor --config /etc/velociraptor/server.config.yaml frontend -v
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

#### Step 6: Start Velociraptor
```bash
sudo systemctl daemon-reload
sudo systemctl start velociraptor
sudo systemctl enable velociraptor
sudo systemctl status velociraptor
```

#### Step 7: Access Velociraptor

Open browser: `https://[VM1-IP]:8889`
- Accept SSL warning
- Login with admin credentials

**✅ Velociraptor GUI should load!**

---

## VM #2: Production Server Setup

### System Preparation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install basic tools
sudo apt install -y curl wget vim net-tools
```

---

### 1. Wazuh Agent Installation

#### Step 1: Add Wazuh Repository
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt update
```

#### Step 2: Install Agent

**IMPORTANT:** Install version matching your manager!
```bash
# If manager is 4.9.2:
WAZUH_MANAGER="192.168.53.24" sudo apt install wazuh-agent=4.9.2-1 -y
```

**Replace `192.168.53.24` with your VM #1 IP!**

#### Step 3: Configure Manager IP

**If auto-configuration didn't work:**
```bash
sudo nano /var/ossec/etc/ossec.conf
```

**Find and change:**
```xml
<address>MANAGER_IP</address>
```

**To:**
```xml
<address>192.168.53.24</address>  <!-- Your VM #1 IP -->
```

#### Step 4: Start Agent
```bash
sudo systemctl start wazuh-agent
sudo systemctl enable wazuh-agent
sudo systemctl status wazuh-agent
```

#### Step 5: Verify Connection

**On VM #1:**
```bash
sudo /var/ossec/bin/agent_control -l
```

**Should show:**
```
ID: 001, Name: [hostname], IP: any, Active
```

**✅ Agent connected!**

---

### 2. Node Exporter Installation

#### Step 1: Download Node Exporter
```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
```

#### Step 2: Install Binary
```bash
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

#### Step 3: Create Systemd Service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

**Paste:**
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

#### Step 4: Start Node Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

#### Step 5: Verify
```bash
curl http://localhost:9100/metrics
```

**Should show metrics output.**

**✅ Node Exporter running!**

---

### 3. CrowdSec Installation

#### Step 1: Add CrowdSec Repository
```bash
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
```

#### Step 2: Install CrowdSec
```bash
sudo apt install crowdsec -y
```

#### Step 3: Start CrowdSec
```bash
sudo systemctl start crowdsec
sudo systemctl enable crowdsec
sudo systemctl status crowdsec
```

#### Step 4: Verify Installation
```bash
sudo cscli metrics
sudo cscli scenarios list
```

**✅ CrowdSec operational!**

---

### 4. Velociraptor Client Installation

#### Step 1: Download Client Binary

**On VM #2:**
```bash
cd /tmp
wget https://github.com/Velocidex/velociraptor/releases/download/v0.7.1/velociraptor-v0.7.1-linux-amd64
sudo mv velociraptor-v0.7.1-linux-amd64 /usr/local/bin/velociraptor-client
sudo chmod +x /usr/local/bin/velociraptor-client
```

#### Step 2: Copy Client Config from VM #1

**On VM #1:**
```bash
# Start temporary HTTP server
sudo cp /etc/velociraptor/client.config.yaml /tmp/velo-client.yaml
sudo chmod 644 /tmp/velo-client.yaml
cd /tmp
python3 -m http.server 8080
```

**On VM #2 (different terminal):**
```bash
wget http://192.168.53.24:8080/velo-client.yaml -O /tmp/client.config.yaml
sudo mkdir -p /etc/velociraptor
sudo mv /tmp/client.config.yaml /etc/velociraptor/
```

**Go back to VM #1 and press Ctrl+C to stop HTTP server**

#### Step 3: Create Velociraptor User
```bash
sudo useradd --no-create-home --shell /bin/false velociraptor
```

#### Step 4: Set Permissions
```bash
sudo chown velociraptor:velociraptor /etc/velociraptor/client.config.yaml
sudo touch /etc/velociraptor.writeback.yaml
sudo chown velociraptor:velociraptor /etc/velociraptor.writeback.yaml
```

#### Step 5: Create Systemd Service
```bash
sudo nano /etc/systemd/system/velociraptor-client.service
```

**Paste:**
```ini
[Unit]
Description=Velociraptor Client
After=network.target

[Service]
Type=simple
User=velociraptor
Group=velociraptor
ExecStart=/usr/local/bin/velociraptor-client --config /etc/velociraptor/client.config.yaml client -v
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

#### Step 6: Start Client
```bash
sudo systemctl daemon-reload
sudo systemctl start velociraptor-client
sudo systemctl enable velociraptor-client
sudo systemctl status velociraptor-client
```

#### Step 7: Verify in Velociraptor GUI

**On VM #1, open Velociraptor GUI:**
- `https://192.168.53.24:8889`
- Should see VM #2 hostname in client list!

**✅ Client connected!**

---

## Configuration

### Prometheus Target Configuration

**Already configured during installation, but to verify:**

**On VM #1:**
```bash
cat /etc/prometheus/prometheus.yml
```

**Should show:**
```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'production-server'
    static_configs:
      - targets: ['192.168.53.92:9100']
```

**Test configuration:**
```bash
sudo promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

---

### Grafana Dashboard Import

**Import Node Exporter Dashboard:**

1. Open Grafana: `http://192.168.53.24:3000`
2. ☰ menu → **Dashboards** → **New** → **Import**
3. Enter dashboard ID: **1860**
4. Click **"Load"**
5. Select **Prometheus** data source
6. Click **"Import"**

**✅ Dashboard ready!**

---

## Verification

### Verify All Services - VM #1
```bash
echo "=== Wazuh Manager ==="
sudo systemctl status wazuh-manager --no-pager | grep Active

echo "=== Wazuh Indexer ==="
sudo systemctl status wazuh-indexer --no-pager | grep Active

echo "=== Wazuh Dashboard ==="
sudo systemctl status wazuh-dashboard --no-pager | grep Active

echo "=== Prometheus ==="
sudo systemctl status prometheus --no-pager | grep Active

echo "=== Grafana ==="
sudo systemctl status grafana-server --no-pager | grep Active

echo "=== Velociraptor ==="
sudo systemctl status velociraptor --no-pager | grep Active
```

**All should show: `active (running)`**

---

### Verify All Agents - VM #2
```bash
echo "=== Wazuh Agent ==="
sudo systemctl status wazuh-agent --no-pager | grep Active

echo "=== Node Exporter ==="
sudo systemctl status node_exporter --no-pager | grep Active

echo "=== CrowdSec ==="
sudo systemctl status crowdsec --no-pager | grep Active

echo "=== Velociraptor Client ==="
sudo systemctl status velociraptor-client --no-pager | grep Active
```

**All should show: `active (running)`**

---

### Test Connectivity

**Check Prometheus Targets:**
```
http://192.168.53.24:9090/targets
```
Both targets should show **UP**

**Check Wazuh Agent:**
```bash
# On VM #1
sudo /var/ossec/bin/agent_control -l
```
Should list VM #2 as **Active**

**Check Velociraptor Client:**
- Open: `https://192.168.53.24:8889`
- Should see VM #2 hostname with green status

---

## Startup Scripts

### VM #1 - Start SOC Services

**Create script:**
```bash
nano ~/start-soc.sh
```

**Paste:**
```bash
#!/bin/bash
echo "=========================================="
echo "Starting SOC Server Services..."
echo "=========================================="

echo "Starting Wazuh Manager..."
sudo systemctl start wazuh-manager
sleep 5

echo "Starting Wazuh Indexer..."
sudo systemctl start wazuh-indexer
sleep 3

echo "Starting Wazuh Dashboard..."
sudo systemctl start wazuh-dashboard
sleep 3

echo "Starting Prometheus..."
sudo systemctl start prometheus
sleep 2

echo "Starting Grafana..."
sudo systemctl start grafana-server
sleep 2

echo "Starting Velociraptor..."
sudo systemctl start velociraptor
sleep 2

echo ""
echo "=========================================="
echo "All services started!"
echo "=========================================="
```

**Make executable:**
```bash
chmod +x ~/start-soc.sh
```

---

### VM #1 - Check Status

**Create script:**
```bash
nano ~/check-soc-status.sh
```

**Paste:**
```bash
#!/bin/bash
echo "=========================================="
echo "SOC SERVER STATUS CHECK"
echo "=========================================="

echo ""
echo "=== Wazuh Manager ==="
sudo systemctl status wazuh-manager --no-pager | grep "Active:"

echo ""
echo "=== Wazuh Indexer ==="
sudo systemctl status wazuh-indexer --no-pager | grep "Active:"

echo ""
echo "=== Wazuh Dashboard ==="
sudo systemctl status wazuh-dashboard --no-pager | grep "Active:"

echo ""
echo "=== Prometheus ==="
sudo systemctl status prometheus --no-pager | grep "Active:"

echo ""
echo "=== Grafana ==="
sudo systemctl status grafana-server --no-pager | grep "Active:"

echo ""
echo "=== Velociraptor Server ==="
sudo systemctl status velociraptor --no-pager | grep "Active:"

echo ""
echo "=========================================="
echo "=== Registered Wazuh Agents ==="
sudo /var/ossec/bin/agent_control -l

echo ""
echo "=========================================="
echo "=== Access URLs ==="
echo "Wazuh Dashboard: https://192.168.53.24"
echo "Prometheus: http://192.168.53.24:9090"
echo "Grafana: http://192.168.53.24:3000"
echo "Velociraptor: https://192.168.53.24:8889"
echo "=========================================="
```

**Make executable:**
```bash
chmod +x ~/check-soc-status.sh
```

---

### VM #2 - Start Agents

**Create script:**
```bash
nano ~/start-agents.sh
```

**Paste:**
```bash
#!/bin/bash
echo "=========================================="
echo "Starting Production Server Agents..."
echo "=========================================="

echo "Starting Wazuh Agent..."
sudo systemctl start wazuh-agent
sleep 3

echo "Starting Node Exporter..."
sudo systemctl start node_exporter
sleep 2

echo "Starting CrowdSec..."
sudo systemctl start crowdsec
sleep 2

echo "Starting Velociraptor Client..."
sudo systemctl start velociraptor-client
sleep 2

echo ""
echo "=========================================="
echo "All agents started!"
echo "=========================================="
```

**Make executable:**
```bash
chmod +x ~/start-agents.sh
```

---

### VM #2 - Check Status

**Create script:**
```bash
nano ~/check-agents-status.sh
```

**Paste:**
```bash
#!/bin/bash
echo "=========================================="
echo "PRODUCTION SERVER STATUS CHECK"
echo "=========================================="

echo ""
echo "=== Wazuh Agent ==="
sudo systemctl status wazuh-agent --no-pager | grep "Active:"

echo ""
echo "=== Node Exporter ==="
sudo systemctl status node_exporter --no-pager | grep "Active:"

echo ""
echo "=== CrowdSec ==="
sudo systemctl status crowdsec --no-pager | grep "Active:"

echo ""
echo "=== Velociraptor Client ==="
sudo systemctl status velociraptor-client --no-pager | grep "Active:"

echo ""
echo "=========================================="
echo "=== CrowdSec Metrics ==="
sudo cscli metrics

echo ""
echo "=========================================="
```

**Make executable:**
```bash
chmod +x ~/check-agents-status.sh
```

---

## Troubleshooting

### Common Issues

#### Issue 1: Wazuh Agent Not Connecting

**Symptoms:**
- Agent status shows "Never connected"
- `agent_control -l` doesn't list agent

**Solutions:**
```bash
# On VM #2, check agent config
sudo grep -A 5 "<server>" /var/ossec/etc/ossec.conf

# Should show correct manager IP
# If not, fix it:
sudo nano /var/ossec/etc/ossec.conf
# Change <address>MANAGER_IP</address> to <address>192.168.53.24</address>

# Restart agent
sudo systemctl restart wazuh-agent
```

---

#### Issue 2: Prometheus Target Down

**Symptoms:**
- Target shows "DOWN" in Prometheus
- Error: "connection refused" or "no route to host"

**Solutions:**
```bash
# On VM #2, verify Node Exporter is running
sudo systemctl status node_exporter

# Check if port is listening
sudo netstat -tlnp | grep 9100

# Test from VM #1
curl http://192.168.53.92:9100/metrics
```

---

#### Issue 3: Disk Space Full on VM #1

**Symptoms:**
- Services failing to start
- "No space left on device" errors

**Solutions:**
```bash
# Check disk usage
df -h

# Clean Wazuh queue (can grow large)
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater/*
sudo systemctl start wazuh-manager

# Clean journal logs
sudo journalctl --vacuum-size=100M
```

---

#### Issue 4: Velociraptor Client Can't Write

**Symptoms:**
- Client service fails with "permission denied"

**Solutions:**
```bash
# Fix permissions
sudo chown velociraptor:velociraptor /etc/velociraptor/client.config.yaml
sudo touch /etc/velociraptor.writeback.yaml
sudo chown velociraptor:velociraptor /etc/velociraptor.writeback.yaml

# Restart client
sudo systemctl restart velociraptor-client
```

---

## 🎉 Installation Complete!

**Your Mini SOC/XDR environment is now operational!**

### Next Steps:

1. **[View Demo Guide](DEMOS.md)** - Learn how to demonstrate each tool
2. **[Read Command Reference](COMMANDS.md)** - Quick command cheat sheet
3. **Practice attacks** - Run the SSH brute force scenario
4. **Take screenshots** - Document your dashboards
5. **Present to team** - Show off your work!

---

## 📚 Additional Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Tutorials](https://grafana.com/tutorials/)
- [Velociraptor Docs](https://docs.velociraptor.app/)
- [CrowdSec Hub](https://hub.crowdsec.net/)

---

**Built with 💙 for Cybersecurity Malaysia Internship**
