# 🛡️ Mini SOC/XDR Environment

> A production-grade Security Operations Center with Extended Detection & Response capabilities built with open-source tools

[![Project Status](https://img.shields.io/badge/Status-Operational-success)]()
[![Built With](https://img.shields.io/badge/Built%20With-Ubuntu%2024.04-orange)]()
[![Tools](https://img.shields.io/badge/Tools-5%20Security%20Tools-blue)]()

![Architecture](https://img.shields.io/badge/Architecture-Two%20VM%20Setup-lightgrey)

---

## 📋 Overview

This project implements a complete **Security Operations Center (SOC)** with **Extended Detection & Response (XDR)** capabilities, demonstrating enterprise-grade security monitoring, threat detection, and incident response using industry-standard open-source tools.

**Built during:** Cybersecurity Malaysia Internship - Digital Forensics Department  
**Duration:** 20 weeks  
**Supervisors:** Dr. Sharifah Nurul Asyikin & Dr. Yusliza Yusoff

---

## 🏗️ Architecture

### Two-VM Setup

**VM #1 - SOC Server (Command Center)**
- **Wazuh Manager + Indexer + Dashboard** - SIEM for threat detection
- **Prometheus** - Metrics collection server
- **Grafana** - Metrics visualization
- **Velociraptor Server** - Digital forensics platform

**VM #2 - Production Server (Monitored System)**
- **Wazuh Agent** - Log collection & forwarding
- **Node Exporter** - System metrics exporter
- **CrowdSec** - Intrusion prevention system
- **Velociraptor Client** - Forensic agent

### Data Flow
```
VM #2 (Production) ──→ Logs & Metrics ──→ VM #1 (SOC) ──→ Dashboards & Alerts
```

---

## 🚀 Key Features

### Defense-in-Depth Security

| Layer | Tool | Function |
|-------|------|----------|
| **Prevention** | CrowdSec | Real-time threat blocking |
| **Detection** | Wazuh | Security event analysis |
| **Monitoring** | Prometheus | Performance tracking |
| **Visualization** | Grafana | Operational dashboards |
| **Response** | Velociraptor | Forensic investigation |

---

## 🎬 Live Demonstrations

All tools demonstrated through simulated attack scenarios:

✅ **SSH Brute Force Attack Detection** - Shows all 5 tools working together  
✅ **Real-time Security Alerting** - Wazuh SIEM event correlation  
✅ **Performance Impact Analysis** - Prometheus metric collection  
✅ **Visual Monitoring** - Grafana dashboard creation  
✅ **Forensic Evidence Collection** - Velociraptor artifact gathering  

**[→ View Complete Demo Guide](DEMOS.md)**

---

## 📊 System Specifications (Ignore low specs bacause my laptop kentang)

### VM #1 (SOC Server)
- **OS:** Ubuntu 24.04 LTS
- **RAM:** 2.5GB
- **Disk:** 28GB
- **IP:** 192.168.53.24
- **Services:** 6 security services running

### VM #2 (Production Server)
- **OS:** Ubuntu 24.04 LTS
- **RAM:** 1.5GB
- **Disk:** 13GB
- **IP:** 192.168.53.92
- **Agents:** 4 monitoring agents active

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| **[SETUP.md](SETUP.md)** | Complete installation guide |
| **[DEMOS.md](DEMOS.md)** | Step-by-step demonstration instructions |
| **[ARCHITECTURE.md](ARCHITECTURE.md)** | Technical architecture details |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues & solutions |
| **[COMMANDS.md](COMMANDS.md)** | Quick command reference |
| **[Complete Guide (PDF)](docs/Complete_SOC_Demo.pdf)** | Full documentation package |

---

## 🔧 Technical Challenges Overcome

### Challenge 1: Resource Constraints
**Problem:** 8GB laptop, Wazuh requires 4GB minimum  
**Solution:** Optimized VM allocation (2.5GB + 1.5GB), sequential workflow  
**Learning:** Resource planning critical for lab environments

### Challenge 2: Disk Space Management
**Problem:** VM #1 disk filled to 100% (17GB in Wazuh queue)  
**Solution:** Cleaned vulnerability detector cache, implemented monitoring  
**Learning:** Proactive disk space monitoring essential

### Challenge 3: Version Compatibility
**Problem:** Wazuh Agent v4.14.1 incompatible with Manager v4.9.2  
**Solution:** Downgraded agent to match manager version  
**Learning:** Version compatibility must be verified before deployment

### Challenge 4: Network Configuration
**Problem:** VMs on different subnets (192.168.53.x vs 192.168.56.x)  
**Solution:** Reconfigured VirtualBox network adapters  
**Learning:** Network architecture requires careful planning

---

## 🎓 Skills Demonstrated

✅ Linux system administration & security hardening  
✅ Security architecture design (SOC/XDR)  
✅ Service integration & orchestration  
✅ Network configuration & troubleshooting  
✅ Incident detection & response workflows  
✅ Performance monitoring & optimization  
✅ Technical documentation & presentation  

---

## 🛠️ Tools & Technologies

### Security & Monitoring
![Wazuh](https://img.shields.io/badge/Wazuh-4.9.2-blue?logo=wazuh)
![Prometheus](https://img.shields.io/badge/Prometheus-2.48.0-orange?logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-Latest-yellow?logo=grafana)
![Velociraptor](https://img.shields.io/badge/Velociraptor-0.7.1-red)
![CrowdSec](https://img.shields.io/badge/CrowdSec-Latest-green)

### Infrastructure
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-orange?logo=ubuntu)
![VirtualBox](https://img.shields.io/badge/VirtualBox-Latest-blue?logo=virtualbox)

---

## 📸 Screenshots

### Wazuh Dashboard - Active Agent Monitoring

<img width="896" height="423" alt="Screenshot 2026-01-08 094153" src="https://github.com/user-attachments/assets/d41a0717-7d2f-43e9-af8e-b210b6d35d7d" />

### Grafana - Node Exporter Dashboard

<img width="1899" height="824" alt="Screenshot 2026-01-08 153852" src="https://github.com/user-attachments/assets/0ebb7221-aa8c-4978-92c7-d55b14b4e19b" />

### Velociraptor - Forensic Collection

<img width="1892" height="811" alt="Screenshot 2026-01-08 155249" src="https://github.com/user-attachments/assets/13330b06-f576-4194-ba21-f30b69f66880" />

### Prometheus - Collect System Metrics (CPU, RAM, disk etc)

<img width="1894" height="814" alt="Screenshot 2026-01-08 170500" src="https://github.com/user-attachments/assets/3758ca5b-8f8d-4360-956d-62986c79d7a3" />

### Crowsdsec (In ProdSever) - Act as a security within the production server itself, will ban suspicious IP address etc.

<img width="1044" height="339" alt="Screenshot 2026-01-08 175620" src="https://github.com/user-attachments/assets/13f8f6e8-b30c-40d3-a5c5-bc75ddf1ec48" />

---

## 🚀 Quick Start

### Prerequisites
- VirtualBox (or similar hypervisor)
- 8GB+ RAM
- 50GB+ disk space
- Basic Linux knowledge

### Installation
```bash
# Clone the repository
git clone https://github.com/Hazim89/mini-soc-xdr-environment.git
cd mini-soc-xdr

# Follow the setup guide
cat SETUP.md
```

### Starting Services
```bash
# VM #1 (SOC Server)
~/start-soc.sh

# VM #2 (Production Server)
~/start-agents.sh
```

**[→ Complete Setup Instructions](SETUP.md)**

---

## 📊 Project Metrics

- **Duration:** 20 weeks
- **Tools Integrated:** 5 security tools
- **Services Deployed:** 10 services across 2 VMs
- **Demos Created:** 15+ demonstration scenarios
- **Lines of Config:** 500+ lines

---

## 🎯 Use Cases

### Educational
- Cybersecurity training environments
- SOC analyst skill development
- Incident response practice labs

### Professional
- Portfolio demonstration
- Interview technical discussions
- Security architecture reference

### Research
- Security tool evaluation
- Detection rule testing
- Performance benchmarking

---

## 🔗 Related Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Velociraptor Documentation](https://docs.velociraptor.app/)
- [CrowdSec Documentation](https://docs.crowdsec.net/)

---

## 📝 Future Enhancements

- [ ] Add Suricata IDS integration
- [ ] Implement ELK Stack comparison
- [ ] Create Docker compose deployment
- [ ] Add Ansible automation scripts
- [ ] Integrate MITRE ATT&CK framework
- [ ] Add Windows endpoint monitoring

---

## 👤 Author

**Ahmad Hazim Bin Ahmad Najmi (Ajim)**  
*Cybersecurity Malaysia - Digital Forensics Intern*

📧 Email: Ahazimnajmi89@gmail.com  
💼 LinkedIn: www.linkedin.com/in/hazimnajmi71324


---

## 📄 License

This project is created for educational purposes as part of the Cybersecurity Malaysia internship program.

---

## 🙏 Acknowledgments

**Supervisors:**
- Dr. Sharifah Nurul Asyikin
- Dr. Yusliza Yusoff

**Organization:**
- Cybersecurity Malaysia - Digital Forensics Department

**Community:**
- Wazuh Community
- Prometheus Community
- Grafana Community
- Velociraptor Community
- CrowdSec Community

---

## ⭐ Support

If you find this project helpful:
- ⭐ Star this repository
- 🍴 Fork it for your own learning
- 📢 Share it with others
- 💬 Provide feedback via Issues

---

**Built with 💙 during Cybersecurity Malaysia Internship**

*Last Updated: January 2026*
