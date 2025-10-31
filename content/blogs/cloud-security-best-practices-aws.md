---
title: "AWS Security Best Practices: Lessons from Production"
date: 2024-03-22
draft: false
author: "Your Name"
tags: ["aws", "cloud security", "security engineering", "devops", "best practices"]
categories: ["Security Engineering", "Cloud"]
image: /images/blogs/aws-security.png
description: "Essential AWS security practices learned from managing production environments and multiple security audits"
---

After managing AWS environments through multiple SOC 2 audits and handling security incidents, here are the critical security controls that actually matter.

## The AWS Security Pyramid

```
       [Automation & Monitoring]
         /                \
    [Identity]          [Network]
       /                    \
  [Data]                [Compute]
     \                      /
         [Logging & Audit]
```

## 1. Identity & Access Management (IAM)

### Enable MFA on EVERYTHING

**Root account MFA is non-negotiable:**
```bash
aws iam enable-mfa-device \
  --user-name root \
  --serial-number arn:aws:iam::ACCOUNT_ID:mfa/root \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

**Enforce MFA for all users:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyAllExceptListedIfNoMFA",
    "Effect": "Deny",
    "NotAction": [
      "iam:CreateVirtualMFADevice",
      "iam:EnableMFADevice",
      "iam:GetUser",
      "iam:ListMFADevices",
      "iam:ListVirtualMFADevices",
      "iam:ResyncMFADevice",
      "sts:GetSessionToken"
    ],
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
    }
  }]
}
```

### Principle of Least Privilege

**Bad IAM policy (too permissive):**
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}
```

**Good IAM policy (least privilege):**
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::my-specific-bucket/specific-prefix/*"
}
```

### Use IAM Roles, Not Access Keys

**For EC2 instances:**
```bash
# Create role
aws iam create-role --role-name MyEC2Role \
  --assume-role-policy-document file://ec2-trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name MyEC2Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Attach to instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-1234567890abcdef0 \
  --iam-instance-profile Name=MyEC2Role
```

**Benefits:**
- ✅ No credentials to manage
- ✅ Automatic rotation
- ✅ Temporary credentials
- ✅ Easier to audit

## 2. Network Security

### Default VPC = Delete It

```bash
# List default VPCs
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"

# Delete default VPC (after backing up)
aws ec2 delete-vpc --vpc-id vpc-xxxxxxxx
```

**Why?**
- Default VPC is internet-accessible
- Often misconfigured
- Not compliant with most frameworks

### Proper Network Segmentation

```
VPC Architecture:
├── Public Subnets (10.0.1.0/24, 10.0.2.0/24)
│   └── NAT Gateways, Load Balancers only
├── Private Subnets (10.0.10.0/24, 10.0.11.0/24)
│   └── Application servers
└── Database Subnets (10.0.20.0/24, 10.0.21.0/24)
    └── RDS, ElastiCache (isolated)
```

### Security Group Best Practices

**Bad security group:**
```bash
# Open to world - DON'T DO THIS
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**Good security group:**
```bash
# Restricted to specific IPs
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24  # Your office IP range
```

**Use security group chaining:**
```bash
# Web tier can access app tier
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 8080 \
  --source-group sg-web

# App tier can access database
aws ec2 authorize-security-group-ingress \
  --group-id sg-db \
  --protocol tcp \
  --port 5432 \
  --source-group sg-app
```

### VPC Flow Logs

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxx \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-flow-logs-bucket
```

**Analyze with Athena:**
```sql
-- Find rejected connections
SELECT srcaddr, dstaddr, dstport, count(*) as attempts
FROM vpc_flow_logs
WHERE action = 'REJECT'
GROUP BY srcaddr, dstaddr, dstport
ORDER BY attempts DESC
LIMIT 20;
```

## 3. Data Protection

### S3 Bucket Security

**Essential S3 bucket security:**
```bash
# Block all public access
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration \
  "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Enable logging
aws s3api put-bucket-logging \
  --bucket my-bucket \
  --bucket-logging-status file://logging.json
```

### Encryption at Rest

**RDS encryption:**
```bash
# New encrypted RDS instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --storage-encrypted \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/xxxxx
```

**EBS encryption:**
```bash
# Enable encryption by default for account
aws ec2 enable-ebs-encryption-by-default --region us-east-1
```

### AWS KMS for Key Management

```bash
# Create customer-managed key
aws kms create-key \
  --description "Application encryption key" \
  --key-policy file://key-policy.json

# Create alias
aws kms create-alias \
  --alias-name alias/my-app-key \
  --target-key-id xxxxx

# Rotate keys automatically
aws kms enable-key-rotation --key-id xxxxx
```

## 4. Logging & Monitoring

### Enable CloudTrail (All Regions)

```bash
aws cloudtrail create-trail \
  --name my-org-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging --name my-org-trail
```

**What to monitor in CloudTrail:**
```
- Console logins without MFA
- Root account usage
- IAM policy changes
- Security group changes
- S3 bucket policy changes
- Failed authentication attempts
```

### AWS Config for Compliance

```bash
# Enable Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/config-role

# Enable continuous recording
aws configservice start-configuration-recorder \
  --configuration-recorder-name default
```

**Useful Config Rules:**
- `s3-bucket-public-read-prohibited`
- `s3-bucket-public-write-prohibited`
- `rds-storage-encrypted`
- `iam-password-policy`
- `root-account-mfa-enabled`

### GuardDuty for Threat Detection

```bash
# Enable GuardDuty
aws guardduty create-detector --enable

# Get findings
aws guardduty list-findings \
  --detector-id xxxxx \
  --finding-criteria '{"Criterion":{"severity":{"Gte":7}}}'
```

### Security Hub for Central Dashboard

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests \
  StandardsArn=arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0
```

## 5. Automation & Infrastructure as Code

### Use CloudFormation/Terraform

**Example Terraform for secure EC2:**
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = "t3.micro"

  # Security
  iam_instance_profile = aws_iam_instance_profile.web.name
  vpc_security_group_ids = [aws_security_group.web.id]

  # Encryption
  ebs_optimized = true
  root_block_device {
    encrypted = true
    kms_key_id = aws_kms_key.main.arn
  }

  # Monitoring
  monitoring = true

  # Metadata service v2 (prevents SSRF)
  metadata_options {
    http_endpoint = "enabled"
    http_tokens   = "required"
  }

  tags = {
    Name = "web-server"
    Environment = "production"
  }
}
```

### Automated Security Scanning

```python
# automated_security_scan.py
import boto3

def scan_s3_buckets():
    """Find public S3 buckets"""
    s3 = boto3.client('s3')
    buckets = s3.list_buckets()['Buckets']

    issues = []
    for bucket in buckets:
        bucket_name = bucket['Name']

        # Check public access block
        try:
            pab = s3.get_public_access_block(Bucket=bucket_name)
            config = pab['PublicAccessBlockConfiguration']

            if not all([
                config.get('BlockPublicAcls'),
                config.get('BlockPublicPolicy'),
                config.get('IgnorePublicAcls'),
                config.get('RestrictPublicBuckets')
            ]):
                issues.append({
                    'bucket': bucket_name,
                    'issue': 'Public access not fully blocked'
                })
        except:
            issues.append({
                'bucket': bucket_name,
                'issue': 'No public access block configured'
            })

        # Check encryption
        try:
            encryption = s3.get_bucket_encryption(Bucket=bucket_name)
        except:
            issues.append({
                'bucket': bucket_name,
                'issue': 'No encryption configured'
            })

    return issues

# Run daily via Lambda
if __name__ == "__main__":
    issues = scan_s3_buckets()
    if issues:
        # Send to Slack/email
        print(f"Found {len(issues)} security issues")
```

## 6. Incident Response Preparation

### Enable AWS Systems Manager Session Manager

```bash
# No SSH keys needed!
aws ssm start-session --target i-1234567890abcdef0
```

**Benefits:**
- ✅ No open port 22
- ✅ All sessions logged
- ✅ IAM-based authentication
- ✅ No bastion hosts needed

### Automated Forensics

```python
# isolate_compromised_instance.py
def isolate_instance(instance_id):
    """Isolate compromised instance for forensics"""
    ec2 = boto3.client('ec2')

    # Create forensics security group (deny all)
    sg = ec2.create_security_group(
        GroupName=f'forensics-{instance_id}',
        Description='Isolated for forensics',
        VpcId='vpc-xxxxx'
    )

    # Apply to instance
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        Groups=[sg['GroupId']]
    )

    # Create snapshot for analysis
    volumes = ec2.describe_volumes(
        Filters=[{'Name': 'attachment.instance-id', 'Values': [instance_id]}]
    )

    for volume in volumes['Volumes']:
        ec2.create_snapshot(
            VolumeId=volume['VolumeId'],
            Description=f'Forensic snapshot of {instance_id}'
        )

    return sg['GroupId']
```

## Quick Wins Checklist

### Implement Today (< 1 hour)

- [ ] Enable MFA on root account
- [ ] Delete default VPC
- [ ] Enable CloudTrail in all regions
- [ ] Enable GuardDuty
- [ ] Block public S3 bucket access

### Implement This Week (< 5 hours)

- [ ] Audit IAM policies for least privilege
- [ ] Enable S3 encryption on all buckets
- [ ] Set up Security Hub
- [ ] Create security group audit
- [ ] Enable VPC Flow Logs

### Implement This Month (< 20 hours)

- [ ] Implement proper VPC architecture
- [ ] Set up centralized logging
- [ ] Create automated security scans
- [ ] Document incident response procedures
- [ ] Conduct security training

## Common AWS Security Mistakes

### ❌ Mistake #1: Hardcoded Credentials
**Fix:** Use IAM roles and Secrets Manager

### ❌ Mistake #2: Open Security Groups
**Fix:** Audit and restrict to specific IPs/ranges

### ❌ Mistake #3: No Encryption
**Fix:** Enable encryption by default

### ❌ Mistake #4: Unused Resources
**Fix:** Regular cleanup, use tags for tracking

### ❌ Mistake #5: No Monitoring
**Fix:** Enable CloudWatch, GuardDuty, Security Hub

## Cost of Security

**Monthly costs for medium environment:**
```
CloudTrail: $5
GuardDuty: $20-50
Config: $10-30
VPC Flow Logs storage: $5-15
Security Hub: $0.001 per check
Total: ~$40-100/month
```

**ROI:** Avoiding ONE breach = priceless

## Resources

**AWS Security Documentation:**
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)

**Tools:**
- [Prowler](https://github.com/prowler-cloud/prowler) - AWS security scanner
- [ScoutSuite](https://github.com/nccgroup/ScoutSuite) - Multi-cloud auditing
- [CloudMapper](https://github.com/duo-labs/cloudmapper) - Visualization

**My Tools:**
- [AWS Security Automation Scripts](https://github.com/yourusername/aws-security)

---

*Securing AWS environments?* Connect with me on [LinkedIn](https://linkedin.com/in/yourprofile) for more cloud security content!

**What's your #1 AWS security concern?** Let me know in the comments!
