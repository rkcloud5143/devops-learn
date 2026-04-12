# AWS RDS — Deep Dive

---

## SUPPORTED ENGINES

```
┌──────────────────┬──────────────────────────────────────────┐
│ Engine           │ Notes                                    │
├──────────────────┼──────────────────────────────────────────┤
│ PostgreSQL       │ Most popular for new projects            │
│ MySQL            │ Most widely used globally                │
│ MariaDB          │ MySQL fork, open source                  │
│ Oracle           │ Enterprise, bring your own license       │
│ SQL Server       │ Microsoft, Windows workloads             │
│ Aurora           │ AWS-built, MySQL/Postgres compatible     │
│                  │ 5x faster MySQL, 3x faster Postgres     │
│                  │ More expensive but auto-scales storage   │
└──────────────────┴──────────────────────────────────────────┘
```

---

## HIGH AVAILABILITY

### Multi-AZ (Failover)
```
┌─── Multi-AZ Deployment ─────────────────────────────────────┐
│                                                               │
│  AZ-a                          AZ-b                          │
│  ┌──────────────────┐         ┌──────────────────┐          │
│  │  RDS Primary     │ ──sync──► RDS Standby      │          │
│  │  (read + write)  │ replica │ (NOT accessible)  │          │
│  └──────────────────┘         └──────────────────┘          │
│         ▲                            │                       │
│         │                            │ automatic failover    │
│    App connects                      │ (1-2 minutes)        │
│    to endpoint                       │                       │
│                                      ▼                       │
│                              Standby becomes Primary         │
│                              DNS endpoint auto-updates       │
│                                                               │
│  - Synchronous replication (zero data loss)                  │
│  - Automatic failover (no manual intervention)               │
│  - Same endpoint URL (app doesn't need to change)            │
│  - Standby is NOT for read traffic                           │
│  - Use for: production databases                             │
└───────────────────────────────────────────────────────────────┘
```

### Read Replicas (Performance)
```
┌─── Read Replicas ────────────────────────────────────────────┐
│                                                               │
│  ┌──────────────────┐                                        │
│  │  RDS Primary     │──── async ────► Read Replica 1         │
│  │  (read + write)  │──── async ────► Read Replica 2         │
│  │                  │──── async ────► Read Replica 3         │
│  └──────────────────┘                (up to 15 for Aurora)   │
│         ▲                                    ▲               │
│         │                                    │               │
│    App WRITES here                    App READS here         │
│    (insert, update, delete)           (select queries)       │
│                                                               │
│  - Asynchronous replication (slight lag)                     │
│  - Each replica has its own endpoint                         │
│  - Can be in different AZ or different REGION                │
│  - Can be promoted to standalone DB (disaster recovery)      │
│  - No charge for replication within same region              │
│  - Use for: read-heavy workloads, reporting, analytics       │
└───────────────────────────────────────────────────────────────┘

Multi-AZ vs Read Replica:
┌──────────────────────┬──────────────────────────────┐
│ Multi-AZ             │ Read Replica                  │
├──────────────────────┼──────────────────────────────┤
│ For failover (HA)    │ For read scaling (performance)│
│ Sync replication     │ Async replication             │
│ Same region only     │ Cross-region possible         │
│ Can't read from it   │ Can read from it              │
│ Auto failover        │ Manual promotion              │
│ One standby          │ Up to 5 (15 for Aurora)       │
└──────────────────────┴──────────────────────────────┘

You can have BOTH: Multi-AZ + Read Replicas
```

---

## BACKUPS & SNAPSHOTS

```
Automated Backups:
- Daily full backup during maintenance window
- Transaction logs every 5 minutes
- Retention: 1-35 days (default 7)
- Point-in-time restore (to any second within retention)
- Stored in S3 (you don't see it)

Manual Snapshots:
- Triggered by you (or automation)
- Kept until you delete them (no expiration)
- Can copy to another region
- Can share with other AWS accounts
- Can restore to a new RDS instance

Restore:
- Always creates a NEW RDS instance (new endpoint)
- Cannot restore to existing instance
- App needs to update connection string
```

---

## SECURITY

```
┌─── RDS Security ─────────────────────────────────────────────┐
│                                                               │
│  Network:                                                    │
│  - Deploy in private subnet (no public access)               │
│  - Security group controls who can connect                   │
│  - Only allow app servers' SG on port 5432/3306              │
│                                                               │
│  Encryption at rest:                                         │
│  - AES-256 using KMS                                         │
│  - Must be enabled at creation (can't encrypt existing DB)   │
│  - Snapshots of encrypted DB are encrypted                   │
│  - Read replicas of encrypted DB are encrypted               │
│                                                               │
│  Encryption in transit:                                      │
│  - SSL/TLS connections                                       │
│  - Force SSL with parameter group: rds.force_ssl = 1        │
│                                                               │
│  Authentication:                                             │
│  - Username/password (traditional)                           │
│  - IAM authentication (token-based, no password)             │
│  - Kerberos (Active Directory)                               │
│                                                               │
│  IAM Auth flow:                                              │
│  EC2 (with IAM role) → generates auth token → connects to RDS│
│  Token valid for 15 minutes, SSL required                    │
└───────────────────────────────────────────────────────────────┘
```

---

## AURORA (AWS Premium Database)

```
┌─── Aurora Architecture ──────────────────────────────────────┐
│                                                               │
│  ┌─── Shared Storage Volume ─────────────────────────────┐  │
│  │  Auto-scales: 10GB → 128TB                             │  │
│  │  6 copies across 3 AZs (self-healing)                  │  │
│  │  Continuous backup to S3                                │  │
│  └────────────────────────────────────────────────────────┘  │
│       ▲           ▲           ▲                              │
│       │           │           │                              │
│  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐                       │
│  │ Writer  │ │ Reader  │ │ Reader  │                       │
│  │Instance │ │Instance │ │Instance │  (up to 15 readers)   │
│  └─────────┘ └─────────┘ └─────────┘                       │
│       ▲           ▲                                          │
│       │           │                                          │
│  Writer       Reader                                        │
│  Endpoint     Endpoint (load balanced across all readers)   │
│                                                               │
│  Key differences from regular RDS:                           │
│  - Storage auto-scales (no provisioning)                     │
│  - 6 copies of data across 3 AZs                            │
│  - Up to 15 read replicas (vs 5 for RDS)                    │
│  - Failover in < 30 seconds                                 │
│  - 20% more expensive than RDS                              │
│  - Backtrack: rewind DB to any point without restore         │
│                                                               │
│  Aurora Serverless:                                          │
│  - Auto-scales compute (not just storage)                    │
│  - Pay per second of use                                     │
│  - Good for infrequent/unpredictable workloads               │
└───────────────────────────────────────────────────────────────┘
```
