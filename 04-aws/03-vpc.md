# AWS VPC — Deep Dive

---

## THE BIG PICTURE

```
VPC = Your own private network inside AWS
      Like having your own data center in the cloud

┌─── VPC: 10.0.0.0/16 (65,536 IPs) ─────────────────────────────────┐
│                                                                      │
│  ┌─── Availability Zone A ──────────────────────────────────────┐   │
│  │                                                               │   │
│  │  ┌─── Public Subnet: 10.0.1.0/24 ────────────────────────┐  │   │
│  │  │  - Has route to Internet Gateway                        │  │   │
│  │  │  - Instances get public IPs                             │  │   │
│  │  │  - For: ALB, NAT Gateway, bastion hosts                │  │   │
│  │  │  ┌──────┐  ┌──────┐  ┌───────────┐                    │  │   │
│  │  │  │ ALB  │  │ NAT  │  │ Bastion   │                    │  │   │
│  │  │  │      │  │ GW   │  │ Host (SSH)│                    │  │   │
│  │  │  └──────┘  └──────┘  └───────────┘                    │  │   │
│  │  └────────────────────────────────────────────────────────┘  │   │
│  │                                                               │   │
│  │  ┌─── Private Subnet: 10.0.2.0/24 ───────────────────────┐  │   │
│  │  │  - NO direct internet access                            │  │   │
│  │  │  - Outbound via NAT Gateway                             │  │   │
│  │  │  - For: App servers, databases, internal services       │  │   │
│  │  │  ┌──────┐  ┌──────┐  ┌──────┐                         │  │   │
│  │  │  │ EC2  │  │ EC2  │  │ RDS  │                         │  │   │
│  │  │  │ App  │  │ App  │  │  DB  │                         │  │   │
│  │  │  └──────┘  └──────┘  └──────┘                         │  │   │
│  │  └────────────────────────────────────────────────────────┘  │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌─── Availability Zone B (same structure for HA) ──────────────┐   │
│  │  Public Subnet: 10.0.3.0/24                                   │   │
│  │  Private Subnet: 10.0.4.0/24                                  │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## VPC COMPONENTS — One by One

### 1. Internet Gateway (IGW)
```
┌──────────────────────────────────────────────────┐
│  Internet Gateway                                 │
│                                                   │
│  - Connects VPC to the internet                  │
│  - One per VPC                                   │
│  - Horizontally scaled, redundant, HA by default │
│  - No bandwidth constraints                      │
│                                                   │
│  Internet ◄──────► IGW ◄──────► VPC              │
│                                                   │
│  Without IGW: your VPC is completely isolated    │
└──────────────────────────────────────────────────┘
```

### 2. NAT Gateway vs NAT Instance
```
Private subnet instances need software updates but
should NOT be directly accessible from internet.

Solution: NAT Gateway (Network Address Translation)

┌──────────┐     ┌───────────┐     ┌─────────┐
│ Private  │────►│    NAT    │────►│   IGW   │────► Internet
│ Instance │     │  Gateway  │     │         │
└──────────┘     │(public    │     └─────────┘
                 │ subnet)   │
                 └───────────┘
                 
Outbound: ✅ Private instance CAN reach internet
Inbound:  ❌ Internet CANNOT reach private instance

┌──────────────────────┬──────────────────────────────┐
│   NAT Gateway        │   NAT Instance               │
├──────────────────────┼──────────────────────────────┤
│ AWS managed          │ You manage (EC2 instance)    │
│ Highly available     │ Single point of failure      │
│ Auto-scales          │ Limited by instance type     │
│ ~$32/month + data    │ Cheaper (t3.micro)           │
│ No maintenance       │ You patch & maintain         │
│ USE THIS ✅          │ Only for cost savings        │
└──────────────────────┴──────────────────────────────┘
```

### 3. Route Tables
```
Every subnet is associated with a route table.
Route table = rules for where network traffic goes.

Public Subnet Route Table:
┌─────────────────────┬──────────────────────┐
│ Destination         │ Target               │
├─────────────────────┼──────────────────────┤
│ 10.0.0.0/16         │ local (within VPC)   │
│ 0.0.0.0/0           │ igw-xxxxx (internet) │
└─────────────────────┴──────────────────────┘

Private Subnet Route Table:
┌─────────────────────┬──────────────────────┐
│ Destination         │ Target               │
├─────────────────────┼──────────────────────┤
│ 10.0.0.0/16         │ local (within VPC)   │
│ 0.0.0.0/0           │ nat-xxxxx (NAT GW)   │
└─────────────────────┴──────────────────────┘

What makes a subnet "public" or "private"?
→ If its route table has 0.0.0.0/0 → IGW = PUBLIC
→ If its route table has 0.0.0.0/0 → NAT = PRIVATE
→ If no 0.0.0.0/0 route at all = ISOLATED
```

### 4. Security Groups (SG) — Instance-Level Firewall
```
┌─── Security Group ───────────────────────────────────────────┐
│                                                               │
│  Attached to: ENI (network interface) of EC2, RDS, ALB, etc.│
│  Level: Instance level                                       │
│  Rules: ALLOW only (no deny rules)                           │
│  Stateful: if inbound allowed, outbound auto-allowed         │
│  Default: all inbound DENIED, all outbound ALLOWED           │
│                                                               │
│  Web Server SG:                                              │
│  ┌──────────┬──────────┬───────────┬────────────────────┐   │
│  │ Type     │ Protocol │ Port      │ Source             │   │
│  ├──────────┼──────────┼───────────┼────────────────────┤   │
│  │ Inbound  │ TCP      │ 80        │ 0.0.0.0/0 (all)   │   │
│  │ Inbound  │ TCP      │ 443       │ 0.0.0.0/0 (all)   │   │
│  │ Inbound  │ TCP      │ 22        │ 10.0.0.0/16 (VPC) │   │
│  │ Outbound │ All      │ All       │ 0.0.0.0/0         │   │
│  └──────────┴──────────┴───────────┴────────────────────┘   │
│                                                               │
│  Database SG:                                                │
│  ┌──────────┬──────────┬───────────┬────────────────────┐   │
│  │ Inbound  │ TCP      │ 5432      │ sg-webserver (SG!) │   │
│  │ Outbound │ All      │ All       │ 0.0.0.0/0         │   │
│  └──────────┴──────────┴───────────┴────────────────────┘   │
│                                                               │
│  KEY: You can reference another SG as source!                │
│  This means "only allow traffic from instances in that SG"   │
│  This is the recommended pattern for multi-tier apps.        │
└───────────────────────────────────────────────────────────────┘
```

### 5. NACLs (Network Access Control Lists) — Subnet-Level Firewall
```
┌─── NACL ─────────────────────────────────────────────────────┐
│                                                               │
│  Attached to: Subnet                                         │
│  Level: Subnet level (applies to ALL instances in subnet)    │
│  Rules: ALLOW and DENY                                       │
│  Stateless: must define BOTH inbound AND outbound rules      │
│  Rules evaluated in NUMBER ORDER (lowest first)              │
│  Default NACL: allows all inbound and outbound               │
│                                                               │
│  Custom NACL Example:                                        │
│  ┌──────┬──────────┬──────────┬───────┬───────────┬────────┐│
│  │ Rule │ Type     │ Protocol │ Port  │ Source    │ Action ││
│  ├──────┼──────────┼──────────┼───────┼───────────┼────────┤│
│  │ 100  │ Inbound  │ TCP      │ 80    │ 0.0.0.0/0│ ALLOW  ││
│  │ 110  │ Inbound  │ TCP      │ 443   │ 0.0.0.0/0│ ALLOW  ││
│  │ 120  │ Inbound  │ TCP      │ 22    │ office IP│ ALLOW  ││
│  │ *    │ Inbound  │ All      │ All   │ 0.0.0.0/0│ DENY   ││
│  ├──────┼──────────┼──────────┼───────┼───────────┼────────┤│
│  │ 100  │ Outbound │ TCP      │ 80    │ 0.0.0.0/0│ ALLOW  ││
│  │ 110  │ Outbound │ TCP      │ 443   │ 0.0.0.0/0│ ALLOW  ││
│  │ 120  │ Outbound │ TCP      │1024-  │ 0.0.0.0/0│ ALLOW  ││
│  │      │          │          │65535  │          │        ││
│  │ *    │ Outbound │ All      │ All   │ 0.0.0.0/0│ DENY   ││
│  └──────┴──────────┴──────────┴───────┴───────────┴────────┘│
│                                                               │
│  Ephemeral ports (1024-65535): needed for return traffic     │
│  because NACLs are STATELESS                                 │
└───────────────────────────────────────────────────────────────┘
```

### 6. VPC Peering
```
Connect two VPCs privately (no internet involved)

┌─── VPC A: 10.0.0.0/16 ───┐     ┌─── VPC B: 172.16.0.0/16 ──┐
│                            │     │                             │
│  EC2 instances             │◄───►│  EC2 instances              │
│                            │ VPC │                             │
│                            │Peer │                             │
└────────────────────────────┘     └─────────────────────────────┘

Rules:
- CIDR blocks must NOT overlap
- NOT transitive (A↔B and B↔C does NOT mean A↔C)
- Can peer across regions and accounts
- Must update route tables in BOTH VPCs
```

### 7. VPC Endpoints (PrivateLink)
```
Access AWS services WITHOUT going through the internet

Without endpoint:
EC2 → NAT GW → IGW → Internet → S3

With endpoint:
EC2 → VPC Endpoint → S3 (stays within AWS network!)

Two types:
┌──────────────────────┬──────────────────────────────┐
│ Gateway Endpoint     │ Interface Endpoint            │
├──────────────────────┼──────────────────────────────┤
│ Free                 │ Costs money (~$7/month + data)│
│ S3 and DynamoDB only │ Most other AWS services       │
│ Route table entry    │ ENI in your subnet            │
│ USE THIS for S3/DDB  │ USE THIS for everything else  │
└──────────────────────┴──────────────────────────────┘
```

---

## VPC FLOW LOGS

```
Capture IP traffic going to/from network interfaces in VPC

VPC Flow Log → CloudWatch Logs or S3

Example log entry:
2 123456789012 eni-abc123 10.0.1.5 52.94.76.5 443 49152 6 25 20000 1620140661 1620140721 ACCEPT OK

Fields: version account-id eni src-ip dst-ip dst-port src-port protocol packets bytes start end action log-status

Use for:
- Troubleshooting connectivity issues
- Security monitoring (detect unusual traffic)
- Compliance auditing
```

---

## INTERVIEW SCENARIO: Design a VPC

```
"Design a VPC for a 3-tier web application"

Answer:
┌─── VPC: 10.0.0.0/16 ────────────────────────────────────────┐
│                                                               │
│  AZ-a                          AZ-b                          │
│  ┌─────────────────────┐      ┌─────────────────────┐       │
│  │ Public: 10.0.1.0/24 │      │ Public: 10.0.3.0/24 │       │
│  │ ALB, NAT GW         │      │ ALB, NAT GW         │       │
│  └─────────────────────┘      └─────────────────────┘       │
│  ┌─────────────────────┐      ┌─────────────────────┐       │
│  │ Private: 10.0.2.0/24│      │ Private: 10.0.4.0/24│       │
│  │ App servers (EC2)    │      │ App servers (EC2)    │       │
│  └─────────────────────┘      └─────────────────────┘       │
│  ┌─────────────────────┐      ┌─────────────────────┐       │
│  │ Private: 10.0.5.0/24│      │ Private: 10.0.6.0/24│       │
│  │ Database (RDS)       │      │ Database (RDS)       │       │
│  └─────────────────────┘      └─────────────────────┘       │
│                                                               │
│  Security Groups:                                            │
│  ALB SG:  inbound 80/443 from 0.0.0.0/0                    │
│  App SG:  inbound 3000 from ALB SG only                     │
│  DB SG:   inbound 5432 from App SG only                     │
│                                                               │
│  VPC Endpoint for S3 (gateway, free)                        │
│  Flow Logs enabled → S3 bucket                              │
└───────────────────────────────────────────────────────────────┘
```
