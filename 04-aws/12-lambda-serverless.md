# AWS Lambda & Serverless — Deep Dive

---

## LAMBDA

```
┌─── Lambda ───────────────────────────────────────────────────┐
│                                                               │
│  Run code WITHOUT managing servers                           │
│  Pay only when code runs (per request + per duration)        │
│  Auto-scales from 0 to thousands of concurrent executions    │
│                                                               │
│  Limits:                                                     │
│  - Timeout: max 15 minutes                                   │
│  - Memory: 128 MB to 10 GB                                   │
│  - Deployment package: 50 MB zipped, 250 MB unzipped         │
│  - Environment variables: 4 KB total                         │
│  - Concurrent executions: 1000 (default, can increase)       │
│  - /tmp storage: 512 MB to 10 GB                             │
│                                                               │
│  Supported runtimes:                                         │
│  Python, Node.js, Java, Go, .NET, Ruby, custom (container)  │
│                                                               │
│  Pricing:                                                    │
│  - First 1M requests/month: FREE                             │
│  - $0.20 per 1M requests after that                          │
│  - $0.0000166667 per GB-second of compute                    │
└───────────────────────────────────────────────────────────────┘
```

### Lambda Triggers (What Invokes Lambda)
```
┌─────────────────────────────────────────────────────────────┐
│  Synchronous (wait for response):                           │
│  - API Gateway → Lambda → response to client                │
│  - ALB → Lambda                                             │
│  - CloudFront (Lambda@Edge)                                 │
│                                                              │
│  Asynchronous (fire and forget):                            │
│  - S3 events (object created/deleted)                       │
│  - SNS notifications                                        │
│  - CloudWatch Events / EventBridge                          │
│  - SES (email)                                              │
│                                                              │
│  Stream-based (poll for records):                           │
│  - DynamoDB Streams                                         │
│  - SQS queues                                               │
│  - Kinesis streams                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## API GATEWAY

```
┌─── API Gateway ──────────────────────────────────────────────┐
│                                                               │
│  Create, publish, and manage REST/HTTP/WebSocket APIs        │
│                                                               │
│  Client → API Gateway → Lambda / EC2 / any HTTP endpoint    │
│                                                               │
│  Features:                                                   │
│  - Rate limiting & throttling                                │
│  - API keys & usage plans                                    │
│  - Request/response transformation                           │
│  - Caching (reduce Lambda invocations)                       │
│  - Custom domain names (api.myapp.com)                       │
│  - CORS support                                              │
│  - Stages (dev, staging, prod)                               │
│  - Canary deployments (10% to new version)                   │
│                                                               │
│  Types:                                                      │
│  ┌──────────────┬────────────────────────────────────────┐  │
│  │ REST API     │ Full featured, more expensive           │  │
│  │ HTTP API     │ Simpler, cheaper, faster (use this!)    │  │
│  │ WebSocket API│ Real-time two-way communication         │  │
│  └──────────────┴────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

---

## DYNAMODB

```
┌─── DynamoDB ─────────────────────────────────────────────────┐
│                                                               │
│  Fully managed NoSQL database                                │
│  Single-digit millisecond latency at any scale               │
│  Serverless (no instances to manage)                         │
│                                                               │
│  Data Model:                                                 │
│  ┌─── Table: Users ──────────────────────────────────────┐  │
│  │                                                        │  │
│  │  Partition Key (PK): user_id                          │  │
│  │  Sort Key (SK): optional, for range queries           │  │
│  │                                                        │  │
│  │  ┌──────────┬──────────┬───────┬──────────────────┐   │  │
│  │  │ user_id  │ name     │ email │ created_at       │   │  │
│  │  │ (PK)     │          │       │                  │   │  │
│  │  ├──────────┼──────────┼───────┼──────────────────┤   │  │
│  │  │ u-001    │ Alice    │ a@x   │ 2024-01-15       │   │  │
│  │  │ u-002    │ Bob      │ b@x   │ 2024-01-16       │   │  │
│  │  └──────────┴──────────┴───────┴──────────────────┘   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  Capacity Modes:                                             │
│  - On-Demand: pay per request (unpredictable traffic)        │
│  - Provisioned: set read/write capacity units (predictable)  │
│    - Can use auto-scaling                                    │
│                                                               │
│  Features:                                                   │
│  - DynamoDB Streams: capture changes (like a changelog)      │
│  - Global Tables: multi-region replication                   │
│  - DAX: in-memory cache (microsecond reads)                  │
│  - TTL: auto-delete expired items                            │
│  - Point-in-time recovery (35 days)                          │
│                                                               │
│  When to use DynamoDB vs RDS:                                │
│  ┌──────────────────────┬──────────────────────────────┐    │
│  │ DynamoDB             │ RDS                           │    │
│  ├──────────────────────┼──────────────────────────────┤    │
│  │ Key-value / document │ Relational (SQL, joins)       │    │
│  │ Serverless           │ Instance-based                │    │
│  │ Infinite scale       │ Vertical scaling (mostly)     │    │
│  │ Simple queries       │ Complex queries (joins, etc.) │    │
│  │ Session store, IoT   │ Traditional apps, reporting   │    │
│  └──────────────────────┴──────────────────────────────┘    │
└───────────────────────────────────────────────────────────────┘
```

---

## SQS, SNS, EventBridge (Messaging)

```
┌─── SQS (Simple Queue Service) ───────────────────────────────┐
│                                                               │
│  Producer ──► [Queue] ──► Consumer                           │
│                                                               │
│  - Messages wait in queue until consumed                     │
│  - Consumer pulls messages (poll-based)                      │
│  - Message deleted after processing                          │
│  - Decouples services (producer doesn't wait for consumer)   │
│                                                               │
│  Types:                                                      │
│  - Standard: unlimited throughput, at-least-once, unordered  │
│  - FIFO: 300 msg/sec, exactly-once, ordered                 │
│                                                               │
│  Use for: order processing, task queues, buffering           │
└───────────────────────────────────────────────────────────────┘

┌─── SNS (Simple Notification Service) ────────────────────────┐
│                                                               │
│  Publisher ──► [Topic] ──► Subscriber 1 (email)              │
│                        ──► Subscriber 2 (Lambda)             │
│                        ──► Subscriber 3 (SQS)               │
│                                                               │
│  - Push-based (sends to all subscribers immediately)         │
│  - Fan-out pattern: one message → many receivers             │
│                                                               │
│  Use for: alerts, notifications, fan-out to multiple queues  │
└───────────────────────────────────────────────────────────────┘

┌─── EventBridge ──────────────────────────────────────────────┐
│                                                               │
│  Event Bus: routes events based on rules                     │
│                                                               │
│  Source (EC2 state change) ──► Rule (match pattern)          │
│                               ──► Target (Lambda, SQS, SNS) │
│                                                               │
│  - Cron/scheduled events ("run every 5 minutes")             │
│  - AWS service events ("EC2 instance terminated")            │
│  - Custom events from your applications                      │
│  - Third-party SaaS events (Zendesk, Datadog)               │
│                                                               │
│  Use for: event-driven architectures, automation             │
└───────────────────────────────────────────────────────────────┘

SQS vs SNS:
SQS = one consumer processes each message (queue)
SNS = all subscribers get every message (broadcast)
Often used together: SNS → multiple SQS queues (fan-out)
```
