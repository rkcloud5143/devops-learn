# AWS Kinesis — Deep Dive

---

## WHAT IS KINESIS

```
Kinesis = Real-time streaming data service

Think of it as a FIREHOSE of data flowing continuously:
- Application logs (thousands per second)
- IoT sensor data
- Clickstream data (user activity)
- Financial transactions
- Social media feeds

SQS = individual messages, processed one at a time
Kinesis = continuous stream of data, processed in real-time
```

---

## KINESIS SERVICES

```
┌─── 1. Kinesis Data Streams ──────────────────────────────────┐
│                                                               │
│  Producers ──► [Shard 1] ──► Consumers                       │
│            ──► [Shard 2] ──► (Lambda, EC2, KDA)              │
│            ──► [Shard 3] ──►                                 │
│                                                               │
│  - Real-time (~200ms latency)                                │
│  - Data retained 24 hours (up to 365 days)                   │
│  - Multiple consumers can read same data                     │
│  - You manage capacity (number of shards)                    │
│  - Each shard: 1 MB/sec in, 2 MB/sec out                    │
│                                                               │
│  Shard = unit of capacity (like a lane on a highway)         │
│  More shards = more throughput                               │
│                                                               │
│  Modes:                                                      │
│  - Provisioned: you choose number of shards                  │
│  - On-demand: auto-scales (up to 200 MB/sec)                │
│                                                               │
│  Use for: real-time analytics, log processing, IoT           │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Kinesis Data Firehose ─────────────────────────────────┐
│                                                               │
│  Producers ──► Firehose ──► Destination                      │
│                    │                                          │
│                    ├──► S3 (most common)                      │
│                    ├──► Redshift (via S3)                     │
│                    ├──► OpenSearch                            │
│                    └──► HTTP endpoint / Splunk                │
│                                                               │
│  - Near real-time (60 second minimum buffer)                 │
│  - Fully managed (no shards to manage)                       │
│  - Auto-scales                                               │
│  - Can transform data with Lambda before delivery            │
│  - Pay per data volume                                       │
│                                                               │
│  Use for: loading streaming data into S3/Redshift/ES         │
└───────────────────────────────────────────────────────────────┘

┌─── 3. Kinesis Data Analytics ────────────────────────────────┐
│  Run SQL queries on streaming data in real-time              │
│  Input: Data Streams or Firehose                             │
│  Output: Data Streams, Firehose, Lambda                      │
│  Use for: real-time dashboards, anomaly detection            │
└──────────────────────────────────────────────────────────────┘

┌─── 4. Kinesis Video Streams ─────────────────────────────────┐
│  Stream video from devices to AWS                            │
│  Use for: security cameras, smart home, ML on video          │
└──────────────────────────────────────────────────────────────┘
```

---

## KINESIS vs SQS

```
┌──────────────────────┬──────────────────────────────┐
│ Kinesis Data Streams │ SQS                          │
├──────────────────────┼──────────────────────────────┤
│ Real-time streaming  │ Message queue                │
│ Multiple consumers   │ One consumer per message     │
│ Data replay (retain) │ Message deleted after consume│
│ Ordered per shard    │ Unordered (standard)         │
│ You manage shards    │ Fully managed                │
│ Higher throughput    │ Simpler to use               │
│ More expensive       │ Cheaper for most use cases   │
│                      │                              │
│ Use for: analytics,  │ Use for: task queues,        │
│ IoT, log aggregation │ decoupling, job processing   │
└──────────────────────┴──────────────────────────────┘
```
