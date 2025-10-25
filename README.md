# CloudLaunch - AWS Infrastructure Project

A secure cloud platform demonstrating AWS fundamentals including S3 static website hosting, IAM access control, and VPC network design.

## Project Overview

CloudLaunch showcases a basic company website with secure document storage and a properly segmented network architecture using AWS core services.

## Table of Contents

- [Task 1: S3 Static Website & IAM](#task-1-s3-static-website--iam)
- [Task 2: VPC Network Design](#task-2-vpc-network-design)
- [Access Credentials](#access-credentials)
- [Architecture Diagram](#architecture-diagram)
- [Verification Steps](#verification-steps)

---

## Task 1: S3 Static Website & IAM

### S3 Buckets Created

#### 1. **cloudlaunch-site-bucket-lekan659**
- **Purpose:** Hosts the public static website
- **Access:** Publicly accessible (read-only)
- **Configuration:** Static website hosting enabled
- **Website URL:** http://cloudlaunch-site-bucket-lekan659.s3-website-us-east-1.amazonaws.com

#### 2. **cloudlaunch-private-bucket-lekan659**
- **Purpose:** Private document storage
- **Access:** Restricted to `cloudlaunch-user` (GetObject/PutObject only)
- **Configuration:** Not publicly accessible

#### 3. **cloudlaunch-visible-only-bucket-lekan659**
- **Purpose:** Demonstration of list-only permissions
- **Access:** `cloudlaunch-user` can list but not access contents
- **Configuration:** Not publicly accessible

### IAM User Configuration

**User:** `cloudlaunch-user`  
**User ARN:** `arn:aws:iam::311410840081:user/cloudlaunch-user`  
**User ID:** `AIDAURAMIKIIVLVCL5V5I`

**Permissions Summary:**
- ✅ ListBucket on all three buckets
- ✅ GetObject on `cloudlaunch-site-bucket`
- ✅ GetObject/PutObject on `cloudlaunch-private-bucket`
- ❌ No DeleteObject permissions anywhere
- ❌ No content access to `cloudlaunch-visible-only-bucket`
- ✅ Read-only VPC access (describe operations)

### IAM Policy Document

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-lekan659",
                "arn:aws:s3:::cloudlaunch-private-bucket-lekan659",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-lekan659"
            ]
        },
        {
            "Sid": "ReadOnlySiteBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-lekan659/*"
        },
        {
            "Sid": "ReadWritePrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket-lekan659/*"
        },
        {
            "Sid": "NoAccessToVisibleOnlyBucketContents",
            "Effect": "Deny",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-visible-only-bucket-lekan659/*"
        },
        {
            "Sid": "ReadOnlyVPCAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways"
            ],
            "Resource": "*"
        }
    ]
}
```

---

## Task 2: VPC Network Design

### VPC Configuration

**VPC Name:** `cloudlaunch-vpc`  
**VPC ID:** `vpc-00964837c7beef6b5`  
**CIDR Block:** `10.0.0.0/16`  
**State:** Active

### Subnets

| Subnet Name | CIDR Block | Type | Purpose |
|-------------|------------|------|---------|
| cloudlaunch-public-subnet | 10.0.1.0/24 | Public | Load balancers, public-facing services |
| cloudlaunch-app-subnet | 10.0.2.0/24 | Private | Application servers |
| cloudlaunch-db-subnet | 10.0.3.0/28 | Private | Database services (RDS) |

### Internet Gateway

**Name:** `cloudlaunch-igw`  
**Attached to:** `cloudlaunch-vpc` (vpc-00964837c7beef6b5)

### Route Tables

#### 1. cloudlaunch-public-rt
- **Associated with:** Public Subnet
- **Routes:**
  - `10.0.0.0/16` → local (VPC)
  - `0.0.0.0/0` → Internet Gateway (cloudlaunch-igw)

#### 2. cloudlaunch-app-rt
- **Associated with:** App Subnet
- **Routes:**
  - `10.0.0.0/16` → local (VPC only)
  - No internet route (fully private)

#### 3. cloudlaunch-db-rt
- **Associated with:** Database Subnet
- **Routes:**
  - `10.0.0.0/16` → local (VPC only)
  - No internet route (fully private)

### Security Groups

#### cloudlaunch-app-sg
- **Purpose:** Application tier security
- **Inbound Rules:**
  - Port 80 (HTTP) from 10.0.0.0/16 (VPC only)
- **Outbound Rules:**
  - All traffic allowed (default)

#### cloudlaunch-db-sg
- **Purpose:** Database tier security
- **Inbound Rules:**
  - Port 3306 (MySQL) from 10.0.2.0/24 (App subnet only)
- **Outbound Rules:**
  - All traffic allowed (default)

### Network Architecture

```
Internet
    │
    ↓
[Internet Gateway]
    │
    ↓
[Public Subnet - 10.0.1.0/24]
(Load Balancers, Public Services)
    │
    ↓
[App Subnet - 10.0.2.0/24]
(Application Servers - Private)
    │
    ↓
[DB Subnet - 10.0.3.0/28]
(Database Services - Private)
```

**Security Design:**
- Public subnet has internet access via IGW
- App and DB subnets are fully private (no NAT Gateway)
- Security groups enforce least-privilege access
- Database only accessible from app tier

---

## 🔐 Access Credentials

### AWS Console Access

**Console Sign-In URL:** https://311410840081.signin.aws.amazon.com/console

**Account ID:** 311410840081

**IAM User:** `cloudlaunch-user`  
**Initial Password:** `CloudLaunch2025!`  
**Password Reset Required:** Yes (on first login)

### Programmatic Access

**Note:** Access keys are provided separately for security. Contact the project maintainer for programmatic access credentials.

**AWS CLI Configuration:**
```bash
aws configure --profile cloudlaunch
# Enter Access Key ID: [Provided separately]
# Enter Secret Access Key: [Provided separately]
# Region: us-east-1
# Output format: json
```

---

## 🧪 Verification Steps

### Test S3 Access

```bash
# Test as cloudlaunch-user
aws s3 ls --profile cloudlaunch
# Should see all three buckets

# Test read access to site bucket
aws s3 cp s3://cloudlaunch-site-bucket-lekan659/index.html . --profile cloudlaunch
# Should succeed

# Test write to private bucket
echo "test" > test.txt
aws s3 cp test.txt s3://cloudlaunch-private-bucket-lekan659/ --profile cloudlaunch
# Should succeed

# Test read from private bucket
aws s3 cp s3://cloudlaunch-private-bucket-lekan659/test.txt . --profile cloudlaunch
# Should succeed

# Try to access visible-only bucket contents
aws s3 cp s3://cloudlaunch-visible-only-bucket-lekan659/file.txt . --profile cloudlaunch
# Should fail (Access Denied)

# Try to delete from any bucket
aws s3 rm s3://cloudlaunch-private-bucket-lekan659/test.txt --profile cloudlaunch
# Should fail (Access Denied)
```

### Test VPC Access

```bash
# View VPC (should succeed)
aws ec2 describe-vpcs --profile cloudlaunch

# View Subnets (should succeed)
aws ec2 describe-subnets --profile cloudlaunch

# Try to create a resource (should fail)
aws ec2 create-security-group --group-name test --description test --vpc-id vpc-00964837c7beef6b5 --profile cloudlaunch
# Should fail (Access Denied)
```

### Access Static Website

Simply visit: http://cloudlaunch-site-bucket-lekan659.s3-website-us-east-1.amazonaws.com

---
---

## Project Completion Checklist

- [x] Three S3 buckets created with correct configurations
- [x] Static website hosted and publicly accessible
- [x] IAM user created with custom policy
- [x] Least-privilege permissions implemented
- [x] VPC created with CIDR 10.0.0.0/16
- [x] Three subnets (public, app, db) configured
- [x] Internet Gateway attached
- [x] Route tables properly configured
- [x] Security groups with correct rules
- [x] IAM user has read-only VPC access
- [x] Password reset enforced on first login
- [x] Comprehensive documentation provided

---

## Key Features

**Security:**
- Least-privilege IAM policies
- Network segmentation with private subnets
- Security group restrictions
- No delete permissions for IAM user

**Best Practices:**
- JSON policy documents for clarity
- Tagged resources for organization
- Separate route tables per subnet type
- Explicit deny for sensitive bucket contents

**Scalability:**
- VPC designed for future expansion
- Subnet sizing appropriate for growth
- Security group architecture ready for EC2 instances

---

## 📝 Notes

- The static website is accessible without authentication
- The private bucket requires authenticated access with proper IAM permissions
- The VPC is designed but no EC2 instances are deployed (as per requirements)
- All resources are tagged with descriptive names for easy identification

---

## 🔗 Links

- **Static Website:** http://cloudlaunch-site-bucket-lekan659.s3-website-us-east-1.amazonaws.com
- **AWS Console:** https://311410840081.signin.aws.amazon.com/console
- **Account ID:** 311410840081

---

**Project by:** Lekan  
**Date:** October 2025  
**AWS Region:** us-east-1