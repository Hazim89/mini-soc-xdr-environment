# 🎬 Live Demonstration Guide

> Step-by-step instructions for demonstrating all SOC/XDR capabilities with detailed UI navigation

---

## 📋 Table of Contents

- [Demo Prerequisites](#demo-prerequisites)
- [Demo 1: Wazuh SIEM](#demo-1-wazuh-siem)
  - [Show Agent Connection](#demo-1a-show-agent-connection)
  - [Generate Security Alert](#demo-1b-generate-security-alert)
  - [File Integrity Monitoring](#demo-1c-file-integrity-monitoring)
  - [View Agent Details](#demo-1d-view-agent-details)
- [Demo 2: Prometheus + Grafana](#demo-2-prometheus--grafana)
  - [Verify Targets](#demo-2a-verify-prometheus-targets)
  - [Query System Metrics](#demo-2b-query-system-metrics)
  - [Generate Load](#demo-2c-generate-cpu-load)
  - [Grafana Dashboard](#demo-2d-grafana-dashboard)
- [Demo 3: Velociraptor DFIR](#demo-3-velociraptor-dfir)
  - [View Connected Clients](#demo-3a-view-connected-clients)
  - [Collect System Information](#demo-3b-collect-system-information)
  - [View Process Tree](#demo-3c-view-process-tree)
  - [Collect Authentication Logs](#demo-3d-collect-authentication-logs)
- [Demo 4: CrowdSec IPS](#demo-4-crowdsec-ips)
  - [Check Status](#demo-4a-check-crowdsec-status)
  - [View Metrics](#demo-4b-view-metrics)
  - [List Banned IPs](#demo-4c-list-banned-ips)
  - [View Detection Scenarios](#demo-4d-view-detection-scenarios)
- [Complete Attack Scenario](#complete-attack-scenario---all-tools-together)
- [Troubleshooting Demos](#troubleshooting-demos)

---

## Demo Prerequisites

### Before Starting Demos

**Ensure both VMs are running:**

**VM #1 (SOC Server):**
```bash
# Start all services
~/start-soc.sh

# Verify status
~/check-soc-status.sh
```

**VM #2 (Production Server):**
```bash
# Start all agents
~/start-agents.sh

# Verify status
~/check-agents-status.sh
```

---

### Open All Dashboards in Browser

**Open these URLs in separate tabs:**

1. **Wazuh:** `https://192.168.53.24` (replace with your IP)
2. **Prometheus:** `http://192.168.53.24:9090`
3. **Grafana:** `http://192.168.53.24:3000`
4. **Velociraptor:** `https://192.168.53.24:8889`

**Log in to each dashboard before starting demos!**

---

### Have VM #2 Terminal Ready

Keep a terminal window open on VM #2 for running attack commands.

---

## Demo 1: Wazuh SIEM

**Purpose:** Demonstrate security event detection and centralized logging

**Time:** 5-7 minutes

**Access URL:** `https://192.168.53.24`

---

### Demo 1A: Show Agent Connection

**Objective:** Show that production server is being monitored

#### Step 1: Access Wazuh Dashboard

1. **Open browser** and go to: `https://192.168.53.24`
2. **Accept SSL warning:**
   - Click **"Advanced"**
   - Click **"Proceed to 192.168.53.24 (unsafe)"**
3. **Log in:**
   - Username: `admin`
   - Password: `yT2qI0O7smhOyf6r1TU+2rOkj734M26` (your password)
4. **Click "Login"**

#### Step 2: Navigate to Endpoints

**You'll see the Wazuh home page. Now navigate:**

1. **Look at the LEFT sidebar** (vertical menu)
2. **Click the ☰ (hamburger) icon** at the very top left
3. **A menu opens** - Look for **"Server management"**
4. **Hover over or click "Server management"**
5. **Click "Endpoints Summary"**

**Alternative navigation:**
- Some Wazuh versions: **Menu → Agents → Overview**
- Or: **Menu → Endpoints Summary** (directly)

#### Step 3: What to Show

**On the Endpoints Summary page, you'll see:**

✅ **Total agents:** 1  
✅ **Active agents:** 1  
✅ **Agent name:** hazimprod (or your VM #2 hostname)  
✅ **Status:** Active (with green indicator)  
✅ **Agent ID:** 001  
✅ **IP Address:** 192.168.53.92  
✅ **Last keep alive:** Recent timestamp (seconds/minutes ago)  

**Say to your senior:**  
*"This dashboard shows our production server (VM #2) is connected and being monitored in real-time. The green status indicates active communication between the agent and manager."*

---

### Demo 1B: Generate Security Alert

**Objective:** Show real-time threat detection

#### Step 1: Generate Failed SSH Attempts

**On VM #2, open terminal and run:**
```bash
# Simulate SSH brute force attack
ssh fakeuser1@localhost
# Type wrong password and press Enter

ssh fakeuser2@localhost  
# Type wrong password and press Enter

ssh fakeuser3@localhost
# Type wrong password and press Enter

ssh fakeuser4@localhost
# Type wrong password and press Enter

ssh fakeuser5@localhost
# Type wrong password and press Enter
```

**Wait 10-15 seconds for logs to be processed.**

#### Step 2: Navigate to Security Events

**In Wazuh Dashboard:**

1. **Click the ☰ menu** (top left)
2. **Look for "Threat Hunting"** section
3. **Click "Threat Hunting"**
4. **Then click "Events"**

**Alternative:** 
- **Menu → Security Events**
- Or **Menu → Modules → Security Events**

#### Step 3: Filter Events

**On the Events page:**

1. **Look at the top right** - you'll see a **time range selector**
2. **Click the time dropdown**
3. **Select "Last 15 minutes"**

4. **Look for the search bar** (usually near the top)
5. **Click in the search bar**
6. **Type or paste:**
```
   agent.name: "hazimprod" AND authentication
```
7. **Press Enter**

#### Step 4: What to Show

**You should now see multiple events:**

✅ **Event descriptions:** "sshd: authentication failed" or similar  
✅ **Rule ID:** 5503, 5710, or similar  
✅ **User attempted:** fakeuser1, fakeuser2, fakeuser3, etc.  
✅ **Source IP:** 127.0.0.1  
✅ **Timestamps:** When each attempt occurred  
✅ **Alert level/severity:** Usually Level 5 (Medium)  

**Click on one event to expand it:**
- Shows full event details
- JSON structure
- Decoder information
- Rule description

**Say to your senior:**  
*"Our SIEM immediately detected the failed login attempts and generated security alerts. In a real SOC, this would trigger an investigation workflow and possibly automated response actions."*

---

### Demo 1C: File Integrity Monitoring

**Objective:** Show detection of file modifications

#### Step 1: Modify a Monitored File

**On VM #2:**
```bash
# Modify a system file that Wazuh monitors
echo "test change for demo" | sudo tee -a /etc/passwd
```

#### Step 2: Wait for FIM Scan

**Wazuh scans files periodically. Wait 2-3 minutes.**

#### Step 3: Search for FIM Events

**In Wazuh Dashboard:**

1. Go to **Menu → Threat Hunting → Events**
2. **Time filter:** "Last 15 minutes"
3. **Search bar:**
```
   syscheck
```
   OR
```
   rule.id:550
```

#### Step 4: What to Show

✅ **File modification detected:** /etc/passwd  
✅ **Change type:** Modified  
✅ **What changed:** Content added  
✅ **Hash values:** MD5/SHA1 before and after  
✅ **Timestamp:** When change occurred  

**Say to your senior:**  
*"Wazuh continuously monitors critical system files. Any unauthorized modification triggers an alert, which is essential for detecting backdoors or malware."*

---

### Demo 1D: View Agent Details

**Objective:** Show comprehensive agent information

#### Step 1: Navigate to Agent Page

1. **Menu → Server Management → Endpoints Summary**
2. **Click on "hazimprod"** (the agent name itself)

#### Step 2: What to Show

**On the agent details page, you'll see tabs:**

**Overview Tab:**
✅ **Operating System:** Ubuntu 24.04  
✅ **Agent version:** 4.9.2  
✅ **IP address:** 192.168.53.92  
✅ **Registration date:** When agent was first registered  
✅ **Last keep alive:** Connection status  
✅ **Uptime:** How long agent has been running  

**Additional tabs:**
- **Configuration:** Agent configuration details
- **Inventory:** Installed packages, processes, network info
- **Stats:** Agent statistics

**Say to your senior:**  
*"This provides complete visibility into the monitored endpoint - OS details, installed software, running processes, and network connections - all from a central dashboard."*

---

## Demo 2: Prometheus + Grafana

**Purpose:** Demonstrate performance monitoring and visualization

**Time:** 5-7 minutes

---

### Demo 2A: Verify Prometheus Targets

**Objective:** Show Prometheus is scraping metrics from production server

**Access URL:** `http://192.168.53.24:9090`

#### Step 1: Navigate to Targets Page

**Option 1: Using Menu**

1. **Open Prometheus** in browser
2. **Look at the TOP menu bar** - you'll see:
   - Alerts
   - Graph
   - **Status** ← Look for this
   - Help
3. **Click "Status"**
4. **A dropdown appears** - Click **"Targets"**

**Option 2: Direct URL**

Just go directly to:
```
http://192.168.53.24:9090/targets
```

#### Step 2: What to Show

**On the Targets page, you'll see a table:**

| Endpoint | State | Labels | Last Scrape | Scrape Duration |
|----------|-------|--------|-------------|-----------------|
| http://localhost:9090/metrics | **UP** ✅ | job="prometheus" | 2s ago | 15ms |
| http://192.168.53.92:9100/metrics | **UP** ✅ | job="production-server" | 3s ago | 20ms |

**Both should show green "UP" status!**

**Say to your senior:**  
*"Prometheus is successfully scraping metrics from both itself and our production server every 15 seconds. The green 'UP' status confirms connectivity and data collection."*

---

### Demo 2B: Query System Metrics

**Objective:** Show real-time metric queries using PromQL

**Access URL:** `http://192.168.53.24:9090`

#### Step 1: Navigate to Query Interface

1. **Open Prometheus:** `http://192.168.53.24:9090`
2. **You'll land on the main page**
3. **You'll see a large text box** (query editor)
4. **Below it are two tabs:** "Table" and "Graph"

#### Query 1: CPU Usage

**Step by step:**

1. **Click in the query box** (the large text area)
2. **Paste this query:**
```
   100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle",job="production-server"}[5m])) * 100)
```
3. **Click the blue "Execute" button** (next to the query box)
4. **Click the "Graph" tab** (below the query box)
5. **You'll see a line graph** showing CPU usage over time

**What to show:**
- ✅ Current CPU usage percentage
- ✅ Historical trend (last hour by default)
- ✅ Hover over graph to see exact values

**Say to your senior:**  
*"This PromQL query calculates real-time CPU usage. We can see the current percentage and historical trends."*

---

#### Query 2: Memory Usage

**Step by step:**

1. **Clear the previous query** (select all, delete)
2. **Paste this query:**
```
   100 * (1 - ((node_memory_MemAvailable_bytes{job="production-server"}) / (node_memory_MemTotal_bytes{job="production-server"})))
```
3. **Click "Execute"**
4. **Click "Graph" tab**

**What to show:**
- ✅ Current memory usage percentage
- ✅ Memory consumption trend

**Say to your senior:**  
*"Here we're tracking memory utilization. This helps identify memory leaks or resource exhaustion."*

---

#### Query 3: Disk Usage

1. **Clear previous query**
2. **Paste:**
```
   100 - ((node_filesystem_avail_bytes{job="production-server",mountpoint="/"} * 100) / node_filesystem_size_bytes{job="production-server",mountpoint="/"})
```
3. **Click "Execute" → "Graph" tab**

**What to show:**
- ✅ Disk space usage percentage
- ✅ Available vs used space

---

#### Query 4: System CPU (for Attack Demo)

**Use this during the attack scenario:**
```
rate(node_cpu_seconds_total{job="production-server",mode="system"}[2m])
```

**Shows CPU time in system mode (spikes during authentication attempts)**

---

#### Query 5: Network Connections
```
node_netstat_Tcp_CurrEstab{job="production-server"}
```

**Shows number of established TCP connections**

---

### Demo 2C: Generate CPU Load

**Objective:** Show real-time detection of performance changes

#### Step 1: Keep Prometheus Open

**Have the CPU usage query ready:**
```
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle",job="production-server"}[5m])) * 100)
```

#### Step 2: Generate Load on VM #2

**On VM #2, run:**
```bash
# Install stress tool if not already installed
sudo apt install stress-ng -y

# Generate CPU load for 30 seconds
stress-ng --cpu 2 --timeout 30s
```

#### Step 3: Watch Prometheus Graph

**In Prometheus:**
1. **Click "Execute" button repeatedly** (every 5-10 seconds)
2. **Watch the graph update**
3. **You should see CPU spike!**

**Say to your senior:**  
*"Watch as I generate CPU load on the production server - Prometheus detects the change in real-time. This is how we monitor performance during attacks or incidents."*

---

### Demo 2D: Grafana Dashboard

**Objective:** Show professional visualization and operational dashboards

**Access URL:** `http://192.168.53.24:3000`

#### Step 1: Log In to Grafana

1. **Open:** `http://192.168.53.24:3000`
2. **Login:**
   - Username: `admin`
   - Password: `727599` (your password)
3. **Click "Log in"**

#### Step 2: Navigate to Dashboards

**Look at the LEFT sidebar** - you'll see icons:

1. **Click the ☰ (hamburger menu) icon** at the top left
   
   OR
   
2. **Look for the "Dashboards" icon** (looks like four squares 📊)

**Then:**
1. **Click "Dashboards"**
2. **You'll see a list of dashboards**
3. **Look for "Node Exporter Full"**
4. **Click on it**

**If you don't see "Node Exporter Full":**

**You need to import it first:**

1. **Click ☰ menu**
2. **Click "Dashboards"**
3. **Click "New" button** (top right) → **"Import"**
4. **In "Import via grafana.com" box, type:** `1860`
5. **Click "Load"**
6. **Select "Prometheus" from dropdown** (appears after loading)
7. **Click "Import"**

#### Step 3: What to Show

**The Node Exporter Full dashboard has multiple sections. Scroll through and show:**

**Section 1: System Overview**
- ✅ **System uptime**
- ✅ **CPU cores**
- ✅ **Total RAM**
- ✅ **Root filesystem size**

**Section 2: CPU Usage**
- ✅ **CPU Basic** - Overall CPU usage graph
- ✅ **CPU Cores** - Individual core usage
- ✅ **System Load** - 1min, 5min, 15min averages

**Section 3: Memory Usage**
- ✅ **Memory Basic** - RAM utilization
- ✅ **Memory Stack** - Breakdown by type
- ✅ **Swap usage** - Swap file usage

**Section 4: Network Traffic**
- ✅ **Network Traffic Basic** - Upload/download
- ✅ **Network Traffic** - Detailed per interface
- ✅ **Network Traffic Errors** - Packet errors

**Section 5: Disk I/O**
- ✅ **Disk R/W Data** - Read/write operations
- ✅ **Disk IOps** - I/O operations per second
- ✅ **Disk Space Used Basic** - Storage usage

**All graphs are live and update automatically!**

**Say to your senior:**  
*"This is what professional SOC teams use for infrastructure monitoring. All metrics from our production server are visualized in real-time. We can set alerts on any metric and create custom dashboards for different teams."*

#### Step 4: Demonstrate Live Updates

**If you ran the stress-ng command earlier:**
- **Point to the CPU graphs**
- **Show the spike that occurred**
- **Demonstrate how historical data is preserved**

---

## Demo 3: Velociraptor DFIR

**Purpose:** Demonstrate digital forensics and incident response capabilities

**Time:** 4-5 minutes

**Access URL:** `https://192.168.53.24:8889`

---

### Demo 3A: View Connected Clients

**Objective:** Show remote forensic capabilities

#### Step 1: Log In to Velociraptor

1. **Open:** `https://192.168.53.24:8889`
2. **Accept SSL warning:**
   - Click "Advanced"
   - Click "Proceed to 192.168.53.24"
3. **Log in** with admin credentials
4. **You'll land on the HOME page**

#### Step 2: Find Client List

**After logging in, you should immediately see:**

- **Client list in the center of the screen**
- **"hazimprod"** listed with a **green dot** (online)

**If you don't see clients immediately:**

1. **Look at the TOP menu bar**
2. **Click "Show All"** or **"Clients"** (computer icon 🖥️)
3. **Client list appears**

#### Step 3: What to Show

**Point out:**

✅ **Client name:** hazimprod  
✅ **Status:** Online (green dot ●)  
✅ **Operating System:** Linux  
✅ **Last seen:** Few seconds ago  
✅ **Client ID:** C.xxxxxxxxxxxxx  

**Say to your senior:**  
*"Velociraptor gives us the ability to remotely collect forensic evidence from any endpoint without disrupting operations. This is critical during incident response."*

---

### Demo 3B: Collect System Information

**Objective:** Show remote data collection

#### Step 1: Open Client Details

1. **Click on "hazimprod"** (the client name)
2. **Client details page opens**
3. **You'll see tabs at the top:**
   - Overview
   - VQL Drilldown
   - **Collected Artifacts** ← Important
   - Server Events
   - Shell

#### Step 2: Start New Collection

**Look for the "New Collection" button:**
- Usually **top right** of the page
- Or **"+" icon**
- Or in the **"Collected Artifacts" tab**

**Click "New Collection"**

#### Step 3: Search for Artifact

**A search interface appears:**

1. **Click in the search box**
2. **Type:** `Generic.Client.Info`
3. **You'll see a list of matching artifacts**
4. **Click on "Generic.Client.Info"** from the list

#### Step 4: Launch Collection

**The artifact details page opens:**

1. **Scroll down to the bottom** of the page
2. **Look for the "Launch" button** (usually bottom right)
3. **Click "Launch"**

**A progress indicator appears - wait 10-20 seconds**

#### Step 5: View Results

**After collection completes:**

1. **Click "View Results"** or **"Notebook" tab**
2. **You'll see collected data**

#### Step 6: What to Show

**System information collected:**

✅ **Hostname:** hazimprod  
✅ **Operating System:** Ubuntu 24.04 LTS  
✅ **Kernel version:** 6.x.x  
✅ **Architecture:** x86_64  
✅ **Total memory:** 1.4GB  
✅ **CPU information:** Processor details  
✅ **System uptime:** How long running  
✅ **Network interfaces:** IPs, MACs  
✅ **Boot time:** When system started  

**Say to your senior:**  
*"In seconds, we collected complete system intelligence remotely. During an incident, we can gather evidence across hundreds of machines simultaneously without alerting attackers."*

---

### Demo 3C: View Process Tree

**Objective:** Show process monitoring

#### Step 1: Start New Collection

1. **Click on "hazimprod" client**
2. **Click "New Collection"**
3. **Search:** `Generic.System.Pstree`
4. **Click on it**
5. **Scroll down and click "Launch"**

#### Step 2: View Results

**After collection (10-20 seconds):**
- **Click "View Results"**

#### Step 3: What to Show

✅ **Complete process tree** - Parent-child relationships  
✅ **Process IDs (PIDs)**  
✅ **Command lines** - What each process is running  
✅ **User ownership** - Which user runs each process  

**Say to your senior:**  
*"In a real investigation, we'd look for suspicious processes, unauthorized services, or malware in the process tree."*

---

### Demo 3D: Collect Authentication Logs

**Objective:** Show log collection for forensics

#### Step 1: Start New Collection

1. **New Collection** on hazimprod
2. **Search:** `Linux.Collection.AuthLogs`
3. **Click on it**
4. **Click "Launch"**

#### Step 2: View Results

**After collection:**
- **Click "View Results"** or **"Notebook"**

#### Step 3: What to Show

✅ **Complete /var/log/auth.log** contents  
✅ **All SSH login attempts**  
✅ **sudo command history**  
✅ **Authentication events**  
✅ **Failed login attempts** (from our earlier demo!)  

**Say to your senior:**  
*"We can collect any log file, registry key, or forensic artifact needed for investigation. This data is collected securely and can be analyzed offline."*

---

### Demo 3E: View Login History (Bonus)

**Quick demo:**

1. **New Collection**
2. **Search:** `Linux.Sys.LastUserLogin`
3. **Launch**
4. **Show:** Recent user login history with timestamps

---

## Demo 4: CrowdSec IPS

**Purpose:** Demonstrate local intrusion prevention and community intelligence

**Time:** 2-3 minutes

**Access:** Command-line on VM #2

---

### Demo 4A: Check CrowdSec Status

**Objective:** Show CrowdSec is running

**On VM #2:**
```bash
# Check service status
sudo systemctl status crowdsec --no-pager
```

**What to show:**
✅ **Status:** active (running)  
✅ **Memory usage:** ~90MB  
✅ **Uptime:** How long running  

**Say to your senior:**  
*"CrowdSec is our local intrusion prevention system. It monitors logs in real-time and can automatically block attackers."*

---

### Demo 4B: View Metrics

**Objective:** Show community protection and monitoring activity

**On VM #2:**
```bash
# Show detailed metrics
sudo cscli metrics
```

#### What to Show

**Table 1: Acquisition Metrics**

Shows which log files are being monitored:
- ✅ `/var/log/auth.log` - SSH/authentication attempts
- ✅ `/var/log/syslog` - System messages  
- ✅ `/var/log/kern.log` - Kernel messages
- ✅ Lines read, parsed, and analyzed

**Table 2: Local API Decisions** (if populated)

Shows banned IPs and reasons:
- ✅ `ssh:bruteforce` - X IPs banned for SSH brute force
- ✅ `ssh:exploit` - X IPs banned for exploit attempts
- ✅ `generic:scan` - X IPs banned for port scanning
- ✅ **Total IPs blocked** (could be thousands from community!)

**Table 3: Parser Metrics**

Shows log parsing activity:
- ✅ Which parsers are active
- ✅ How many log lines processed
- ✅ SSH logs analyzed

**Say to your senior:**  
*"CrowdSec continuously monitors our logs. The 'Local API Decisions' shows IPs that have been blocked - either from local detection or from community intelligence. We're protected by crowd-sourced threat data from thousands of users worldwide."*

---

### Demo 4C: List Banned IPs

**On VM #2:**
```bash
# Show currently banned IPs
sudo cscli decisions list
```

**What might show:**
- ✅ List of IP addresses
- ✅ Ban reason (e.g., "crowdsecurity/ssh-bf")
- ✅ Duration of ban
- ✅ Origin (local or community)

**If list is empty:**  
*"Currently no active local bans, but CrowdSec is monitoring and ready to block attacks automatically."*

---

### Demo 4D: View Detection Scenarios

**On VM #2:**
```bash
# Show installed detection rules
sudo cscli scenarios list | head -20
```

**What to show:**

✅ **Detection scenarios installed:**
- `crowdsecurity/ssh-bf` - SSH brute force detection
- `crowdsecurity/ssh-slow-bf` - Slow SSH brute force
- `crowdsecurity/ssh-cve-2024-6387` - SSH vulnerability
- `crowdsecurity/ssh-exploit` - SSH exploit detection
- Many more...

**Say to your senior:**  
*"These are community-maintained detection rules. When attacks match these patterns, CrowdSec automatically bans the attacker's IP at the firewall level."*

**Optional - Show collections:**
```bash
# Show installed collections
sudo cscli collections list
```

---

## Complete Attack Scenario - All Tools Together

**Purpose:** Demonstrate defense-in-depth - all 5 tools detecting the same attack

**Time:** 7-10 minutes

**This is your BEST demo - shows everything working together!**

---

### Pre-Demo Setup

**Have these ready:**

**Browser tabs open:**
1. Wazuh - Events page (agent.name: "hazimprod" AND authentication)
2. Prometheus - Graph page (query box ready)
3. Grafana - Node Exporter dashboard
4. Velociraptor - hazimprod client page

**Terminals:**
- VM #2 - Ready to run attack
- VM #2 - Ready to check CrowdSec

---

### Step 1: Launch the Attack (30 seconds)

**Say to your senior:**  
*"I'm going to simulate an SSH brute force attack - an attacker trying to break into our production server with multiple failed login attempts. Watch how all five security tools detect and respond."*

**On VM #2, run:**
```bash
# Simulate SSH brute force attack
for i in {1..15}; do 
  echo "Attack attempt $i..."
  ssh attacker$i@localhost
  sleep 1
done
```

**Press Ctrl+C after 10-15 attempts**

---

### Step 2: Layer 1 - Wazuh Detection (1 minute)

**Say:** *"First, let's check our SIEM..."*

**Switch to Wazuh tab:**

1. **Refresh** the Events page
2. **Search bar:** `agent.name: "hazimprod" AND authentication`
3. **Press Enter**

**Show:**
- ✅ **Multiple authentication failure events**
- ✅ **Rule triggered:** sshd authentication failed
- ✅ **Users attempted:** attacker1, attacker2, attacker3...
- ✅ **Timestamps:** When each attempt occurred
- ✅ **Alert severity:** Level 5

**Click on one event to expand:**
- Full event details
- Source IP
- Decoder information

**Say to your senior:**  
*"Layer 1 - SIEM Detection: Wazuh immediately detected all failed login attempts and generated security alerts. In a real SOC, this would trigger an investigation workflow."*

---

### Step 3: Layer 2 - Prometheus Monitoring (1 minute)

**Say:** *"Now let's see the performance impact..."*

**Switch to Prometheus tab:**

1. **In query box, paste:**
```
   rate(node_cpu_seconds_total{job="production-server",mode="system"}[2m])
```
2. **Click "Execute"**
3. **Click "Graph" tab**

**Show:**
- ✅ **CPU spike** during authentication attempts
- ✅ **System resource consumption**

**Try another query:**
```
node_netstat_Tcp_CurrEstab{job="production-server"}
```

**Show:**
- ✅ **Network connection changes**

**Say to your senior:**  
*"Layer 2 - Performance Monitoring: Prometheus captured the attack's impact on system resources. We can see CPU spikes and network activity changes."*

---

### Step 4: Layer 3 - Grafana Visualization (1 minute)

**Say:** *"Let's visualize this in Grafana..."*

**Switch to Grafana tab (Node Exporter dashboard):**

**Point out graphs:**
- ✅ **CPU Usage** - Look for small spike
- ✅ **System Load** - May show increase
- ✅ **Network Traffic** - Slight increase during attack
- ✅ **Memory** - Stable (shows system handled attack)

**Say to your senior:**  
*"Layer 3 - Visualization: Grafana gives us instant visual insight. We can see the attack's impact at a glance. Notice the system remained stable - no resource exhaustion."*

---

### Step 5: Layer 4 - Velociraptor Forensics (2 minutes)

**Say:** *"Now let's collect forensic evidence..."*

**Switch to Velociraptor tab:**

1. **Click on "hazimprod" client**
2. **Click "New Collection"**
3. **Search:** `Linux.Collection.AuthLogs`
4. **Click on it**
5. **Scroll down, click "Launch"**
6. **Wait 10-20 seconds**
7. **Click "View Results"**

**Show:**
- ✅ **Complete /var/log/auth.log** collected remotely
- ✅ **All failed SSH attempts** visible in logs
- ✅ **Evidence collected** for investigation

**Optional - Collect more:**
- `Linux.Sys.LastUserLogin` - Login history
- `Linux.Network.Netstat` - Network connections during attack

**Say to your senior:**  
*"Layer 4 - Incident Response: Velociraptor allows us to remotely collect forensic evidence without disrupting the production server. In a real incident, we'd collect this across all potentially compromised systems."*

---

### Step 6: Layer 5 - CrowdSec Protection (1 minute)

**Say:** *"Finally, our local protection layer..."*

**On VM #2 terminal:**
```bash
# Check if attack was detected
sudo cscli alerts list | head -10

# Check current metrics
sudo cscli metrics
```

**Show:**

**If alerts show:**
- ✅ **SSH brute force detected**
- ✅ **Decision made** (IP banned or would be in production)

**If no local alerts:**
- ✅ **Show Parser Metrics** - auth.log is being monitored
- ✅ **Show detection scenarios:** `sudo cscli scenarios list | grep ssh`

**Say to your senior:**  
*"Layer 5 - Local Prevention: CrowdSec is monitoring and has detection rules loaded. In a production environment with external attacks, it would automatically ban the attacker's IP at the firewall level, blocking them before they can cause damage."*

---

### Summary (1 minute)

**Say to your senior:**  

*"What we just demonstrated is a complete Security Operations Center response to a cyber attack. Let me summarize:"*

**Show this mental table:**

| Layer | Tool | Function | Result |
|-------|------|----------|--------|
| **Detection** | Wazuh | Security event analysis | ✅ Alerts generated |
| **Monitoring** | Prometheus | Performance tracking | ✅ Impact tracked |
| **Visualization** | Grafana | Dashboard visualization | ✅ Impact visualized |
| **Response** | Velociraptor | Forensic collection | ✅ Evidence gathered |
| **Prevention** | CrowdSec | Real-time blocking | ✅ Threats monitored |

*"This is defense-in-depth: multiple security layers working together for complete visibility and rapid response. This is how enterprise SOCs operate."*

---

## Troubleshooting Demos

### If Wazuh Events Don't Show

**Check:**
```bash
# On VM #2 - verify agent is running and connected
sudo systemctl status wazuh-agent
sudo tail -30 /var/ossec/logs/ossec.log

# On VM #1 - verify agent is registered
sudo /var/ossec/bin/agent_control -l
```

**Wait 30-60 seconds** after generating events for logs to be processed.

---

### If Prometheus Shows No Data

**Check:**
```bash
# On VM #2 - verify Node Exporter is running
sudo systemctl status node_exporter
curl http://localhost:9100/metrics

# On VM #1 - check targets
# Open: http://192.168.53.24:9090/targets
# Both should show UP
```

---

### If Grafana Dashboard is Empty

**Check:**
1. Prometheus data source configured?
2. Time range set correctly? (try "Last 5 minutes")
3. Variables set? (some dashboards need hostname selected)

---

### If Velociraptor Client Not Showing

**Check:**
```bash
# On VM #2 - verify client is running
sudo systemctl status velociraptor-client
sudo journalctl -u velociraptor-client -n 50

# Check client can reach server
ping -c 4 192.168.53.24
```

**In Velociraptor GUI:**
- Click "Show All" or refresh page (F5)
- Check filter settings

---

### If CrowdSec Shows No Metrics

**Check:**
```bash
# Verify service is running
sudo systemctl status crowdsec

# Check logs
sudo tail -50 /var/log/crowdsec.log

# Verify scenarios installed
sudo cscli scenarios list
```

---

**Built with 💙 for Cybersecurity Malaysia Internship**
