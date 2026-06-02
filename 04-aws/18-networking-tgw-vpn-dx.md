# AWS Networking — TGW, Direct Connect, VPN, Peering — Deep Dive

---

## THE BIG PICTURE

```
How do you connect networks in AWS?

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  VPC ◄──► VPC              = VPC Peering (simple, 1-to-1)       │
│  VPC ◄──► Many VPCs        = Transit Gateway (hub-and-spoke)    │
│  VPC ◄──► On-Premises      = VPN or Direct Connect              │
│  VPC ◄──► Internet         = Internet Gateway                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. VPC PEERING

```
┌─── VPC Peering ──────────────────────────────────────────────┐
│                                                               │
│  Direct private connection between TWO VPCs                  │
│                                                               │
│  ┌─── VPC A ───┐         ┌─── VPC B ───┐                    │
│  │ 10.0.0.0/16 │◄──────►│ 172.16.0.0/16│                    │
│  └─────────────┘ Peering └──────────────┘                    │
│                                                               │
│  Rules:                                                      │
│  ✓ Works across regions                                      │
│  ✓ Works across AWS accounts                                 │
│  ✓ Traffic stays on AWS backbone (private)                   │
│  ✗ CIDR blocks must NOT overlap                              │
│  ✗ NOT transitive: A↔B and B↔C does NOT mean A↔C            │
│  ✗ Must update route tables in BOTH VPCs                     │
│  ✗ One peering connection per VPC pair                       │
│                                                               │
│  Route table in VPC A:                                       │
│  ┌─────────────────┬──────────────────────┐                  │
│  │ 172.16.0.0/16   │ pcx-xxxxx (peering)  │                  │
│  └─────────────────┴──────────────────────┘                  │
│                                                               │
│  Good for: 2-3 VPCs, simple setups                           │
│  Bad for: many VPCs (N VPCs = N*(N-1)/2 peering connections) │
│                                                               │
│  10 VPCs = 45 peering connections! → Use Transit Gateway     │
└───────────────────────────────────────────────────────────────┘
```

---

## 2. TRANSIT GATEWAY (TGW)

```
┌─── Transit Gateway ─────────────────────────────────────────┐
│                                                               │
│  Hub-and-spoke model: ONE central hub connects everything    │
│                                                               │
│       ┌─── VPC A ───┐                                        │
│       │ 10.0.0.0/16 │──┐                                    │
│       └─────────────┘  │                                     │
│                         │                                     │
│       ┌─── VPC B ───┐  │  ┌──────────────────┐              │
│       │ 10.1.0.0/16 │──┼──│ Transit Gateway  │              │
│       └─────────────┘  │  │   (TGW)          │              │
│                         │  │                  │              │
│       ┌─── VPC C ───┐  │  │  Regional hub    │              │
│       │ 10.2.0.0/16 │──┘  │  Up to 5000 VPCs │              │
│       └─────────────┘     │                  │              │
│                            │                  │              │
│       ┌─── On-Prem ────┐  │                  │              │
│       │ Data Center     │──┤  VPN or Direct   │              │
│       └─────────────────┘  │  Connect         │              │
│                            └──────────────────┘              │
│                                                               │
│  Features:                                                   │
│  ✓ Transitive routing (A↔TGW↔B means A can reach B)        │
│  ✓ Supports VPN and Direct Connect attachments               │
│  ✓ Route tables for traffic control                          │
│  ✓ Cross-region peering (TGW in region A ↔ TGW in region B) │
│  ✓ Multicast support                                         │
│  ✓ Centralized network management                            │
│                                                               │
│  Cost: $0.05/hour per attachment + $0.02/GB data processed   │
│                                                               │
│  Use for: enterprise networks, many VPCs, hybrid cloud       │
└───────────────────────────────────────────────────────────────┘

TGW Route Tables:
┌─────────────────────────────────────────────────────────────┐
│  You can create multiple route tables for segmentation:     │
│                                                              │
│  Production RT:  VPC-A, VPC-B can talk to each other        │
│  Dev RT:         VPC-C, VPC-D can talk to each other        │
│  Shared RT:      VPC-Shared accessible from all             │
│                                                              │
│  This isolates prod from dev at the network level           │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. AWS VPN (Site-to-Site)

```
┌─── Site-to-Site VPN ─────────────────────────────────────────┐
│                                                               │
│  Encrypted tunnel over the PUBLIC internet                   │
│                                                               │
│  On-Premises                              AWS                │
│  ┌──────────────┐    IPsec tunnel    ┌──────────────┐       │
│  │ Customer     │◄═══════════════════►│ Virtual      │       │
│  │ Gateway      │  (encrypted)       │ Private      │       │
│  │ (your router)│                    │ Gateway (VGW)│       │
│  └──────────────┘                    └──────┬───────┘       │
│                                              │               │
│                                        ┌─────▼─────┐        │
│                                        │    VPC    │        │
│                                        └───────────┘        │
│                                                               │
│  Setup:                                                      │
│  1. Create Virtual Private Gateway (VGW) → attach to VPC    │
│  2. Create Customer Gateway (CGW) → your on-prem router IP  │
│  3. Create VPN Connection (2 tunnels for HA)                 │
│  4. Update route tables                                      │
│                                                               │
│  Specs:                                                      │
│  - Bandwidth: up to 1.25 Gbps per tunnel                    │
│  - 2 tunnels per connection (for redundancy)                 │
│  - Encrypted with IPsec                                      │
│  - Setup time: minutes                                       │
│  - Cost: $0.05/hour per connection                           │
│                                                               │
│  Limitations:                                                │
│  - Goes over internet (variable latency)                     │
│  - Bandwidth limited                                         │
│  - For higher needs → Direct Connect                         │
└───────────────────────────────────────────────────────────────┘

┌─── Client VPN ───────────────────────────────────────────────┐
│  For individual users (laptops) connecting to AWS VPC        │
│  OpenVPN-based                                               │
│  Use for: remote workers accessing private resources         │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. AWS DIRECT CONNECT (DX)

```
┌─── Direct Connect ───────────────────────────────────────────┐
│                                                               │
│  DEDICATED private connection (NOT over internet)            │
│                                                               │
│  On-Premises          DX Location           AWS              │
│  ┌──────────┐     ┌──────────────┐     ┌──────────┐        │
│  │ Your     │─────│ AWS Direct   │─────│ AWS      │        │
│  │ Data     │fiber│ Connect      │fiber│ Region   │        │
│  │ Center   │     │ Location     │     │          │        │
│  └──────────┘     │ (colocation) │     └──────────┘        │
│                    └──────────────┘                          │
│                                                               │
│  Connection speeds:                                          │
│  - Dedicated: 1 Gbps, 10 Gbps, 100 Gbps                    │
│  - Hosted: 50 Mbps to 10 Gbps (via partner)                 │
│                                                               │
│  Benefits:                                                   │
│  ✓ Consistent low latency (not over internet)               │
│  ✓ High bandwidth (up to 100 Gbps)                          │
│  ✓ Private connection (more secure)                          │
│  ✓ Reduced data transfer costs                               │
│                                                               │
│  Drawbacks:                                                  │
│  ✗ Setup time: weeks to months (physical fiber)              │
│  ✗ Expensive ($0.30/hour for 1 Gbps + data transfer)        │
│  ✗ NOT encrypted by default (add VPN on top for encryption)  │
│  ✗ Single point of failure (need 2 connections for HA)       │
│                                                               │
│  Virtual Interfaces (VIFs):                                  │
│  ┌──────────────┬────────────────────────────────────────┐  │
│  │ Public VIF   │ Access public AWS services (S3, DynamoDB)│  │
│  │ Private VIF  │ Access VPC resources (EC2, RDS)          │  │
│  │ Transit VIF  │ Access via Transit Gateway               │  │
│  └──────────────┴────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

---

## 5. AWS CLOUD WAN

```
┌─── Cloud WAN ────────────────────────────────────────────────┐
│                                                               │
│  Managed global wide-area network                            │
│  Connects VPCs, on-premises, and branch offices globally     │
│                                                               │
│  Think of it as: Transit Gateway but GLOBAL + managed        │
│                                                               │
│  ┌─── Core Network ─────────────────────────────────────┐   │
│  │                                                       │   │
│  │  ca-central-1 ◄──────► us-east-1 ◄──────► eu-west-1 │   │
│  │  ┌─────┐              ┌─────┐             ┌─────┐    │   │
│  │  │VPC-A│              │VPC-B│             │VPC-C│    │   │
│  │  └─────┘              └─────┘             └─────┘    │   │
│  │      │                                        │       │   │
│  │  ┌───┴────┐                              ┌────┴───┐  │   │
│  │  │On-Prem │                              │Branch  │  │   │
│  │  │Toronto │                              │London  │  │   │
│  │  └────────┘                              └────────┘  │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  Features:                                                   │
│  - Network policies (segment prod vs dev globally)           │
│  - Automated routing                                         │
│  - Dashboard for monitoring                                  │
│  - Supports VPN and Direct Connect attachments               │
│                                                               │
│  Use for: large enterprises with global presence             │
│  Most companies use TGW instead (simpler, cheaper)           │
└───────────────────────────────────────────────────────────────┘
```

---

## COMPARISON TABLE

```
┌──────────────────┬──────────┬───────────┬──────────┬──────────┐
│                  │ VPC      │ Transit   │ Site-to- │ Direct   │
│                  │ Peering  │ Gateway   │ Site VPN │ Connect  │
├──────────────────┼──────────┼───────────┼──────────┼──────────┤
│ Connects         │ 2 VPCs   │ Many VPCs │ On-prem  │ On-prem  │
│                  │          │ + on-prem │ to VPC   │ to VPC   │
│ Transitive       │ No       │ Yes       │ No       │ No       │
│ Encrypted        │ Yes(AWS) │ Yes(AWS)  │ Yes(IPsec│ No*      │
│ Bandwidth        │ No limit │ 50 Gbps   │ 1.25 Gbps│ 100 Gbps │
│ Latency          │ Low      │ Low       │ Variable │ Lowest   │
│ Setup time       │ Minutes  │ Minutes   │ Minutes  │ Weeks    │
│ Cost             │ Data only│ $$        │ $        │ $$$      │
│ Cross-region     │ Yes      │ Yes       │ Yes      │ Yes      │
│ Cross-account    │ Yes      │ Yes       │ Yes      │ Yes      │
└──────────────────┴──────────┴───────────┴──────────┴──────────┘

* Direct Connect + VPN = encrypted Direct Connect
```

---

## INTERVIEW SCENARIO

```
Q: "Your company has 20 VPCs and an on-premises data center.
    How do you connect them?"

A: Transit Gateway
   - Attach all 20 VPCs to TGW
   - Attach VPN or Direct Connect to TGW for on-premises
   - Use TGW route tables to segment prod/dev
   - If low latency needed for on-prem: Direct Connect
   - If quick setup needed: Site-to-Site VPN
   - For HA: use both (DX primary, VPN backup)

Q: "When would you use VPC Peering instead of TGW?"

A: When you only have 2-3 VPCs that need to communicate
   - Simpler setup
   - No per-hour charge (only data transfer)
   - Lower latency (direct connection)
```
