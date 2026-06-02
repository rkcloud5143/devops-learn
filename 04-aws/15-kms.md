# AWS KMS (Key Management Service) — Deep Dive

---

## WHAT IS KMS

```
KMS = Centralized service to create and manage encryption keys

┌─── KMS ──────────────────────────────────────────────────────┐
│                                                               │
│  You create a KEY → use it to encrypt/decrypt data           │
│  AWS manages the key storage and rotation                    │
│  You control WHO can use the key (via key policy)            │
│                                                               │
│  Used by: S3, EBS, RDS, Lambda, Secrets Manager, etc.       │
│  Almost every AWS encryption feature uses KMS behind scenes  │
└───────────────────────────────────────────────────────────────┘
```

---

## KEY TYPES

```
┌─── 1. AWS Managed Keys ─────────────────────────────────────┐
│  Created by: AWS (automatically)                             │
│  Name format: aws/s3, aws/ebs, aws/rds                      │
│  Rotation: automatic every year                              │
│  Cost: free                                                  │
│  Control: you CANNOT manage or delete them                   │
│  Use for: default encryption (SSE-S3, EBS default)          │
└──────────────────────────────────────────────────────────────┘

┌─── 2. Customer Managed Keys (CMK) ──────────────────────────┐
│  Created by: YOU                                             │
│  Rotation: optional (enable yearly auto-rotation)            │
│  Cost: $1/month per key + $0.03 per 10,000 API calls        │
│  Control: full control (create, disable, delete, policy)     │
│  Use for: production workloads, compliance, audit trail      │
│                                                               │
│  Key Policy (who can use this key):                          │
│  {                                                            │
│    "Effect": "Allow",                                        │
│    "Principal": {"AWS": "arn:aws:iam::111:role/AppRole"},    │
│    "Action": ["kms:Decrypt", "kms:GenerateDataKey"],         │
│    "Resource": "*"                                           │
│  }                                                            │
└──────────────────────────────────────────────────────────────┘

┌─── 3. AWS Owned Keys ───────────────────────────────────────┐
│  Created by: AWS for internal use                            │
│  You CANNOT view, manage, or audit them                      │
│  Free, used by some services internally                      │
└──────────────────────────────────────────────────────────────┘

┌─── 4. Imported Keys ────────────────────────────────────────┐
│  Created by: YOU (outside AWS)                               │
│  You import your own key material into KMS                   │
│  You manage rotation manually                                │
│  Use for: regulatory requirements to control key material    │
└──────────────────────────────────────────────────────────────┘
```

---

## ENVELOPE ENCRYPTION (How KMS Actually Works)

```
Problem: KMS can only encrypt up to 4KB of data directly
Solution: Envelope encryption

┌─── Step 1: Generate Data Key ────────────────────────────────┐
│                                                               │
│  Your App ──► KMS: "GenerateDataKey"                         │
│                                                               │
│  KMS returns:                                                │
│  ┌──────────────────┐  ┌──────────────────────┐             │
│  │ Plaintext        │  │ Encrypted            │             │
│  │ Data Key         │  │ Data Key             │             │
│  │ (use to encrypt) │  │ (store alongside     │             │
│  │                  │  │  encrypted data)     │             │
│  └──────────────────┘  └──────────────────────┘             │
└──────────────────────────────────────────────────────────────┘

┌─── Step 2: Encrypt Your Data ────────────────────────────────┐
│                                                               │
│  Plaintext Data Key + Your Data → Encrypted Data             │
│  Then DISCARD the plaintext data key from memory             │
│  Store: Encrypted Data + Encrypted Data Key together         │
└──────────────────────────────────────────────────────────────┘

┌─── Step 3: Decrypt ──────────────────────────────────────────┐
│                                                               │
│  Send Encrypted Data Key → KMS: "Decrypt"                    │
│  KMS returns Plaintext Data Key                              │
│  Use Plaintext Data Key to decrypt your data                 │
└──────────────────────────────────────────────────────────────┘

This is how S3, EBS, RDS encryption works under the hood.
```

---

## KMS vs SECRETS MANAGER vs SSM PARAMETER STORE

```
┌──────────────────────┬──────────────────┬──────────────────────┐
│ KMS                  │ Secrets Manager  │ SSM Parameter Store  │
├──────────────────────┼──────────────────┼──────────────────────┤
│ Encryption keys      │ Secrets (DB pass,│ Config values +      │
│                      │ API keys)        │ secrets              │
│ $1/key/month         │ $0.40/secret/mo  │ Free (standard)      │
│ Encrypt/decrypt data │ Auto-rotation    │ No auto-rotation     │
│                      │ of secrets       │ (for free tier)      │
│ Used BY other        │ Stores the       │ Stores config +      │
│ services             │ actual secret    │ secrets              │
│                      │ values           │                      │
└──────────────────────┴──────────────────┴──────────────────────┘

Rule of thumb:
- DB passwords, API keys → Secrets Manager (auto-rotation)
- Config values, feature flags → SSM Parameter Store (free)
- Encryption of data → KMS
```
