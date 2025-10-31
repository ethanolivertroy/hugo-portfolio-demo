---
title: "Building the Ultimate Cybersecurity Homelab in 2024"
date: 2024-05-12
draft: false
author: "Your Name"
tags: ["homelab", "lab", "learning", "vmware", "active directory", "siem"]
categories: ["Personal", "Technical", "Learning"]
image: /images/blogs/homelab.png
description: "Complete guide to building a cybersecurity homelab for hands-on practice with offensive and defensive security"
---

A proper homelab is the secret weapon of every skilled security professional. Here's how I built mine for under $500, and how you can too.

## Why You Need a Homelab

**Benefits:**
- 🎯 **Hands-on practice** in safe environment
- 🔐 **Test attacks** without legal concerns
- 📊 **Learn defensive tools** (SIEM, IDS/IPS)
- 💼 **Portfolio projects** for resume
- 🧪 **Break things** without consequences

**My homelab projects:**
- Penetration testing practice
- Malware analysis
- SIEM configuration and tuning
- Active Directory attacks
- Security tool development

## Homelab Options: Physical vs Virtual

### Option 1: Physical Hardware

**Pros:**
- More realistic
- Better performance
- Can practice hardware hacking
- Learn enterprise equipment

**Cons:**
- Expensive ($500-2000+)
- Loud (servers have fans)
- Power hungry ($30-50/month)
- Space requirements

**My setup:** Small form factor PC + managed switch

### Option 2: Virtual (My Recommendation for Beginners)

**Pros:**
- Cheap ($0-500)
- Portable (laptop)
- Easy snapshots/cloning
- No noise/heat

**Cons:**
- Resource limited
- Can't practice physical attacks
- Less "real world"

**My recommendation:** Start virtual, add physical later if needed

## My Virtual Homelab Architecture

### Hardware Requirements

**Minimum:**
- CPU: 4 cores
- RAM: 16GB
- Storage: 500GB SSD
- Cost: $0 (use existing computer)

**Recommended:**
- CPU: 8+ cores (AMD Ryzen or Intel i7)
- RAM: 32GB (sweet spot)
- Storage: 1TB NVMe SSD
- Cost: ~$500 for dedicated mini PC

**My actual setup:**
- Intel NUC 11 Pro
- i7-1165G7 (4 cores, 8 threads)
- 64GB RAM (overkill but nice)
- 2TB NVMe SSD
- Cost: ~$800 (2 years ago)

### Virtualization Platform

**Options:**

| Platform | Cost | Pros | Cons |
|----------|------|------|------|
| **VMware Workstation** | Free (personal use) | User-friendly, great snapshot | Resource heavy |
| **Proxmox VE** | Free | Enterprise features, lightweight | Steeper learning curve |
| **VirtualBox** | Free | Cross-platform, easy | Less performant |
| **ESXi** | Free tier available | Industry standard | Requires dedicated hardware |

**My choice:** Proxmox VE (free, powerful, web-based)

## The Network Topology

```
                        Internet
                            |
                    [Physical Router]
                            |
                    [Virtual Firewall - pfSense]
                            |
                +-----------+------------+
                |           |            |
         [Security Net] [Corp Net] [Attack Net]
                |           |            |
        +-------+---+   +---+----+   +---+-----+
        |   |   |   |   |   |    |   |         |
      SIEM IDS DNS ELK  DC  Web  Kali   Vuln
                                       Machines
```

### Network Segmentation

**Why separate networks?**
- Isolate attack traffic
- Prevent lab from affecting home network
- Practice network security controls
- Realistic enterprise environment

**My networks:**

1. **Management Network (VLAN 10)**
   - Purpose: Proxmox access, administration
   - Subnet: 10.0.10.0/24

2. **Corporate Network (VLAN 20)**
   - Purpose: "Company" environment to attack
   - Subnet: 192.168.20.0/24
   - Contains: AD, workstations, servers

3. **Security Network (VLAN 30)**
   - Purpose: Security tools
   - Subnet: 10.0.30.0/24
   - Contains: SIEM, IDS, jump box

4. **Attack Network (VLAN 40)**
   - Purpose: Attacker machines
   - Subnet: 10.0.40.0/24
   - Contains: Kali, exploit dev box

5. **DMZ (VLAN 50)**
   - Purpose: Internet-facing services
   - Subnet: 172.16.50.0/24
   - Contains: Web servers, email

## Essential Virtual Machines

### 1. pfSense Firewall

**Purpose:** Network gateway, routing, firewall rules

**Specs:**
- 2 vCPU
- 2GB RAM
- 20GB disk

**Configuration:**
```bash
# WAN: Connected to physical network
# LAN1: Corporate network
# LAN2: Security network
# LAN3: Attack network
# LAN4: DMZ

# Firewall rules:
- Block Attack → Corporate by default
- Allow Security → All (monitoring)
- Allow Corporate → Internet
- Block DMZ → Corporate
```

**Why pfSense:**
- Free and powerful
- VPN server built-in
- Great logging
- Industry-relevant

### 2. Active Directory Domain Controller

**Purpose:** Practice AD attacks (Kerberoasting, Pass-the-Hash, etc.)

**Specs:**
- Windows Server 2019/2022
- 2 vCPU
- 4GB RAM
- 60GB disk

**Setup script:**
```powershell
# Install AD DS
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Configure domain
Import-Module ADDSDeployment
Install-ADDSForest `
    -DomainName "corp.lab" `
    -DomainNetbiosName "CORP" `
    -InstallDns `
    -Force

# Create vulnerable users for practice
New-ADUser -Name "SQL Service" -SamAccountName "sqlsvc" `
    -UserPrincipalName "sqlsvc@corp.lab" `
    -ServicePrincipalName "MSSQLSvc/sql.corp.lab:1433" `
    -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) `
    -Enabled $true

# Create admin user
Add-ADGroupMember -Identity "Domain Admins" -Members "sqlsvc"

# Enable old protocols for practice (LLMNR, NTLM)
```

### 3. SIEM - Splunk

**Purpose:** Log aggregation, security monitoring, alert creation

**Specs:**
- 4 vCPU
- 8GB RAM
- 100GB disk

**Why Splunk:**
- Free tier (500MB/day)
- Industry standard
- Great for resume

**Setup:**
```bash
# Download Splunk Enterprise
wget -O splunk.tgz 'https://download.splunk.com/...'

# Install
tar -xvzf splunk.tgz -C /opt
cd /opt/splunk/bin
./splunk start --accept-license

# Configure to start at boot
./splunk enable boot-start

# Install apps
- Splunk Enterprise Security (trial)
- Windows TA
- Linux TA
- Sysmon TA
```

**Log sources to configure:**
- Windows Event Logs
- Sysmon logs
- Linux syslogs
- Firewall logs
- IDS alerts

### 4. Kali Linux (Attacker)

**Purpose:** Penetration testing, exploitation

**Specs:**
- 2 vCPU
- 4GB RAM
- 80GB disk

**My customization:**
```bash
# Update everything
sudo apt update && sudo apt upgrade -y

# Install additional tools
sudo apt install -y \
    bloodhound \
    covenant \
    crackmapexec \
    evil-winrm \
    responder

# Install custom tools
git clone https://github.com/ly4k/Certipy.git
cd Certipy && python3 setup.py install

# Setup aliases
echo "alias ll='ls -la'" >> ~/.zshrc
echo "alias ports='netstat -tuln'" >> ~/.zshrc
```

### 5. Windows 10 Workstation

**Purpose:** User endpoint for attacks

**Specs:**
- 2 vCPU
- 4GB RAM
- 60GB disk

**Configuration:**
```powershell
# Join to domain
Add-Computer -DomainName "corp.lab" -Credential (Get-Credential)

# Install Sysmon for logging
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "Sysmon.zip"
Expand-Archive Sysmon.zip
cd Sysmon
.\sysmon64.exe -accepteula -i sysmonconfig.xml

# Configure Splunk forwarder
Install-Package splunkforwarder.msi
```

### 6. Vulnerable Machines

**Purpose:** Practice exploitation without legal concerns

**My collection:**
- **Metasploitable 2** - Linux vulnerabilities
- **DVWA** - Web application testing
- **Vulnhub machines** - Various challenges
- **HackTheBox retired boxes** - Real-world scenarios

**Specs per machine:**
- 1-2 vCPU
- 2GB RAM
- 20GB disk

## Advanced Components

### Security Onion (IDS/NSM)

**Purpose:** Network security monitoring, intrusion detection

**Specs:**
- 4 vCPU
- 12GB RAM
- 200GB disk

**What it includes:**
- Suricata (IDS)
- Zeek (network monitoring)
- Elasticsearch (storage)
- Kibana (visualization)

**Configuration:**
```bash
# Configure to monitor Corp network
# Setup rules for:
- Lateral movement detection
- C2 communication
- Data exfiltration
- Credential dumping
```

### ELK Stack (Alternative to Splunk)

**Purpose:** Free alternative for log aggregation

**Components:**
- Elasticsearch (storage)
- Logstash (processing)
- Kibana (visualization)
- Beats (shippers)

**Benefits:**
- Completely free
- Open source
- Scalable

### Malware Analysis Lab

**Purpose:** Safe malware analysis

**Components:**
- REMnux (Linux for malware analysis)
- FlareVM (Windows malware analysis)
- Cuckoo Sandbox (automated analysis)

**Critical:** Completely isolated network, no internet

## Infrastructure as Code

**Why automate?**
- Rebuild lab quickly
- Share configurations
- Version control
- Consistent deployments

### My Terraform Setup

```hcl
# terraform/main.tf
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
    }
  }
}

provider "proxmox" {
  pm_api_url = "https://proxmox.lab:8006/api2/json"
  pm_user    = "terraform@pam"
  pm_password = var.proxmox_password
}

resource "proxmox_vm_qemu" "windows_dc" {
  name        = "DC01"
  target_node = "proxmox"
  clone       = "win2019-template"

  cores   = 2
  memory  = 4096
  sockets = 1

  network {
    model  = "virtio"
    bridge = "vmbr20"  # Corporate network
  }

  disk {
    size    = "60G"
    type    = "virtio"
    storage = "local-lvm"
  }
}

# Similar resources for other VMs...
```

### Ansible Playbooks

```yaml
# ansible/configure-siem.yml
---
- name: Configure Splunk SIEM
  hosts: siem
  become: yes
  tasks:
    - name: Install Splunk
      unarchive:
        src: "{{ splunk_package }}"
        dest: /opt
        remote_src: yes

    - name: Start Splunk
      command: /opt/splunk/bin/splunk start --accept-license --answer-yes

    - name: Install apps
      command: /opt/splunk/bin/splunk install app {{ item }}
      loop:
        - splunk-windows-ta.tgz
        - splunk-linux-ta.tgz
```

## Practice Scenarios

### Scenario 1: AD Penetration Test

**Objective:** Compromise domain admin from user account

**Attack path:**
```
1. Enumerate domain (bloodhound)
2. Kerberoast (find weak SPN passwords)
3. Crack hash (hashcat)
4. Use credentials to move laterally
5. Dump LSASS (mimikatz)
6. Pass-the-hash to domain controller
7. Extract NTDS.dit
```

**Blue team:** Monitor in SIEM for detection

### Scenario 2: Web Application Attack

**Objective:** SQL injection to RCE

**Attack path:**
```
1. Find SQL injection (DVWA)
2. Exploit to dump database
3. Find credentials
4. Upload web shell
5. Reverse shell
6. Privilege escalation
7. Persistence
```

**Blue team:** Tune WAF rules to detect

### Scenario 3: Ransomware Simulation

**Objective:** Simulate ransomware attack

**Attack path:**
```
1. Initial access (phishing simulation)
2. Execute payload
3. Disable defenses
4. Lateral movement
5. Data exfiltration
6. Encryption (safe script, not real crypto)
```

**Blue team:** Practice incident response

## Cost Breakdown

### Budget Option ($0)

```
Hardware: Your existing laptop
Hypervisor: VirtualBox (free)
VMs: All free (Windows trial, Linux)
Total: $0 (just your time)
```

### Recommended Option ($300-500)

```
Hardware: Used mini PC on eBay ($200-400)
  - Intel NUC 8-10 gen
  - 32GB RAM
  - 512GB SSD
Storage: Additional 1TB SSD ($60)
Network: Managed switch (optional, $40)
Total: ~$300-500
```

### My Setup (~$800)

```
Hardware: Intel NUC 11 Pro ($550)
RAM upgrade: 64GB ($180)
Storage: 2TB NVMe ($150)
Switch: Netgear 8-port managed ($80)
Total: ~$960
```

**ROI:** Priceless. This lab helped me:
- Pass OSCP certification
- Land current job
- Build portfolio projects
- Continuous learning

## Maintenance & Best Practices

### Regular Tasks

**Weekly:**
- [ ] Update Kali Linux
- [ ] Review SIEM alerts
- [ ] Backup important VMs

**Monthly:**
- [ ] Update all VMs
- [ ] Review and update firewall rules
- [ ] Practice new techniques

**Quarterly:**
- [ ] Rebuild vulnerable machines
- [ ] Update AD with new attack vectors
- [ ] Review and optimize resource usage

### Snapshot Strategy

```
Before major changes:
1. Snapshot all VMs
2. Perform test/attack
3. Document results
4. Revert or keep based on success

Permanent snapshots:
- Clean AD install
- Configured SIEM baseline
- Fresh Kali with tools
```

### Documentation

**What I document:**
- Network diagrams (draw.io)
- VM inventory (spreadsheet)
- Attack paths (markdown)
- Detection rules (git repo)
- Configuration files (git repo)

**Benefits:**
- Easy rebuilds
- Portfolio evidence
- Share with community
- Learn from past experiments

## Resources for Learning

### Free Content

**YouTube Channels:**
- NetworkChuck (networking + labs)
- IPPS

ec (HTB walkthroughs)
- John Hammond (CTF + malware analysis)

**Websites:**
- r/homelab (community)
- homelabos.com (automation)
- VulnHub (vulnerable machines)

### Paid Training Using Homelab

**Practice areas:**
- OSCP (penetration testing)
- eCPPT (professional pentesting)
- Blue Team Labs Online (defensive)
- Cybrary courses

## Common Mistakes to Avoid

### ❌ Building Too Big Too Soon
Start small, add gradually. Don't try to replicate enterprise on day 1.

### ❌ Not Documenting
Write down what you build. Future you will thank you.

### ❌ Neglecting Backups
Snapshot before major changes. Lost work is frustrating.

### ❌ Poor Network Segmentation
Isolate attack network properly. Don't compromise home network.

### ❌ Not Using Homelab
Building is fun, but **use it regularly** for value.

## Next Steps

### Month 1: Foundation
- Install hypervisor
- Build pfSense
- Create AD domain
- Add Kali Linux

### Month 2: Security Tools
- Install SIEM
- Configure log forwarding
- Add IDS
- Create alerts

### Month 3: Practice
- Practice AD attacks
- Learn SIEM querying
- Build detection rules
- Document everything

### Month 4+: Advanced
- Add malware analysis
- Terraform everything
- Build custom tools
- Share knowledge (blog posts)

## Conclusion

A homelab is the difference between knowing theory and **actually doing security work**.

**Benefits:**
- ✅ Hands-on experience
- ✅ Safe practice environment
- ✅ Portfolio projects
- ✅ Continuous learning
- ✅ Career advancement

**Investment:**
- 💰 $0-800 (one-time)
- ⏰ Ongoing time (but fun!)
- 📈 Career ROI: Infinite

**Start today.** Even a simple lab beats no lab.

---

*Want my complete homelab documentation?* Check out my [GitHub repo](https://github.com/yourusername/security-homelab) with Terraform configs, Ansible playbooks, and setup guides.

*Questions about homelabs?* Hit me up on [Twitter](https://twitter.com/yourusername) or [LinkedIn](https://linkedin.com/in/yourprofile)!

**Building a homelab?** What's your first component? Let me know in the comments!
