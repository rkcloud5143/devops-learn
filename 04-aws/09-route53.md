# AWS Route 53 — Deep Dive

---

## DNS RECORD TYPES

```
┌──────────┬──────────────────────────────────────────────────────┐
│ Type     │ What it does                                        │
├──────────┼──────────────────────────────────────────────────────┤
│ A        │ Maps domain → IPv4 address                          │
│          │ example.com → 93.184.216.34                         │
├──────────┼──────────────────────────────────────────────────────┤
│ AAAA     │ Maps domain → IPv6 address                          │
│          │ example.com → 2606:2800:220:1:248:1893:25c8:1946    │
├──────────┼──────────────────────────────────────────────────────┤
│ CNAME    │ Maps domain → another domain                        │
│          │ www.example.com → example.com                       │
│          │ CANNOT be used for root domain (example.com)        │
├──────────┼──────────────────────────────────────────────────────┤
│ ALIAS    │ Maps domain → AWS resource (Route 53 specific)      │
│          │ example.com → d1234.cloudfront.net                  │
│          │ CAN be used for root domain (unlike CNAME)          │
│          │ Free of charge for queries                          │
│          │ Works with: ALB, CloudFront, S3, API Gateway        │
├──────────┼──────────────────────────────────────────────────────┤
│ MX       │ Mail server records                                 │
│          │ example.com → mail.example.com (priority 10)        │
├──────────┼──────────────────────────────────────────────────────┤
│ TXT      │ Text records (verification, SPF, DKIM)              │
│          │ example.com → "v=spf1 include:_spf.google.com ~all" │
├──────────┼──────────────────────────────────────────────────────┤
│ NS       │ Name server records (which DNS server is authority) │
└──────────┴──────────────────────────────────────────────────────┘

CNAME vs ALIAS (interview question!):
┌──────────────────────┬──────────────────────────────┐
│ CNAME                │ ALIAS                        │
├──────────────────────┼──────────────────────────────┤
│ Points to any domain │ Points to AWS resources only  │
│ NOT for root domain  │ Works for root domain         │
│ Charged for queries  │ Free queries                  │
│ Standard DNS         │ Route 53 extension            │
└──────────────────────┴──────────────────────────────┘
```

---

## ROUTING POLICIES

```
┌─── 1. Simple ────────────────────────────────────────────────┐
│  One record, one or more values                              │
│  If multiple values: client picks randomly                   │
│  No health checks                                            │
│  Use for: single resource                                    │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Weighted ──────────────────────────────────────────────┐
│  Split traffic by percentage                                 │
│                                                               │
│  www.myapp.com:                                              │
│    70% → EC2 instance A (v1)                                 │
│    20% → EC2 instance B (v2)                                 │
│    10% → EC2 instance C (v3)                                 │
│                                                               │
│  Use for: canary deployments, A/B testing, gradual migration │
└───────────────────────────────────────────────────────────────┘

┌─── 3. Latency-Based ────────────────────────────────────────┐
│  Route to region with lowest latency for the user            │
│                                                               │
│  User in Toronto → ca-central-1 (lowest latency)            │
│  User in London  → eu-west-1 (lowest latency)               │
│                                                               │
│  Use for: global applications, multi-region deployments      │
└──────────────────────────────────────────────────────────────┘

┌─── 4. Failover ─────────────────────────────────────────────┐
│  Active-passive setup                                        │
│                                                               │
│  Primary (healthy)  → route traffic here                     │
│  Primary (unhealthy)→ route to secondary (DR)                │
│                                                               │
│  ┌──────────┐  health check  ┌──────────┐                   │
│  │ Primary  │ ◄─────────────►│ Route 53 │                   │
│  │ (active) │    fails!      │          │                   │
│  └──────────┘                │ switches │                   │
│                              │ to ──────┤                   │
│  ┌──────────┐                │          │                   │
│  │Secondary │ ◄──────────────│          │                   │
│  │(passive) │                └──────────┘                   │
│  └──────────┘                                                │
│                                                               │
│  Use for: disaster recovery                                  │
└──────────────────────────────────────────────────────────────┘

┌─── 5. Geolocation ──────────────────────────────────────────┐
│  Route based on user's geographic location                   │
│                                                               │
│  Users in Canada  → Canadian servers (data residency)        │
│  Users in EU      → EU servers (GDPR compliance)             │
│  Default          → US servers                               │
│                                                               │
│  Use for: content localization, legal restrictions            │
└──────────────────────────────────────────────────────────────┘

┌─── 6. Multi-Value Answer ───────────────────────────────────┐
│  Returns multiple healthy IPs (up to 8)                      │
│  Client-side load balancing                                  │
│  Like simple routing but with health checks                  │
└──────────────────────────────────────────────────────────────┘
```

---

## PUBLIC vs PRIVATE HOSTED ZONES

```
┌─── Public Hosted Zone ───────────────────────────────────────┐
│                                                               │
│  Resolves domain names from the INTERNET                     │
│                                                               │
│  Internet users → Route 53 → "myapp.com = 54.23.1.100"      │
│                                                               │
│  - Anyone on the internet can query                          │
│  - Used for: public websites, APIs, external services        │
│  - You register domain (or transfer) in Route 53             │
│  - Cost: $0.50/month per hosted zone + $0.40 per 1M queries │
│                                                               │
│  Example records:                                            │
│  myapp.com         A     54.23.1.100                         │
│  www.myapp.com     CNAME myapp.com                           │
│  api.myapp.com     ALIAS my-alb-123.elb.amazonaws.com        │
└───────────────────────────────────────────────────────────────┘

┌─── Private Hosted Zone ──────────────────────────────────────┐
│                                                               │
│  Resolves domain names ONLY within your VPC(s)               │
│  NOT accessible from the internet                            │
│                                                               │
│  EC2 in VPC → Route 53 → "db.internal = 10.0.2.50"          │
│                                                               │
│  - Only instances in associated VPCs can query               │
│  - Used for: internal service discovery, private DNS         │
│  - Can associate with multiple VPCs (even cross-account)     │
│  - Cost: $0.50/month per hosted zone                         │
│                                                               │
│  Example records:                                            │
│  db.internal.myapp.com       A     10.0.2.50                 │
│  cache.internal.myapp.com    A     10.0.2.100                │
│  api.internal.myapp.com      A     10.0.1.25                 │
└───────────────────────────────────────────────────────────────┘

┌─── How They Work Together ───────────────────────────────────┐
│                                                               │
│  Internet User:                                              │
│  myapp.com → Public Zone → ALB public IP                     │
│                                                               │
│  EC2 Instance (inside VPC):                                  │
│  myapp.com → Public Zone → ALB public IP (same)              │
│  db.internal.myapp.com → Private Zone → 10.0.2.50           │
│                                                               │
│  Private zone takes priority for matching domains            │
│  within the VPC (split-horizon DNS)                          │
└───────────────────────────────────────────────────────────────┘

Setup Private Hosted Zone:
1. Create hosted zone → select "Private"
2. Associate with VPC(s)
3. Add records (A, CNAME, etc.)
4. EC2 instances in those VPCs can now resolve the names

Use case: microservices calling each other by name
  user-service → http://order-service.internal:3000/api/orders
  Instead of hardcoding IPs that change
```

---

## HEALTH CHECKS

```
Route 53 can monitor the health of your resources:

1. Endpoint health check:
   - Route 53 sends requests to your endpoint
   - HTTP, HTTPS, or TCP
   - Healthy if 2xx or 3xx response
   - Can check specific text in response body

2. Calculated health check:
   - Combines multiple health checks (AND, OR, NOT)
   - "Healthy if at least 2 of 3 endpoints are healthy"

3. CloudWatch alarm health check:
   - Healthy/unhealthy based on CloudWatch alarm state
   - For private resources (not publicly accessible)

Health checks are from Route 53 health checkers (global).
Your security group must allow inbound from Route 53 IPs.
```
