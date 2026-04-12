# AWS CloudTrail — Deep Dive

---

## WHAT IS CLOUDTRAIL

```
CloudTrail = records EVERY API call made in your AWS account

WHO did WHAT, WHEN, and FROM WHERE

┌─── CloudTrail Event ─────────────────────────────────────────┐
│                                                               │
│  Who:     IAM user "dev-1" (arn:aws:iam::111:user/dev-1)    │
│  What:    ec2:TerminateInstances                             │
│  When:    2024-01-15T10:30:00Z                               │
│  Where:   ca-central-1                                       │
│  From IP: 203.0.113.50                                       │
│  Status:  Success                                            │
│  Target:  i-0abc123def456                                    │
│                                                               │
└───────────────────────────────────────────────────────────────┘

CloudWatch = HOW is my system performing? (metrics, logs, alarms)
CloudTrail = WHO did WHAT in my account? (audit, security)
```

---

## EVENT TYPES

```
┌─── 1. Management Events (Control Plane) ─────────────────────┐
│                                                               │
│  Operations that manage AWS resources:                       │
│  - CreateBucket, DeleteBucket                                │
│  - RunInstances, TerminateInstances                          │
│  - CreateUser, AttachRolePolicy                              │
│  - CreateVPC, ModifySecurityGroup                            │
│                                                               │
│  Logged by DEFAULT (free for 90 days in Event History)       │
│  Read events: Describe*, List*, Get*                         │
│  Write events: Create*, Delete*, Put*, Update*               │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Data Events (Data Plane) ──────────────────────────────┐
│                                                               │
│  Operations ON the data inside resources:                    │
│  - S3: GetObject, PutObject, DeleteObject                    │
│  - Lambda: Invoke                                            │
│  - DynamoDB: GetItem, PutItem                                │
│                                                               │
│  NOT logged by default (high volume, extra cost)             │
│  Must enable explicitly per resource                         │
└───────────────────────────────────────────────────────────────┘

┌─── 3. Insights Events ──────────────────────────────────────┐
│                                                               │
│  Detects UNUSUAL activity automatically:                     │
│  - Spike in TerminateInstances calls                         │
│  - Burst of failed API calls (unauthorized)                  │
│  - Unusual resource provisioning                             │
│                                                               │
│  Extra cost, must enable                                     │
└───────────────────────────────────────────────────────────────┘
```

---

## TRAIL CONFIGURATION

```
┌─── Trail ────────────────────────────────────────────────────┐
│                                                               │
│  Trail = configuration that delivers events to S3/CloudWatch │
│                                                               │
│  Options:                                                    │
│  ┌────────────────────────┬──────────────────────────────┐  │
│  │ Single-region trail    │ Events from one region only   │  │
│  │ Multi-region trail     │ Events from ALL regions (✓)   │  │
│  │ Organization trail     │ All accounts in AWS Org       │  │
│  └────────────────────────┴──────────────────────────────┘  │
│                                                               │
│  Destinations:                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  CloudTrail ──► S3 Bucket (long-term storage)        │   │
│  │             ──► CloudWatch Logs (real-time alerting)  │   │
│  │             ──► EventBridge (trigger automation)      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
│  Best practice: multi-region trail → S3 + CloudWatch Logs   │
└───────────────────────────────────────────────────────────────┘
```

---

## SECURITY & INTEGRITY

```
Log File Integrity Validation:
- CloudTrail creates a hash (digest) for each log file
- You can verify logs haven't been tampered with
- Enable this for compliance

S3 Bucket Protection:
- Enable S3 versioning (prevent deletion)
- Enable MFA Delete (require MFA to delete logs)
- Bucket policy: deny delete for everyone except admin
- Enable SSE-KMS encryption
- Enable S3 Object Lock (WORM — write once, read many)
```

---

## COMMON USE CASES

```
1. Security Investigation:
   "Who deleted the production S3 bucket?"
   → Filter: eventName=DeleteBucket, resourceName=prod-bucket

2. Compliance Auditing:
   "Show all IAM changes in the last 30 days"
   → Filter: eventSource=iam.amazonaws.com

3. Unauthorized Access Detection:
   CloudTrail → CloudWatch Logs → Metric Filter:
   "Count events where errorCode=UnauthorizedAccess"
   → Alarm if > 5 in 5 minutes → SNS alert

4. Root Account Usage Alert:
   CloudTrail → EventBridge rule:
   "If userIdentity.type = Root" → SNS notification
   (Root should NEVER be used for daily work)

5. Cost Investigation:
   "Who launched those 50 m5.4xlarge instances?"
   → Filter: eventName=RunInstances, instanceType=m5.4xlarge
```

---

## CLOUDTRAIL vs CONFIG vs CLOUDWATCH

```
┌──────────────────┬──────────────────┬──────────────────────┐
│ CloudTrail       │ AWS Config       │ CloudWatch           │
├──────────────────┼──────────────────┼──────────────────────┤
│ WHO did WHAT     │ WHAT changed     │ HOW is it performing │
│ API audit log    │ Resource config  │ Metrics & logs       │
│                  │ history          │                      │
│ "dev-1 deleted   │ "SG rule changed │ "CPU is at 95%"     │
│  the bucket"     │  from X to Y"    │                      │
│                  │                  │                      │
│ Security/audit   │ Compliance/drift │ Operations/alerting  │
└──────────────────┴──────────────────┴──────────────────────┘
```
