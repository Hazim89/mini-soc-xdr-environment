# Velociraptor - Digital Forensics & Incident Response

## 📖 Overview

Velociraptor is an advanced digital forensics and incident response (DFIR) platform that enables security teams to investigate security incidents at scale. It provides SOC analysts with the ability to remotely collect forensic evidence from endpoints without disrupting operations or alerting attackers.

---

## 🎯 What Problem Does It Solve?

### **Traditional Incident Response Challenges:**

**Problem 1: Slow Investigation**
- Must physically or remotely access each system
- SSH/RDP creates logs that attackers can see
- Investigating 100 systems takes days

**Problem 2: Evidence Destruction**
- Attackers delete logs and artifacts
- By the time you investigate, evidence is gone
- No way to preserve volatile data

**Problem 3: Operational Impact**
- Must take systems offline for forensics
- Disk imaging disrupts business operations
- Users can't work during investigation

### **Velociraptor's Solution:**

✅ **Silent Collection:** Gather evidence without alerting the attacker
✅ **Parallel Execution:** Query 1000 endpoints simultaneously  
✅ **Zero Disruption:** Systems remain online and operational
✅ **Real-Time:** Get results in minutes, not hours
✅ **Scalable:** One analyst can investigate entire networks

---

## 🔍 Core Capabilities

### **1. VQL (Velociraptor Query Language)**

VQL is a SQL-like language designed specifically for forensic investigations.

**Example 1: Find Suspicious Processes**
```vql
SELECT Name, Pid, Username, Exe, CommandLine
FROM pslist()
WHERE Exe =~ "/tmp/|/dev/shm/|Downloads"
  OR Name =~ "nc|ncat|socat|mimikatz"
```
**What it does:** Finds processes running from temporary directories or with known hacker tool names

**Example 2: Hunt for Ransomware**
```vql
SELECT FullPath, Size, Mtime, Atime
FROM glob(globs="/**/*.encrypted")
WHERE Mtime > now() - 3600
ORDER BY Mtime DESC
```
**What it does:** Finds files with .encrypted extension modified in the last hour

**Example 3: Network Backdoors**
```vql
SELECT Laddr, Raddr, Status, Pid, Name
FROM netstat()
WHERE (Raddr.Port IN (4444, 31337, 1337, 8080) 
  OR Laddr.Port > 10000)
  AND Status = "ESTABLISHED"
```
**What it does:** Finds network connections to common backdoor ports

---

### **2. Remote Evidence Collection**

**What Gets Collected:**

**Memory Artifacts:**
- Running processes and loaded modules
- Network connections (active and listening)
- Injected code and DLL hijacking
- Command history (bash_history, PowerShell)

**Disk Artifacts:**
- File system timeline (who accessed what, when)
- Recently modified files
- Deleted files (from journals/recycle bins)
- Browser history and downloads

**System Artifacts:**
- User accounts and groups
- Scheduled tasks / cron jobs
- Installed software
- System configuration changes

**Persistence Mechanisms:**
- SSH authorized_keys (backdoor SSH access)
- Startup scripts and services
- Registry run keys (Windows)
- LD_PRELOAD hijacking (Linux)

---

### **3. Hunt Manager**

**What is a "Hunt"?**
A hunt is a VQL query sent to multiple endpoints simultaneously.

**Hunt Workflow:**
```
1. Create hunt (write VQL query)
   ↓
2. Select targets (all servers? just web servers? specific IPs?)
   ↓
3. Launch hunt (query sent to all targets)
   ↓
4. Collect results (aggregated data from all endpoints)
   ↓
5. Analyze (find patterns, indicators of compromise)
```

**Example Hunt Scenario:**

**Situation:** Suspected data breach, need to check if attacker accessed sensitive files

**Hunt Configuration:**
```vql
SELECT FullPath, AccessTime, User
FROM glob(globs="/home/*/Documents/**/*.pdf")
WHERE AccessTime > timestamp(epoch="2025-01-15")
ORDER BY AccessTime DESC
```

**Target:** All 200 file servers

**Result:** 
- 15 servers show suspicious PDF access from user "backup_admin"
- Access times: 2 AM - 4 AM (unusual hours)
- User "backup_admin" doesn't exist in Active Directory
- **Conclusion:** Attacker created fake account and exfiltrated documents

**Time to complete:** 3 minutes (vs. 30+ hours manually checking)

---

## 🎯 Integration with SOC/XDR

### **Velociraptor's Role in the Kill Chain:**
```
┌─────────────────────────────────────────────────────────┐
│  DETECT (Wazuh)                                         │
│  "Alert: Suspicious process execution on Server-42"    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  INVESTIGATE (Velociraptor)                             │
│  - What process? (pslist)                               │
│  - When did it start? (timeline)                        │
│  - What files did it touch? (file access logs)          │
│  - Who started it? (user context)                       │
│  - Is it on other systems? (hunt across fleet)          │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  RESPOND (Manual/Automated)                             │
│  - Isolate affected systems                             │
│  - Kill malicious processes                             │
│  - Remove persistence mechanisms                        │
│  - Patch vulnerabilities                                │
└─────────────────────────────────────────────────────────┘
```

---

## 🚀 Working Demo Queries

### **Demo 1: Process Enumeration**

**VQL Query:**
```vql
SELECT Name, Pid, Ppid, Username, Exe, CommandLine, CreateTime
FROM pslist()
WHERE Name =~ "ssh|bash|python|nc"
ORDER BY CreateTime DESC
LIMIT 30
```

**Demo Explanation:**
"I'm querying the production server to enumerate all running processes related to SSH, Bash shells, Python scripts, and Netcat - common tools used by attackers. In a real investigation, we'd look for:
- Processes running from /tmp/ or /dev/shm/ (attacker staging areas)
- Unusual parent-child relationships (web server spawning shell)
- Processes with obfuscated names (spaces, special characters)"

**What to point out in results:**
- Process hierarchy (parent → child relationships)
- Usernames running processes (root? web user? unknown?)
- Executable paths (system binaries vs. suspicious locations)
- Command-line arguments (what flags were used?)

---

### **Demo 2: Network Connections**

**VQL Query:**
```vql
SELECT 
  Laddr.IP AS LocalIP,
  Laddr.Port AS LocalPort,
  Raddr.IP AS RemoteIP,
  Raddr.Port AS RemotePort,
  Status,
  Pid,
  Name AS ProcessName
FROM netstat()
WHERE Status = "ESTABLISHED"
ORDER BY RemoteIP
LIMIT 30
```

**Demo Explanation:**
"Here I'm collecting active network connections to identify what external systems this server is communicating with. During incident response, this reveals:
- Command & Control servers (suspicious IPs)
- Data exfiltration (large transfers to cloud storage)
- Lateral movement (connections to other internal servers)
- Reverse shells (connections on unusual ports)"

**Red flags to highlight:**
- Connections to foreign countries (for domestic-only systems)
- High ports (>10000) often used by malware
- Common backdoor ports: 4444 (Metasploit), 31337, 1337
- Unknown processes with network activity

---

### **Demo 3: Recently Modified Files**

**VQL Query:**
```vql
SELECT FullPath, Size, Mtime, Mode, Owner
FROM glob(globs="/tmp/**")
WHERE Mtime > now() - 3600
ORDER BY Mtime DESC
LIMIT 30
```

**Demo Explanation:**
"I'm searching for files modified in the last hour in /tmp/ - a high-risk location because:
- World-writable (any user can write files there)
- Common malware staging directory
- Scripts execute from there
- Attackers drop tools there temporarily

Files we'd investigate immediately:
- .sh scripts with random names (malicious scripts)
- Binaries (nc, ncat, nmap, etc.)
- Hidden files (starting with .)
- Files with suspicious permissions (setuid, world-writable)"

**What to emphasize:**
- Size (large files could be data exfiltration staging)
- Permissions (777 = world-writable, very suspicious)
- Owner (root-owned files in /tmp/ are unusual)
- Modification time (recently modified = active threat)

---

### **Demo 4: SSH Backdoor Detection**

**VQL Query:**
```vql
SELECT FullPath, Mtime, Size,
  read_file(filename=FullPath) AS Content
FROM glob(globs="/home/*/.ssh/authorized_keys")
```

**Demo Explanation:**
"SSH authorized_keys files are a common persistence mechanism. Attackers add their own SSH public key, allowing them to log back in even if passwords change. This query collects all authorized_keys files across all user accounts.

What we look for:
- Keys with unknown fingerprints
- Keys added recently (check Mtime)
- Keys in service accounts that shouldn't have SSH access
- Suspiciously long comments (attackers hide data in comments)"

**Follow-up investigation:**
"If we find a suspicious key, we can:
1. Check auth.log to see when it was added
2. Search other systems for the same key (attacker reuses keys)
3. Identify which systems the attacker accessed with this key
4. Timeline the attacker's activities"

---

### **Demo 5: User Account Enumeration**

**VQL Query:**
```vql
SELECT User, Uid, Gid, Shell, Description, Homedir, LastLogin
FROM Artifact.Linux.Sys.Users()
WHERE (Uid >= 1000 OR Uid = 0)
  AND Shell != "/usr/sbin/nologin"
ORDER BY Uid
```

**Demo Explanation:**
"This collects all user accounts that can log in (have a shell assigned). During breach investigations, we need to identify:
- Legitimate users vs. attacker-created accounts
- Accounts with root privileges (UID 0)
- Accounts with unusual names (typosquatting legitimate accounts)
- Recently created accounts

Attacker account patterns:
- Similar to legitimate names ('admln' vs 'admin')
- System-sounding names ('apache', 'backup', 'test')
- Multiple accounts with UID 0 (should only be root)
- Accounts with /bin/bash but no LastLogin (created but not used yet)"

---

## 📊 Performance & Scalability

### **Resource Usage**

**Velociraptor Client (per endpoint):**
- Idle: ~20MB RAM, <1% CPU
- During collection: 50-100MB RAM, 5-10% CPU (brief spike)
- Network: 10KB-10MB depending on collection (typically <1MB)

**Velociraptor Server:**
- 100 clients: ~500MB RAM
- 1000 clients: ~2GB RAM
- Scales horizontally (add more servers)

### **Collection Speed**

| Artifact Type | 1 Client | 100 Clients | 1000 Clients |
|---------------|----------|-------------|--------------|
| Process List | 2 sec | 5 sec | 10 sec |
| Network Connections | 3 sec | 6 sec | 12 sec |
| File Search (100 files) | 5 sec | 15 sec | 30 sec |
| Full Memory Dump | 5 min | 5 min | 5 min* |

*Parallel execution - all clients dump simultaneously

---

## 🎓 Learning Resources

### **Official Documentation**
- [Velociraptor Docs](https://docs.velociraptor.app/)
- [VQL Reference](https://docs.velociraptor.app/vql_reference/)
- [Artifact Exchange](https://docs.velociraptor.app/exchange/)

### **Community**
- [GitHub](https://github.com/Velocidex/velociraptor)
- [Discord](https://docs.velociraptor.app/discord/)
- [Blog](https://docs.velociraptor.app/blog/)

### **Training**
- [SANS FOR509: Enterprise Cloud Forensics](https://www.sans.org/cyber-security-courses/enterprise-cloud-forensics-and-incident-response/)
- [Velociraptor Training Videos](https://www.youtube.com/@velocidex)

---

## 🔐 Security Considerations

### **Authentication**
- Mutual TLS (mTLS) - both client and server verify certificates
- Certificate pinning - prevents man-in-the-middle attacks
- No password authentication - certificates only

### **Encryption**
- All communications encrypted with TLS 1.3
- Data at rest encrypted in server database
- Certificate rotation supported

### **Access Control**
- Role-based access control (RBAC)
- Audit logging of all actions
- Read-only vs. investigator vs. administrator roles

### **Privacy**
- Targeted collection (only collect what's needed)
- Data retention policies
- GDPR compliance considerations

---

## 🏆 Real-World Success Stories

### **Case Study 1: Ransomware Response**
**Organization:** Mid-size healthcare provider (500 endpoints)

**Incident:** Ransomware encryption detected on file server

**Velociraptor Response:**
1. Hunted for ransomware process across all 500 endpoints (2 minutes)
2. Found 15 infected systems before encryption spread
3. Collected process memory dumps for malware analysis
4. Identified patient zero (initial infection point)
5. Reconstructed attacker timeline (entry → reconnaissance → encryption)

**Outcome:** 
- Contained in 30 minutes (vs. typical 6-8 hours)
- Only 15 systems affected (vs. potential 500)
- $2M+ in damages prevented
- Full forensic evidence preserved for legal action

---

### **Case Study 2: Insider Threat**
**Organization:** Financial services company

**Incident:** Suspected insider theft of customer data

**Velociraptor Investigation:**
1. Collected file access logs from all file servers (5 minutes)
2. USB device connection history from suspect's workstation
3. Browser history showing Dropbox access
4. Network connections to personal cloud storage
5. Timeline of 50GB data exfiltration over 2 weeks

**Outcome:**
- Silent investigation (suspect unaware)
- Complete evidence package for HR/legal
- Identified all exfiltrated files
- Prevented further data loss

---

## 💡 Best Practices

### **Hunt Strategy**

**1. Start Broad, Then Narrow**
```vql
-- First: Check all systems for any suspicious processes
SELECT * FROM pslist() WHERE Exe =~ "/tmp/"

-- Then: Focus on specific IOCs (indicators of compromise)
SELECT * FROM pslist() WHERE Hash.SHA256 = "abc123..."
```

**2. Preserve Context**
- Don't just collect the suspicious file
- Collect surrounding files, timestamps, access logs
- Timeline is crucial for understanding attack sequence

**3. Document Everything**
- Use Velociraptor's Notebook feature
- Add comments explaining each query
- Export results with timestamps

### **Performance Optimization**

**DO:**
- Use LIMIT to cap results
- Filter early in the query (WHERE clause)
- Schedule heavy hunts during off-hours
- Batch large collections

**DON'T:**
- Collect entire disk images unnecessarily
- Run full file system scans on production systems
- Execute unlimited queries without testing first

---

## 🔄 Integration with Other Tools

### **SIEM Integration (Wazuh)**
```
Wazuh Alert → Webhook → Velociraptor Hunt
```
**Example:** Wazuh detects malware → Automatically triggers Velociraptor to collect process memory from affected system

### **Ticketing Systems**
```
Velociraptor Hunt Result → API → Create Ticket
```
**Example:** Hunt finds suspicious activity → Auto-creates incident ticket with evidence attached

### **Threat Intelligence**
```
IOC Feed → Velociraptor Hunt
```
**Example:** New malware hash published → Velociraptor hunts all systems for that hash

---

## 📈 Metrics & Reporting

### **Key Performance Indicators (KPIs)**

**Mean Time to Investigate (MTTI):**
- **Without Velociraptor:** 2-6 hours per system
- **With Velociraptor:** 5-15 minutes across 100+ systems

**Investigation Coverage:**
- **Without Velociraptor:** 10-20 systems per incident
- **With Velociraptor:** 100% of endpoints (entire fleet)

**Evidence Completeness:**
- **Without Velociraptor:** 60-70% (attacker deletes evidence)
- **With Velociraptor:** 90-95% (silent collection preserves evidence)

### **Automated Reporting**

Velociraptor can generate:
- Executive summaries (what happened, business impact)
- Technical reports (IOCs, timeline, affected systems)
- Compliance reports (evidence handling, chain of custody)
- Incident statistics (duration, scope, severity)

---

## 🎯 Conclusion

**Velociraptor transforms incident response from:**
- Reactive → Proactive
- Manual → Automated
- Slow → Fast
- Limited → Comprehensive

**In our SOC/XDR environment, Velociraptor provides the "Response" in Extended Detection and Response:**
- **Wazuh detects** the threat
- **Velociraptor investigates** the scope
- **Analysts respond** with complete information

**This is the future of digital forensics - fast, scalable, and silent.**

---

**For questions or support, refer to the main [README.md](README.md) or open an issue.**
