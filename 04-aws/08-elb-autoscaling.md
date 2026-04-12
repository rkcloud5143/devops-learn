# AWS ELB + Auto Scaling — Deep Dive

---

## ELASTIC LOAD BALANCER (ELB)

### Three Types
```
┌─── ALB (Application Load Balancer) — Layer 7 ───────────────┐
│                                                               │
│  Works at: HTTP/HTTPS level                                  │
│  Can route based on:                                         │
│  - URL path:    /api/* → API servers, /web/* → web servers   │
│  - Hostname:    api.myapp.com → API, www.myapp.com → web     │
│  - Query string: ?platform=mobile → mobile servers           │
│  - HTTP headers                                              │
│                                                               │
│  Features:                                                   │
│  - SSL/TLS termination                                       │
│  - Sticky sessions (cookie-based)                            │
│  - WebSocket support                                         │
│  - HTTP/2 support                                            │
│  - Authentication (Cognito, OIDC)                            │
│  - WAF integration                                           │
│                                                               │
│  Target types: instances, IPs, Lambda functions              │
│  THIS IS THE ONE YOU'LL USE 90% OF THE TIME                  │
└───────────────────────────────────────────────────────────────┘

┌─── NLB (Network Load Balancer) — Layer 4 ────────────────────┐
│                                                               │
│  Works at: TCP/UDP level                                     │
│  Millions of requests per second                             │
│  Ultra-low latency (~100ms vs ~400ms for ALB)                │
│  Static IP per AZ (or Elastic IP)                            │
│                                                               │
│  Use for: gaming, IoT, real-time streaming, non-HTTP         │
│  Target types: instances, IPs, ALB (NLB in front of ALB)    │
└───────────────────────────────────────────────────────────────┘

┌─── CLB (Classic Load Balancer) — Legacy ─────────────────────┐
│  Layer 4 + Layer 7 (limited)                                 │
│  DO NOT use for new projects                                 │
│  Only exists for backward compatibility                      │
└───────────────────────────────────────────────────────────────┘
```

### ALB Architecture
```
                    Internet
                       │
              ┌────────▼────────┐
              │       ALB       │
              │  Listener: 443  │ ← HTTPS
              │  Listener: 80   │ ← HTTP (redirect to 443)
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │  Listener Rules │
              │                 │
              │  /api/*  → TG1  │
              │  /web/*  → TG2  │
              │  default → TG3  │
              └───┬─────────┬───┘
                  │         │
         ┌────────▼──┐  ┌───▼────────┐
         │ Target    │  │ Target     │
         │ Group 1   │  │ Group 2    │
         │ (API)     │  │ (Web)      │
         │           │  │            │
         │ ┌──┐ ┌──┐│  │ ┌──┐ ┌──┐ │
         │ │EC│ │EC││  │ │EC│ │EC│ │
         │ └──┘ └──┘│  │ └──┘ └──┘ │
         └──────────┘  └───────────┘

Target Group:
- Group of targets (EC2, IP, Lambda)
- Has its own health check settings
- ALB routes to healthy targets only
```

### Health Checks
```
ALB sends health check requests to targets:

GET /health HTTP/1.1    → Target responds 200 OK = healthy
                        → Target responds 5xx   = unhealthy

Settings:
- Path: /health (or /, /api/health)
- Interval: 30 seconds (how often to check)
- Timeout: 5 seconds (how long to wait)
- Healthy threshold: 3 (consecutive successes to mark healthy)
- Unhealthy threshold: 2 (consecutive failures to mark unhealthy)

Unhealthy targets are removed from rotation automatically.
```

### SSL/TLS on ALB
```
┌──────────┐  HTTPS   ┌──────┐  HTTP    ┌──────────┐
│  Client  │ ────────►│ ALB  │ ────────►│   EC2    │
│          │  (443)   │      │  (80)    │          │
└──────────┘          └──────┘          └──────────┘

- SSL certificate managed by ACM (AWS Certificate Manager)
- ACM provides FREE public SSL certificates
- ALB terminates SSL (decrypts) and forwards HTTP to targets
- This offloads encryption work from your EC2 instances
```

---

## AUTO SCALING GROUP (ASG)

### How It Works
```
┌─── Auto Scaling Group ───────────────────────────────────────┐
│                                                               │
│  Launch Template:                                            │
│  - AMI ID                                                    │
│  - Instance type (t3.medium)                                 │
│  - Security groups                                           │
│  - Key pair                                                  │
│  - User data (bootstrap script)                              │
│  - IAM role                                                  │
│                                                               │
│  Capacity Settings:                                          │
│  - Minimum: 2    (never go below this)                       │
│  - Desired: 2    (current target)                            │
│  - Maximum: 6    (never go above this)                       │
│                                                               │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    │
│  │ EC2  │ │ EC2  │ │      │ │      │ │      │ │      │    │
│  │  ✓   │ │  ✓   │ │      │ │      │ │      │ │      │    │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘    │
│  min=2     desired=2                            max=6       │
│                                                               │
│  Scale OUT: CPU > 70% → add instances (up to max)           │
│  Scale IN:  CPU < 30% → remove instances (down to min)      │
└───────────────────────────────────────────────────────────────┘
```

### Scaling Policies
```
┌─── 1. Target Tracking (recommended) ────────────────────────┐
│  "Keep average CPU at 50%"                                   │
│  ASG automatically adds/removes instances to maintain target │
│  Simplest to configure                                       │
│                                                               │
│  Common targets:                                             │
│  - Average CPU utilization: 50%                              │
│  - Request count per target: 1000                            │
│  - Average network in/out                                    │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Step Scaling ─────────────────────────────────────────┐
│  "If CPU > 70%, add 2 instances"                            │
│  "If CPU > 90%, add 4 instances"                            │
│  "If CPU < 30%, remove 1 instance"                          │
│  More control, more complex                                  │
└──────────────────────────────────────────────────────────────┘

┌─── 3. Scheduled Scaling ────────────────────────────────────┐
│  "At 9 AM, set desired to 10"                               │
│  "At 6 PM, set desired to 2"                                │
│  For predictable traffic patterns                            │
└──────────────────────────────────────────────────────────────┘

┌─── 4. Predictive Scaling ───────────────────────────────────┐
│  ML-based, analyzes historical patterns                      │
│  Pre-scales before traffic spike                             │
│  Good for recurring patterns                                 │
└──────────────────────────────────────────────────────────────┘

Cooldown Period:
- After scaling, wait X seconds before next scaling action
- Default: 300 seconds (5 minutes)
- Prevents rapid scale up/down (thrashing)
```

### ASG + ALB Together
```
                    Internet
                       │
              ┌────────▼────────┐
              │       ALB       │
              │  Health checks  │──── unhealthy? → ASG replaces it
              └────────┬────────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │   EC2    │ │   EC2    │ │   EC2    │
    │  (AZ-a)  │ │  (AZ-b)  │ │  (AZ-a)  │
    └──────────┘ └──────────┘ └──────────┘
    └──────── Auto Scaling Group ─────────┘

ASG uses ALB health checks (not just EC2 status checks)
If ALB marks a target unhealthy → ASG terminates and replaces it
```
