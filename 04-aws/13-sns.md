# AWS SNS (Simple Notification Service) — Deep Dive

---

## HOW SNS WORKS

```
Publisher ──► Topic ──► Subscriber 1 (Email)
                   ──► Subscriber 2 (SMS)
                   ──► Subscriber 3 (Lambda)
                   ──► Subscriber 4 (SQS)
                   ──► Subscriber 5 (HTTP endpoint)

ONE message → delivered to ALL subscribers (fan-out)
Push-based: SNS pushes to subscribers immediately
```

---

## SNS COMPONENTS

```
┌─── Topic ────────────────────────────────────────────────────┐
│  A communication channel (like a mailing list)               │
│                                                               │
│  Types:                                                      │
│  ┌──────────────┬────────────────────────────────────────┐  │
│  │ Standard     │ Best effort ordering, at-least-once     │  │
│  │              │ Nearly unlimited throughput              │  │
│  │              │ Subscribers: SQS, Lambda, HTTP, email   │  │
│  ├──────────────┼────────────────────────────────────────┤  │
│  │ FIFO         │ Strict ordering, exactly-once           │  │
│  │              │ 300 msg/sec (or 10 MB/sec batched)      │  │
│  │              │ Subscribers: SQS FIFO only              │  │
│  └──────────────┴────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘

┌─── Subscribers ──────────────────────────────────────────────┐
│  - Email / Email-JSON                                        │
│  - SMS (text messages)                                       │
│  - HTTP/HTTPS endpoints                                      │
│  - SQS queues                                                │
│  - Lambda functions                                          │
│  - Kinesis Data Firehose                                     │
│  - Platform applications (mobile push: iOS, Android)         │
└───────────────────────────────────────────────────────────────┘
```

---

## FAN-OUT PATTERN (Very Common)

```
┌─── SNS + SQS Fan-Out ───────────────────────────────────────┐
│                                                               │
│  Order Service                                               │
│       │                                                       │
│       ▼                                                       │
│  ┌─── SNS Topic: order-placed ───────────────────────────┐  │
│  │                                                        │  │
│  │  ──► SQS: payment-queue    ──► Payment Service        │  │
│  │  ──► SQS: inventory-queue  ──► Inventory Service      │  │
│  │  ──► SQS: email-queue      ──► Email Service          │  │
│  │  ──► Lambda: analytics     ──► Write to DynamoDB      │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  Benefits:                                                   │
│  - Services are decoupled (don't know about each other)     │
│  - Adding new consumer = just add new SQS subscription      │
│  - Each SQS queue processes independently                    │
│  - If one service is slow, others aren't affected            │
└───────────────────────────────────────────────────────────────┘
```

---

## SNS + CLOUDWATCH ALARMS

```
CloudWatch Alarm ──► SNS Topic ──► Email (ops team)
                                ──► Lambda (auto-remediate)
                                ──► Slack (via Lambda/webhook)

This is the standard alerting pattern in AWS.
```

---

## MESSAGE FILTERING

```
Without filtering: every subscriber gets every message
With filtering: subscribers only get messages they care about

SNS Topic: order-events
├── Subscriber 1 (filter: {"status": ["placed"]})     → new orders only
├── Subscriber 2 (filter: {"status": ["cancelled"]})  → cancellations only
└── Subscriber 3 (no filter)                           → gets everything
```
