# AWS CloudWatch — Deep Dive

---

## THREE PILLARS

```
┌─── CloudWatch ───────────────────────────────────────────────┐
│                                                               │
│  1. METRICS  → numbers over time (CPU, memory, requests)     │
│  2. LOGS     → text output from applications and services    │
│  3. ALARMS   → trigger actions when metrics cross thresholds │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## METRICS

```
Default EC2 Metrics (free, every 5 min):
┌──────────────────────────┬───────────────────────────────────┐
│ Metric                   │ What it measures                  │
├──────────────────────────┼───────────────────────────────────┤
│ CPUUtilization           │ CPU usage percentage              │
│ NetworkIn / NetworkOut   │ Network traffic bytes             │
│ DiskReadOps/DiskWriteOps │ Disk I/O operations               │
│ StatusCheckFailed        │ Instance or system health         │
└──────────────────────────┴───────────────────────────────────┘

NOT included by default (need CloudWatch Agent):
- Memory utilization (RAM)
- Disk space usage
- Custom application metrics

Detailed Monitoring: every 1 minute (costs extra)

Custom Metrics:
- You push your own metrics via API
- Example: active_users, queue_depth, order_count
- Resolution: 1 second to 5 minutes
```

---

## LOGS

```
┌─── CloudWatch Logs Architecture ─────────────────────────────┐
│                                                               │
│  Log Group: /aws/lambda/my-function                          │
│  └── Log Stream: 2024/01/15/[$LATEST]abc123                 │
│      └── Log Events: individual log lines with timestamps    │
│                                                               │
│  Sources:                                                    │
│  - EC2 instances (via CloudWatch Agent)                      │
│  - Lambda functions (automatic)                              │
│  - ECS containers (via awslogs driver)                       │
│  - API Gateway (access logs)                                 │
│  - VPC Flow Logs                                             │
│  - Route 53 (DNS query logs)                                 │
│  - CloudTrail (API audit logs)                               │
│                                                               │
│  Features:                                                   │
│  - Filter patterns: search for "ERROR" or specific text      │
│  - Metric filters: create metrics from log patterns          │
│    Example: count "ERROR" occurrences → alarm if > 10/min   │
│  - Log Insights: SQL-like query language for logs            │
│  - Export to S3 for long-term storage                        │
│  - Stream to Lambda or Elasticsearch for processing          │
│  - Retention: 1 day to 10 years (or never expire)           │
└───────────────────────────────────────────────────────────────┘

CloudWatch Agent (install on EC2):
- Collects system metrics (memory, disk)
- Collects log files from EC2
- Sends both to CloudWatch
- Configured via JSON file
```

---

## ALARMS

```
Alarm States:
┌──────────┐     threshold crossed     ┌──────────┐
│    OK    │ ─────────────────────────► │  ALARM   │
└──────────┘                            └──────────┘
     ▲                                       │
     │          threshold recovered          │
     └───────────────────────────────────────┘

                ┌──────────────────┐
                │ INSUFFICIENT_DATA│  (not enough data yet)
                └──────────────────┘

Alarm Actions:
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  Metric crosses threshold                                   │
│       │                                                      │
│       ▼                                                      │
│  ┌─── Alarm triggers ───────────────────────────────────┐   │
│  │                                                       │   │
│  │  → SNS notification (email, SMS, Slack via Lambda)   │   │
│  │  → Auto Scaling action (scale out/in)                │   │
│  │  → EC2 action (stop, terminate, reboot, recover)     │   │
│  │  → Lambda function                                   │   │
│  │  → Systems Manager action                            │   │
│  │                                                       │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  Example alarms:                                            │
│  - EC2 CPU > 80% for 5 minutes → SNS email + scale out     │
│  - ALB 5xx errors > 10 in 1 minute → SNS to Slack          │
│  - RDS free storage < 10GB → SNS email                      │
│  - Lambda errors > 5 in 5 minutes → SNS + incident          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## CLOUDTRAIL (Audit — Often Confused with CloudWatch)

```
CloudWatch = performance monitoring (metrics, logs, alarms)
CloudTrail = API audit trail (WHO did WHAT and WHEN)

┌─── CloudTrail ───────────────────────────────────────────────┐
│                                                               │
│  Records every API call made in your AWS account:            │
│                                                               │
│  WHO:   IAM user "dev-1"                                     │
│  WHAT:  ec2:TerminateInstances                               │
│  WHEN:  2024-01-15T10:30:00Z                                 │
│  WHERE: ca-central-1                                         │
│  FROM:  IP 203.0.113.50                                      │
│                                                               │
│  Use for:                                                    │
│  - Security investigation ("who deleted that S3 bucket?")    │
│  - Compliance auditing                                       │
│  - Detecting unauthorized access                             │
│                                                               │
│  Store trails in S3 bucket for long-term retention           │
│  Send to CloudWatch Logs for real-time alerting              │
└───────────────────────────────────────────────────────────────┘

Interview tip:
"How do you know who deleted an EC2 instance?"
→ Check CloudTrail logs for ec2:TerminateInstances API call
```
