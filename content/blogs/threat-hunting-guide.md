---
title: "Proactive Threat Hunting: A Blue Team's Guide"
date: 2024-04-18
draft: false
author: "Your Name"
tags: ["threat hunting", "blue team", "soc", "defensive security", "siem"]
categories: ["SOC/Defensive", "Technical"]
image: /images/blogs/threat-hunting.png
description: "Learn proactive threat hunting techniques to find adversaries hiding in your network before they cause damage"
---

Waiting for alerts isn't enough. Learn how to proactively hunt for threats in your environment using the MITRE ATT&CK framework and real-world techniques.

## What is Threat Hunting?

**Definition:** Proactive searching for cyber threats that evade existing security solutions.

**Key difference from SOC monitoring:**
- ❌ SOC: React to alerts
- ✅ Threat Hunting: Proactively search for unknown threats

**Why hunt?**
- Average breach detection time: 207 days (IBM 2023)
- Adversaries are already inside
- Alerts miss sophisticated attacks
- Reduce dwell time

## The Threat Hunting Lifecycle

```
1. Hypothesis
   ↓
2. Investigation
   ↓
3. Discovery
   ↓
4. Response
   ↓
5. Documentation → (Feed back to Detection Engineering)
```

## Hunt Hypothesis Development

### Hypothesis Formula

```
"If [ADVERSARY BEHAVIOR] is occurring,
then [OBSERVABLE ARTIFACTS] should exist in [DATA SOURCE]"
```

### Examples:

**Hypothesis 1: Kerberoasting**
```
If attackers are Kerberoasting,
then requests for service tickets (TGS-REQ) with RC4 encryption
should exist in Windows Security Event logs (4769)
```

**Hypothesis 2: Living-off-the-Land**
```
If attackers are using LOLBins for persistence,
then suspicious usage of legitimate tools
should exist in Sysmon ProcessCreate events
```

**Hypothesis 3: Data Exfiltration**
```
If attackers are exfiltrating data,
then unusually large outbound transfers
should exist in network flow logs
```

## Essential Data Sources

### Tier 1: Must-Have

**Windows Event Logs:**
- Security.evtx (Event ID 4624, 4625, 4672, 4776, 4769)
- System.evtx
- Application.evtx

**Sysmon:**
```xml
<!-- Essential Sysmon Events -->
Event ID 1: Process Creation
Event ID 3: Network Connection
Event ID 7: Image Loaded
Event ID 8: CreateRemoteThread
Event ID 10: ProcessAccess
Event ID 11: FileCreate
Event ID 22: DNS Query
```

**Network Logs:**
- Firewall logs
- Proxy logs
- DNS logs
- NetFlow/IPFIX

### Tier 2: Advanced

- PowerShell logging
- EDR telemetry (CrowdStrike, SentinelOne)
- Cloud logs (Azure AD, AWS CloudTrail)
- Email gateway logs
- VPN logs

## Hunt Mission 1: Detecting Lateral Movement

### MITRE ATT&CK Technique: Pass-the-Hash (T1550.002)

**Hypothesis:**
"Attackers with compromised credentials are using Pass-the-Hash for lateral movement"

**Indicators:**
- Logon Type 3 (network) from unusual sources
- NTLM authentication when Kerberos expected
- Multiple failed logons followed by success
- Logons from service accounts to workstations

### Splunk Hunt Query

```spl
index=windows EventCode=4624 Logon_Type=3
| eval hour=strftime(_time,"%H")
| stats count by src_ip, user, hour
| where count > 10
| search NOT src_ip IN ("10.0.0.100", "10.0.0.101")  # Known servers
| lookup user_baseline.csv user OUTPUT normal_src_ips
| where NOT match(src_ip, normal_src_ips)
| table _time, user, src_ip, dest, count
| sort - count
```

**What this detects:**
- Unusual source IPs for users
- High volume of network logons
- Logons outside normal hours

### Investigation Steps

1. **Identify anomaly:**
   ```
   User: backupsvc
   Source IP: 10.50.25.143 (workstation)
   Destination: Multiple servers
   Time: 2:47 AM (outside business hours)
   ```

2. **Expand investigation:**
   ```spl
   index=windows user="backupsvc"
   | stats count by EventCode, src_ip, dest
   | sort - count
   ```

3. **Check for privilege abuse:**
   ```spl
   index=windows user="backupsvc" EventCode IN (4672, 4673, 4674)
   ```

4. **Hunt for tools:**
   ```spl
   index=sysmon EventCode=1 CommandLine="*mimikat*" OR CommandLine="*psexec*"
   | table _time, user, Computer, CommandLine
   ```

### Outcome & Response

**If confirmed:**
- Isolate affected systems
- Reset compromised credentials
- Review access logs
- Check for data exfiltration
- Create detection rule

## Hunt Mission 2: Command and Control Detection

### MITRE ATT&CK Technique: C2 via Web Protocols (T1071.001)

**Hypothesis:**
"Malware is beaconing to C2 servers using HTTPS"

**Indicators:**
- Regular outbound connections (beaconing pattern)
- Connections to newly registered domains
- Unusual TLS certificates
- High-entropy domain names

### Investigation with Zeek Logs

```bash
# Find beaconing patterns (regular intervals)
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p duration \
| awk '{if ($2 !~ /^(10\.|192\.168\.|172\.(1[6-9]|2[0-9]|3[01]))/) print}' \
| sort | uniq -c | sort -rn | head -20
```

### Splunk Query for C2 Detection

```spl
index=proxy
| bin _time span=1h
| stats dc(uri) as unique_urls, count by _time, src_ip, dest_domain
| where count > 100 AND unique_urls < 5
| eval beacon_score = count / unique_urls
| where beacon_score > 50
| table _time, src_ip, dest_domain, count, unique_urls, beacon_score
| sort - beacon_score
```

**What this detects:**
- High frequency connections
- Low URI diversity (beaconing characteristic)
- Potential C2 communication

### Advanced: Domain Age Analysis

```python
import whois
from datetime import datetime, timedelta

def check_domain_age(domain):
    """Check if domain was recently registered"""
    try:
        w = whois.whois(domain)
        creation_date = w.creation_date

        if isinstance(creation_date, list):
            creation_date = creation_date[0]

        age = (datetime.now() - creation_date).days

        if age < 30:
            return {"domain": domain, "age_days": age, "suspicious": True}
        return {"domain": domain, "age_days": age, "suspicious": False}
    except:
        return {"domain": domain, "error": "Unable to query"}

# Check domains from proxy logs
suspicious_domains = [
    "update-center-ms[.]com",
    "secure-cdn-cache[.]net",
    "api-services-v2[.]org"
]

for domain in suspicious_domains:
    result = check_domain_age(domain)
    print(result)
```

## Hunt Mission 3: Privilege Escalation

### MITRE ATT&CK: Token Impersonation (T1134)

**Hypothesis:**
"Attackers are abusing tokens for privilege escalation"

**Indicators:**
- Processes running with unexpected integrity levels
- SeDebugPrivilege abuse
- Token manipulation events

### Hunt Query

```spl
index=sysmon EventCode=10 TargetImage="C:\\Windows\\System32\\lsass.exe"
| search NOT SourceImage IN (
    "C:\\Windows\\System32\\svchost.exe",
    "C:\\Program Files\\*",
    "C:\\Windows\\System32\\wbem\\WmiPrvSE.exe"
)
| stats count by SourceImage, SourceUser
| sort - count
```

**What we're looking for:**
- Unusual processes accessing LSASS
- Credential dumping attempts
- Token theft indicators

### Investigation Flow

```
1. Identify suspicious LSASS access
   ↓
2. Check process lineage (parent-child)
   ↓
3. Look for associated network activity
   ↓
4. Search for persistence mechanisms
   ↓
5. Correlate with authentication logs
```

### Splunk Process Tree Query

```spl
index=sysmon EventCode=1
| eval process_tree = ParentImage + " → " + Image
| table _time, Computer, User, process_tree, CommandLine
| search CommandLine="*-enc*" OR CommandLine="*bypass*" OR CommandLine="*hidden*"
```

## Hunt Mission 4: Data Exfiltration

### MITRE ATT&CK: Exfiltration Over C2 Channel (T1041)

**Hypothesis:**
"Attackers are exfiltrating data over encrypted channels"

**Indicators:**
- Large outbound transfers
- Transfers during off-hours
- Uploads to cloud storage
- Unusual protocols

### NetFlow Analysis

```spl
index=netflow
| stats sum(bytes_out) as total_upload by src_ip, dest_ip
| where total_upload > 1073741824  # 1GB threshold
| lookup asset_inventory src_ip OUTPUT asset_type, normal_traffic
| where total_upload > normal_traffic * 3  # 3x normal
| table src_ip, asset_type, dest_ip, total_upload
| sort - total_upload
```

### DNS Tunneling Detection

```spl
index=dns
| eval query_length = len(query)
| stats avg(query_length) as avg_length, max(query_length) as max_length, dc(query) as unique_queries by src_ip
| where avg_length > 40 OR max_length > 60
| table src_ip, avg_length, max_length, unique_queries
| sort - avg_length
```

**Red flags:**
- Average query length > 40 characters
- High entropy subdomain names
- Many unique queries to same domain

## Building a Threat Hunt Playbook

### Playbook Template

```markdown
# Hunt Playbook: [NAME]

**MITRE ATT&CK Technique:** [ID and Name]
**Hunt Hypothesis:** [Hypothesis statement]
**Data Sources Required:** [List sources]

## Investigation Steps

1. Initial query:
   [Splunk/query here]

2. Pivot points:
   - [Correlation 1]
   - [Correlation 2]

3. Indicators of compromise:
   - [IOC 1]
   - [IOC 2]

## Expected False Positives
- [FP scenario 1]
- [FP scenario 2]

## Response Actions
1. [Action 1]
2. [Action 2]

## Detection Engineering Feedback
- Create alert for: [condition]
- Tune existing rule: [rule name]
```

### Example Playbooks to Build

1. **Kerberoasting Detection**
2. **Golden Ticket Detection**
3. **PowerShell Empire C2**
4. **Suspicious Service Creation**
5. **RDP Session Hijacking**
6. **Scheduled Task Persistence**
7. **WMI Persistence**
8. **DLL Search Order Hijacking**

## Threat Hunting Tools

### Essential Tools

| Tool | Purpose | Cost |
|------|---------|------|
| **Splunk/ELK** | Log analysis | Free tier |
| **Velociraptor** | Endpoint forensics | Free |
| **RITA** | Beacon detection | Free |
| **Sigma** | Detection rules | Free |
| **Chainsaw** | Windows event analysis | Free |
| **DeepBlueCLI** | PowerShell log analysis | Free |

### My Hunting Toolkit

```bash
# ~/.hunt_tools/

/opt/rita              # Beaconing analysis
/opt/velociraptor      # Endpoint queries
/opt/chainsaw          # Event log hunting
/opt/sigma-cli         # Sigma rule testing
/opt/jq                # JSON parsing
~/scripts/
  ├── beacon_detector.py
  ├── domain_age_checker.py
  ├── process_tree_analyzer.py
  └── lateral_movement_finder.py
```

## Metrics & Reporting

### Key Metrics

**Hunt Effectiveness:**
- Hunts conducted per month
- Threats discovered
- Mean time to discovery
- False positive rate

**Detection Coverage:**
- MITRE ATT&CK coverage %
- Techniques hunted for
- New detections created
- Reduced alert fatigue

### Hunt Report Template

```markdown
# Threat Hunt Report

**Hunt ID:** TH-2024-042
**Date:** 2024-04-18
**Hunter:** Your Name
**Status:** Completed

## Hypothesis
[What you hunted for]

## Scope
- Time range: [dates]
- Systems: [count]
- Data sources: [list]

## Findings
- Threats discovered: [count]
- False positives: [count]
- IOCs identified: [list]

## Actions Taken
1. [Action 1]
2. [Action 2]

## Detections Created
- [New rule 1]
- [Updated rule 2]

## Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Advanced Techniques

### Machine Learning for Anomaly Detection

```python
from sklearn.ensemble import IsolationForest
import pandas as pd

# Load authentication data
df = pd.read_csv('auth_logs.csv')

# Feature engineering
features = df[['hour_of_day', 'login_count', 'unique_ips', 'failed_attempts']]

# Train model
model = IsolationForest(contamination=0.1, random_state=42)
df['anomaly'] = model.fit_predict(features)

# Flag anomalies
anomalies = df[df['anomaly'] == -1]
print(f"Found {len(anomalies)} anomalous authentication patterns")
```

### Automated Hunt Scheduling

```yaml
# cron job for automated hunting
0 2 * * * /usr/bin/python3 /opt/hunt_scripts/daily_lateral_movement_hunt.py
0 3 * * * /usr/bin/python3 /opt/hunt_scripts/c2_beacon_detector.py
0 4 * * * /usr/bin/python3 /opt/hunt_scripts/privilege_escalation_hunt.py
0 5 * * * /usr/bin/python3 /opt/hunt_scripts/generate_hunt_report.py
```

## Common Mistakes to Avoid

### ❌ Hunting Without Clear Hypothesis
**Fix:** Always start with MITRE ATT&CK framework

### ❌ Not Documenting Findings
**Fix:** Document everything, even negative results

### ❌ Ignoring False Positives
**Fix:** Tune and refine, create whitelist logic

### ❌ Hunting in Isolation
**Fix:** Share findings with team, create detections

### ❌ Alert Fatigue from Hunts
**Fix:** Hunt != Alert. Create high-fidelity detections only

## Getting Started with Threat Hunting

### Week 1: Prepare
- Map your data sources
- Understand MITRE ATT&CK
- Set up hunting environment
- Create hypothesis backlog

### Week 2: First Hunt
- Pick easy technique (e.g., suspicious PowerShell)
- Write hypothesis
- Execute hunt
- Document findings

### Week 3: Refine
- Tune detection rules
- Add to playbook library
- Share with team
- Plan next hunt

### Week 4+: Mature
- Schedule regular hunts
- Build automation
- Track metrics
- Continuous improvement

## Resources for Hunters

**Learning:**
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Sigma Rules Repository](https://github.com/SigmaHQ/sigma)
- [SANS Threat Hunting Summit](https://www.sans.org/cyber-security-summit/)

**Communities:**
- r/blueteamsec
- ThreatHunting Slack
- SANS DFIR Slack

**Books:**
- "Practical Threat Intelligence and Data-Driven Threat Hunting" - Valentina Costa-Gazcón
- "The Practice of Network Security Monitoring" - Richard Bejtlich

## Conclusion

Threat hunting is the difference between **finding threats** and **waiting to be breached**.

**Key Principles:**
- 🎯 Hypothesis-driven approach
- 📊 Data is your foundation
- 🔄 Continuous iteration
- 📝 Document everything
- 🚨 Feed back to detection engineering

**Start hunting today.** Even one hunt per week will mature your security program.

---

*Want my complete threat hunting playbook library?* Check out my [GitHub repo](https://github.com/yourusername/threat-hunting-playbooks) with Sigma rules and Splunk queries.

*Questions about threat hunting?* Let's connect on [LinkedIn](https://linkedin.com/in/yourprofile)!

**What's your favorite hunting technique?** Share in the comments!
