# AWS EBS & EFS (Storage Services) — Deep Dive

---

## EBS (Elastic Block Store)

```
EBS = virtual hard drive attached to EC2 over the network
Like plugging a USB drive into your server (but over network)

┌──────────┐         network         ┌──────────┐
│   EC2    │◄═══════════════════════►│   EBS    │
│ Instance │                         │  Volume  │
└──────────┘                         └──────────┘
```

### Key Rules
```
- One EBS volume → one EC2 instance (usually)
  (io1/io2 support multi-attach to up to 16 instances)
- Locked to ONE Availability Zone (can't attach cross-AZ)
- Persists after EC2 stop/terminate (if configured)
- Can resize on the fly (increase size, change type)
- Billed for provisioned size (not used size)
```

### Volume Types
```
┌─── SSD (for random I/O) ────────────────────────────────────┐
│                                                               │
│  gp3 (General Purpose SSD) ← DEFAULT, use this most often   │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  IOPS: 3,000 baseline (up to 16,000)                   │  │
│  │  Throughput: 125 MB/s (up to 1,000 MB/s)               │  │
│  │  Size: 1 GB - 16 TB                                    │  │
│  │  Cost: $0.08/GB/month                                   │  │
│  │  Use for: boot volumes, dev/test, small databases       │  │
│  │  IOPS and throughput can be set independently of size   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  gp2 (Previous Generation) ← legacy, prefer gp3             │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  IOPS: 3 per GB (min 100, max 16,000)                  │  │
│  │  Burst up to 3,000 IOPS for small volumes               │  │
│  │  Cost: $0.10/GB/month (more expensive than gp3!)        │  │
│  │  IOPS tied to volume size (bigger = more IOPS)          │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  io2 Block Express (Provisioned IOPS SSD) ← databases       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  IOPS: up to 256,000                                    │  │
│  │  Throughput: up to 4,000 MB/s                           │  │
│  │  Size: 4 GB - 64 TB                                     │  │
│  │  Cost: $0.125/GB + $0.065 per provisioned IOPS          │  │
│  │  99.999% durability (vs 99.9% for gp)                   │  │
│  │  Multi-attach supported                                  │  │
│  │  Use for: production databases, critical workloads       │  │
│  └────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘

┌─── HDD (for sequential I/O) ────────────────────────────────┐
│                                                               │
│  st1 (Throughput Optimized HDD)                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Throughput: up to 500 MB/s                             │  │
│  │  IOPS: up to 500                                        │  │
│  │  Cost: $0.045/GB/month                                  │  │
│  │  CANNOT be boot volume                                  │  │
│  │  Use for: big data, data warehouses, log processing     │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                               │
│  sc1 (Cold HDD)                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Throughput: up to 250 MB/s                             │  │
│  │  IOPS: up to 250                                        │  │
│  │  Cost: $0.015/GB/month (cheapest)                       │  │
│  │  CANNOT be boot volume                                  │  │
│  │  Use for: infrequent access, archival                   │  │
│  └────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘

Quick decision:
- Boot volume / general app → gp3
- Database needing high IOPS → io2
- Big data / streaming → st1
- Archive / rarely accessed → sc1
```

### EBS Snapshots
```
Snapshot = point-in-time backup of an EBS volume (stored in S3)

┌──────────┐   snapshot   ┌──────────┐   restore   ┌──────────┐
│ EBS Vol  │ ───────────► │ Snapshot │ ───────────► │ New EBS  │
│ (AZ-a)   │              │ (S3)     │              │ (any AZ) │
└──────────┘              └──────────┘              └──────────┘

Key points:
- Incremental (only changed blocks are saved)
- Can copy snapshots across regions (disaster recovery!)
- Can create AMI from snapshot
- Can share snapshots with other AWS accounts
- Snapshot Archive: 75% cheaper, 24-48 hour restore
- Recycle Bin: protect against accidental deletion (1 day - 1 year)

Commands:
aws ec2 create-snapshot --volume-id vol-xxx --description "backup"
aws ec2 copy-snapshot --source-region us-east-1 --source-snapshot-id snap-xxx
```

### EBS Encryption
```
- Uses KMS keys (AES-256)
- Encryption at rest + in transit (between EC2 and EBS)
- Snapshots of encrypted volumes are encrypted
- Volumes from encrypted snapshots are encrypted
- Minimal impact on latency
- To encrypt an unencrypted volume:
  1. Create snapshot
  2. Copy snapshot with encryption enabled
  3. Create new volume from encrypted snapshot
  4. Attach to EC2
```

---

## EFS (Elastic File System)

```
EFS = shared network file system (NFS) for multiple EC2 instances

┌──────────┐
│   EC2    │──┐
│  (AZ-a)  │  │
└──────────┘  │     ┌──────────────────┐
              ├────►│      EFS         │
┌──────────┐  │     │  (shared files)  │
│   EC2    │──┤     │                  │
│  (AZ-b)  │  │     │  /shared/app/    │
└──────────┘  │     │  /shared/config/ │
              │     │  /shared/uploads/ │
┌──────────┐  │     └──────────────────┘
│   EC2    │──┘
│  (AZ-c)  │        Multi-AZ by default
└──────────┘         Auto-scales (no provisioning!)
```

### EBS vs EFS vs Instance Store
```
┌──────────────────┬──────────────────┬──────────────────────┐
│ EBS              │ EFS              │ Instance Store       │
├──────────────────┼──────────────────┼──────────────────────┤
│ Block storage    │ File storage     │ Block storage        │
│ One EC2 (usually)│ Many EC2s        │ One EC2              │
│ One AZ           │ Multi-AZ         │ Physically attached  │
│ Provisioned size │ Auto-scales      │ Fixed size           │
│ Persists         │ Persists         │ LOST on stop/term    │
│ Fast (network)   │ Good (NFS)       │ Fastest (local SSD)  │
│ $0.08/GB (gp3)   │ $0.30/GB (std)   │ Free (included)      │
│                  │ Linux only       │                      │
│ Boot volumes ✓   │ Boot volumes ✗   │ Boot volumes ✗       │
│                  │                  │                      │
│ Use: databases,  │ Use: shared      │ Use: temp data,      │
│ boot, single app │ content, CMS,    │ cache, buffers       │
│                  │ home dirs        │                      │
└──────────────────┴──────────────────┴──────────────────────┘
```

### EFS Storage Classes
```
┌────────────────────────┬────────┬────────────────────────────┐
│ Class                  │ Cost   │ Use Case                   │
├────────────────────────┼────────┼────────────────────────────┤
│ Standard               │ $0.30  │ Frequently accessed files  │
│ Standard-IA            │ $0.025 │ Infrequent access          │
│ One Zone               │ $0.16  │ Single AZ (cheaper, no HA) │
│ One Zone-IA            │ $0.013 │ Single AZ + infrequent     │
└────────────────────────┴────────┴────────────────────────────┘

Lifecycle policy: auto-move files to IA after X days (like S3)
```

### EFS Performance
```
Performance modes:
- General Purpose: low latency (web serving, CMS) ← default
- Max I/O: higher latency, higher throughput (big data, media)

Throughput modes:
- Bursting: scales with storage size
- Provisioned: set throughput independently
- Elastic: auto-scales throughput (recommended)
```

---

## S3 vs EBS vs EFS (Interview Question)

```
"Where should I store this?"

User uploads (images, PDFs)     → S3 (object storage, unlimited)
Database files                  → EBS (block storage, single EC2)
Shared app config across EC2s   → EFS (file storage, multi-EC2)
Static website                  → S3 + CloudFront
Terraform state                 → S3 (with versioning + locking)
Docker images                   → ECR (container registry)
Secrets / passwords             → Secrets Manager or SSM
Log files (long-term)           → S3 (lifecycle to Glacier)
Temp/cache data                 → Instance Store (fastest, ephemeral)
```
