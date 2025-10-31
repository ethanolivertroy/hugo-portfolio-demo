---
title: "Automating Security Operations with Python"
date: 2024-08-25
draft: false
author: "Your Name"
tags: ["python", "automation", "security engineering", "devops", "scripting"]
categories: ["Security Engineering", "Technical"]
image: /images/blogs/python-security.png
description: "Learn how to automate common security tasks with Python to improve efficiency and reduce human error"
---

Manual security tasks are time-consuming and error-prone. In this guide, I'll share practical Python automation scripts I use daily to streamline security operations.

## Why Automate Security Tasks?

**Benefits:**
- ⚡ **Speed**: Automated tasks run in seconds vs hours
- 🎯 **Consistency**: Same process every time, no human error
- 📊 **Scalability**: Handle thousands of assets effortlessly
- 🔄 **Repeatability**: Schedule tasks to run automatically
- 🧠 **Free up time**: Focus on complex analysis, not repetitive tasks

## Environment Setup

### Required Libraries

```bash
pip install requests boto3 python-dotenv pandas openpyxl jira slack-sdk
```

### Project Structure

```
security-automation/
├── .env  # Store secrets
├── config.py  # Configuration
├── utils/
│   ├── aws_helper.py
│   ├── slack_helper.py
│   └── jira_helper.py
└── scripts/
    ├── vuln_scan_report.py
    ├── access_review.py
    ├── cert_monitor.py
    └── threat_intel.py
```

## Use Case 1: Automated SSL Certificate Monitoring

**Problem:** Manual cert expiration checks miss renewals, causing outages

**Solution:** Automated daily monitoring with Slack alerts

```python
# cert_monitor.py
import ssl
import socket
from datetime import datetime, timedelta
import pandas as pd
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError
import os
from dotenv import load_dotenv

load_dotenv()

class CertificateMonitor:
    def __init__(self):
        self.slack_client = WebClient(token=os.getenv('SLACK_BOT_TOKEN'))
        self.slack_channel = '#security-alerts'
        self.warning_days = 30  # Alert if cert expires within 30 days

    def check_ssl_cert(self, hostname, port=443):
        """Check SSL certificate expiration"""
        try:
            context = ssl.create_default_context()
            with socket.create_connection((hostname, port), timeout=10) as sock:
                with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                    cert = ssock.getpeercert()

                    # Parse expiration date
                    expires = datetime.strptime(
                        cert['notAfter'],
                        '%b %d %H:%M:%S %Y %Z'
                    )

                    days_remaining = (expires - datetime.now()).days

                    return {
                        'hostname': hostname,
                        'expires': expires.strftime('%Y-%m-%d'),
                        'days_remaining': days_remaining,
                        'issuer': cert['issuer'][1][0][1],
                        'status': self._get_status(days_remaining)
                    }
        except Exception as e:
            return {
                'hostname': hostname,
                'error': str(e),
                'status': 'ERROR'
            }

    def _get_status(self, days_remaining):
        """Determine certificate status"""
        if days_remaining < 0:
            return 'EXPIRED'
        elif days_remaining <= 7:
            return 'CRITICAL'
        elif days_remaining <= self.warning_days:
            return 'WARNING'
        else:
            return 'OK'

    def send_slack_alert(self, cert_data):
        """Send alert to Slack"""
        if cert_data['status'] in ['EXPIRED', 'CRITICAL', 'WARNING']:
            color = {
                'EXPIRED': 'danger',
                'CRITICAL': 'danger',
                'WARNING': 'warning'
            }[cert_data['status']]

            message = {
                "channel": self.slack_channel,
                "attachments": [{
                    "color": color,
                    "title": f"SSL Certificate Alert: {cert_data['hostname']}",
                    "fields": [
                        {"title": "Status", "value": cert_data['status'], "short": True},
                        {"title": "Days Remaining", "value": str(cert_data['days_remaining']), "short": True},
                        {"title": "Expires", "value": cert_data['expires'], "short": True},
                        {"title": "Issuer", "value": cert_data.get('issuer', 'Unknown'), "short": True}
                    ]
                }]
            }

            try:
                self.slack_client.chat_postMessage(**message)
            except SlackApiError as e:
                print(f"Slack error: {e.response['error']}")

    def monitor_certificates(self, domains):
        """Monitor multiple domains"""
        results = []
        for domain in domains:
            print(f"Checking {domain}...")
            cert_data = self.check_ssl_cert(domain)
            results.append(cert_data)
            self.send_slack_alert(cert_data)

        # Generate Excel report
        df = pd.DataFrame(results)
        df.to_excel(f'cert_report_{datetime.now().strftime("%Y%m%d")}.xlsx', index=False)
        print(f"\nReport generated. Total domains checked: {len(domains)}")

        return results

# Usage
if __name__ == "__main__":
    monitor = CertificateMonitor()

    domains = [
        'google.com',
        'github.com',
        'example.com',
        'yourcompany.com',
        'api.yourcompany.com'
    ]

    results = monitor.monitor_certificates(domains)
```

**Schedule with cron:**
```bash
0 9 * * * /usr/bin/python3 /path/to/cert_monitor.py
```

## Use Case 2: Automated AWS Security Audit

**Problem:** Manual AWS security checks are time-consuming

**Solution:** Automated daily security posture assessment

```python
# aws_security_audit.py
import boto3
from datetime import datetime
import pandas as pd

class AWSSecurityAuditor:
    def __init__(self, profile='default'):
        session = boto3.Session(profile_name=profile)
        self.ec2 = session.client('ec2')
        self.iam = session.client('iam')
        self.s3 = session.client('s3')

    def check_security_groups(self):
        """Find overly permissive security groups"""
        issues = []

        security_groups = self.ec2.describe_security_groups()

        for sg in security_groups['SecurityGroups']:
            for rule in sg.get('IpPermissions', []):
                for ip_range in rule.get('IpRanges', []):
                    if ip_range.get('CidrIp') == '0.0.0.0/0':
                        issues.append({
                            'type': 'Security Group',
                            'resource': sg['GroupId'],
                            'issue': f"Port {rule.get('FromPort', 'ALL')} open to 0.0.0.0/0",
                            'severity': 'HIGH',
                            'vpc': sg.get('VpcId', 'N/A')
                        })

        return issues

    def check_s3_buckets(self):
        """Check S3 bucket public access"""
        issues = []

        buckets = self.s3.list_buckets()

        for bucket in buckets['Buckets']:
            bucket_name = bucket['Name']

            try:
                # Check public access block
                public_block = self.s3.get_public_access_block(
                    Bucket=bucket_name
                )
                config = public_block['PublicAccessBlockConfiguration']

                if not all([
                    config.get('BlockPublicAcls'),
                    config.get('BlockPublicPolicy'),
                    config.get('IgnorePublicAcls'),
                    config.get('RestrictPublicBuckets')
                ]):
                    issues.append({
                        'type': 'S3 Bucket',
                        'resource': bucket_name,
                        'issue': 'Public access not fully blocked',
                        'severity': 'CRITICAL'
                    })
            except self.s3.exceptions.NoSuchPublicAccessBlockConfiguration:
                issues.append({
                    'type': 'S3 Bucket',
                    'resource': bucket_name,
                    'issue': 'No public access block configured',
                    'severity': 'CRITICAL'
                })
            except Exception as e:
                print(f"Error checking {bucket_name}: {e}")

        return issues

    def check_iam_users(self):
        """Check for IAM security issues"""
        issues = []

        users = self.iam.list_users()

        for user in users['Users']:
            username = user['UserName']

            # Check for access keys
            access_keys = self.iam.list_access_keys(UserName=username)

            for key in access_keys['AccessKeyMetadata']:
                # Check key age
                key_age = (datetime.now(key['CreateDate'].tzinfo) - key['CreateDate']).days

                if key_age > 90:
                    issues.append({
                        'type': 'IAM User',
                        'resource': username,
                        'issue': f'Access key older than 90 days ({key_age} days)',
                        'severity': 'MEDIUM'
                    })

            # Check MFA
            try:
                mfa_devices = self.iam.list_mfa_devices(UserName=username)
                if not mfa_devices['MFADevices']:
                    issues.append({
                        'type': 'IAM User',
                        'resource': username,
                        'issue': 'MFA not enabled',
                        'severity': 'HIGH'
                    })
            except Exception as e:
                print(f"Error checking MFA for {username}: {e}")

        return issues

    def run_full_audit(self):
        """Run complete security audit"""
        print("Starting AWS Security Audit...")

        all_issues = []
        all_issues.extend(self.check_security_groups())
        all_issues.extend(self.check_s3_buckets())
        all_issues.extend(self.check_iam_users())

        # Generate report
        df = pd.DataFrame(all_issues)

        if not df.empty:
            report_file = f'aws_security_audit_{datetime.now().strftime("%Y%m%d")}.xlsx'
            df.to_excel(report_file, index=False)

            # Summary
            print(f"\n=== AUDIT SUMMARY ===")
            print(f"Total issues found: {len(all_issues)}")
            print(df['severity'].value_counts())
            print(f"\nDetailed report: {report_file}")
        else:
            print("No issues found!")

        return all_issues

# Usage
if __name__ == "__main__":
    auditor = AWSSecurityAuditor(profile='production')
    issues = auditor.run_full_audit()
```

## Use Case 3: Automated Vulnerability Report Aggregation

**Problem:** Vulnerability data scattered across multiple tools

**Solution:** Consolidated reporting with priority scoring

```python
# vuln_aggregator.py
import requests
from datetime import datetime
import pandas as pd

class VulnerabilityAggregator:
    def __init__(self, nessus_url, nessus_key, nessus_secret):
        self.nessus_url = nessus_url
        self.session = requests.Session()
        self.session.headers.update({
            'X-ApiKeys': f'accessKey={nessus_key}; secretKey={nessus_secret}'
        })

    def get_nessus_vulnerabilities(self):
        """Fetch vulnerabilities from Nessus"""
        vulns = []

        # Get latest scan
        scans = self.session.get(f"{self.nessus_url}/scans").json()

        for scan in scans['scans']:
            scan_id = scan['id']

            # Get scan details
            details = self.session.get(
                f"{self.nessus_url}/scans/{scan_id}"
            ).json()

            for vuln in details.get('vulnerabilities', []):
                vulns.append({
                    'source': 'Nessus',
                    'severity': vuln['severity'],
                    'plugin_name': vuln['plugin_name'],
                    'count': vuln['count'],
                    'cvss_score': vuln.get('cvss_base_score', 0)
                })

        return vulns

    def calculate_priority(self, vuln):
        """Calculate remediation priority"""
        severity_weight = {
            'Critical': 10,
            'High': 7,
            'Medium': 4,
            'Low': 1
        }

        priority_score = (
            severity_weight.get(vuln['severity'], 0) *
            vuln.get('cvss_score', 0) *
            vuln.get('count', 1)
        )

        if priority_score >= 70:
            return 'P0 - Critical'
        elif priority_score >= 40:
            return 'P1 - High'
        elif priority_score >= 20:
            return 'P2 - Medium'
        else:
            return 'P3 - Low'

    def generate_executive_report(self, vulns):
        """Generate executive summary"""
        df = pd.DataFrame(vulns)

        # Add priority
        df['priority'] = df.apply(self.calculate_priority, axis=1)

        # Sort by priority
        df = df.sort_values('priority')

        # Save report
        report_file = f'vulnerability_report_{datetime.now().strftime("%Y%m%d")}.xlsx'

        with pd.ExcelWriter(report_file, engine='openpyxl') as writer:
            # Summary sheet
            summary = df.groupby(['severity', 'priority']).size().reset_index(name='count')
            summary.to_excel(writer, sheet_name='Summary', index=False)

            # Detailed sheet
            df.to_excel(writer, sheet_name='All Vulnerabilities', index=False)

            # Top 20 critical
            df.head(20).to_excel(writer, sheet_name='Top 20 Priority', index=False)

        print(f"Report generated: {report_file}")
        return report_file

# Usage would require actual Nessus credentials
```

## Use Case 4: Security Metrics Dashboard Automation

```python
# security_metrics.py
import boto3
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import seaborn as sns

class SecurityMetricsDashboard:
    def __init__(self):
        self.cloudwatch = boto3.client('cloudwatch')
        self.guardduty = boto3.client('guardduty')

    def get_failed_login_attempts(self, days=7):
        """Track failed authentication attempts"""
        # Query CloudWatch Logs
        end_time = datetime.now()
        start_time = end_time - timedelta(days=days)

        # This would query your actual logs
        # Placeholder for demonstration
        return [5, 12, 8, 15, 20, 10, 7]  # Daily counts

    def get_security_findings(self):
        """Get GuardDuty findings"""
        detector_id = self.guardduty.list_detectors()['DetectorIds'][0]

        findings = self.guardduty.list_findings(
            DetectorId=detector_id,
            FindingCriteria={
                'Criterion': {
                    'severity': {'Gte': 4}
                }
            }
        )

        return len(findings['FindingIds'])

    def generate_weekly_dashboard(self):
        """Generate security metrics dashboard"""
        fig, axes = plt.subplots(2, 2, figsize=(15, 10))
        fig.suptitle('Weekly Security Metrics Dashboard', fontsize=16)

        # Failed logins
        failed_logins = self.get_failed_login_attempts()
        axes[0, 0].plot(range(1, 8), failed_logins, marker='o')
        axes[0, 0].set_title('Failed Login Attempts')
        axes[0, 0].set_xlabel('Day')
        axes[0, 0].set_ylabel('Count')

        # Vulnerability severity distribution
        vuln_data = {'Critical': 5, 'High': 15, 'Medium': 30, 'Low': 50}
        axes[0, 1].pie(vuln_data.values(), labels=vuln_data.keys(), autopct='%1.1f%%')
        axes[0, 1].set_title('Vulnerability Distribution')

        # Security findings trend
        findings_trend = [8, 12, 10, 15, 11, 9, 7]
        axes[1, 0].bar(range(1, 8), findings_trend)
        axes[1, 0].set_title('Security Findings')
        axes[1, 0].set_xlabel('Day')
        axes[1, 0].set_ylabel('Count')

        # Compliance score
        compliance_scores = [85, 87, 90, 92, 93, 94, 95]
        axes[1, 1].plot(range(1, 8), compliance_scores, marker='o', color='green')
        axes[1, 1].set_title('Compliance Score Trend')
        axes[1, 1].set_xlabel('Day')
        axes[1, 1].set_ylabel('Score %')
        axes[1, 1].set_ylim([80, 100])

        plt.tight_layout()
        plt.savefig(f'security_dashboard_{datetime.now().strftime("%Y%m%d")}.png', dpi=300)
        print("Dashboard generated!")

# Usage
dashboard = SecurityMetricsDashboard()
dashboard.generate_weekly_dashboard()
```

## Best Practices

### 1. Security

```python
# Never hardcode credentials
from dotenv import load_dotenv
import os

load_dotenv()
API_KEY = os.getenv('API_KEY')

# Use AWS Secrets Manager
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])
```

### 2. Error Handling

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('security_automation.log'),
        logging.StreamHandler()
    ]
)

def safe_api_call(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logging.error(f"Error in {func.__name__}: {str(e)}")
            return None
    return wrapper
```

### 3. Rate Limiting

```python
import time
from functools import wraps

def rate_limit(max_calls=10, time_frame=60):
    """Rate limit decorator"""
    calls = []

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            calls[:] = [c for c in calls if c > now - time_frame]

            if len(calls) >= max_calls:
                sleep_time = time_frame - (now - calls[0])
                time.sleep(sleep_time)

            calls.append(time.time())
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

## Deployment

### Docker Container

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "cert_monitor.py"]
```

### Kubernetes CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: security-audit
spec:
  schedule: "0 9 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: auditor
            image: your-registry/security-automation:latest
            env:
            - name: AWS_PROFILE
              value: "production"
          restartPolicy: OnFailure
```

## Measuring Success

**Before Automation:**
- 4 hours/week on certificate monitoring
- 6 hours/week on access reviews
- 8 hours/week on vulnerability reporting

**After Automation:**
- 15 minutes/week reviewing automated reports
- **Time saved:** 17.75 hours/week
- **ROI:** Infinite (recurring time savings)

## Conclusion

Security automation isn't about replacing security professionals—it's about amplifying their effectiveness. By automating repetitive tasks, you can:

- Focus on high-value security analysis
- Reduce human error
- Improve response times
- Scale security operations

Start small, automate one task at a time, and build your security automation library.

**Want the complete code repository?** Check out my [GitHub](https://github.com/yourusername/security-automation) for all scripts and more examples!

---

*Have automation questions? Let's connect on [LinkedIn](https://linkedin.com/in/yourprofile).*
