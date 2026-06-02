# Cloudflare — Complete Guide 🌐

DNS, CDN, WAF, Zero Trust, Workers — the full Cloudflare ecosystem.

---

## What is Cloudflare?

```
Cloudflare sits BETWEEN your users and your servers.

User → Cloudflare Edge (300+ cities) → Your Origin Server

Services:
  DNS          → Fastest DNS in the world (1.1.1.1)
  CDN          → Cache content at the edge
  WAF          → Block attacks before they reach you
  DDoS         → Absorb massive traffic floods
  Zero Trust   → Replace VPN with identity-based access
  Workers      → Run code at the edge (serverless)
  Tunnels      → Connect your infra without opening ports
```

---

## DNS

### How It Works
```
User types myapp.com
  │
  ▼
Cloudflare DNS resolves it
  │
  ├── Proxied (orange cloud ☁️) → Traffic goes THROUGH Cloudflare
  │   Your real IP is hidden. CDN, WAF, DDoS protection active.
  │
  └── DNS-only (grey cloud ☁) → Traffic goes DIRECTLY to your server
      Just DNS resolution. No protection.
```

### Record Types
```
A       → Points to IPv4 address
AAAA    → Points to IPv6 address
CNAME   → Alias to another domain (can be at root with Cloudflare!)
MX      → Mail server
TXT     → Text records (SPF, DKIM, verification)
SRV     → Service discovery
```

### DNS + AWS Integration
```
# Typical setup:
myapp.com → CNAME → ALB DNS name (xxx.us-west-1.elb.amazonaws.com)
api.myapp.com → CNAME → ALB or API Gateway
static.myapp.com → CNAME → CloudFront distribution (d123.cloudfront.net)
```

---

## CDN (Content Delivery Network)

### How Caching Works
```
First request:
  User (Tokyo) → Cloudflare Edge (Tokyo) → Origin (US) → Response
                  Edge caches the response

Second request:
  User (Tokyo) → Cloudflare Edge (Tokyo) → Cached! (no origin hit)
                  Response in <10ms
```

### Cache Rules
```
Default cached: Static files (JS, CSS, images, fonts)
Not cached: HTML, API responses (unless you configure it)

Control via headers:
  Cache-Control: public, max-age=31536000    → Cache for 1 year
  Cache-Control: no-store                     → Never cache
  Cache-Control: s-maxage=3600               → Edge cache 1 hour

Cloudflare Page Rules (legacy) or Cache Rules (new):
  - Cache everything for /static/*
  - Bypass cache for /api/*
  - Cache HTML for 5 minutes
```

### Purge Cache
```bash
# Purge everything
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache" \
  -H "Authorization: Bearer $CF_TOKEN" \
  -d '{"purge_everything":true}'

# Purge specific URLs
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache" \
  -H "Authorization: Bearer $CF_TOKEN" \
  -d '{"files":["https://myapp.com/style.css"]}'
```

---

## WAF (Web Application Firewall)

### Layers of Protection
```
Request arrives at Cloudflare:
  │
  ├── 1. DDoS mitigation (volumetric attacks blocked)
  ├── 2. IP reputation (known bad IPs blocked)
  ├── 3. Bot management (automated traffic filtered)
  ├── 4. WAF managed rules (OWASP top 10)
  │      - SQL injection
  │      - XSS (cross-site scripting)
  │      - Path traversal
  │      - Command injection
  ├── 5. WAF custom rules (your rules)
  │      - Block specific countries
  │      - Rate limit login page
  │      - Challenge suspicious requests
  └── 6. Rate limiting (too many requests = blocked)
```

### Custom WAF Rules
```
# Block all traffic from specific country
(ip.geoip.country eq "XX") → Block

# Rate limit login endpoint
URI Path contains "/login" AND Rate > 5 requests/10s → Challenge

# Block non-browser traffic to main site
(not http.request.headers["user-agent"] contains "Mozilla") → Block

# Allow only specific IPs to admin
URI Path contains "/admin" AND NOT (ip.src in {1.2.3.4 5.6.7.8}) → Block
```

---

## Zero Trust (Replace Your VPN)

### The Old Way (VPN)
```
Employee → VPN → Inside network → Access EVERYTHING
Problem: Once inside VPN, they can reach ALL internal resources
```

### The Cloudflare Way (Zero Trust)
```
Employee → WARP client → Cloudflare → Identity check → ONLY specific resource
Every request verified: Who are you? What device? What are you accessing?

Components:
  WARP Client     → Installed on device, creates tunnel to Cloudflare
  Access          → Identity-based access to web apps (replaces VPN for web)
  Gateway         → Network-level policies (firewall rules based on identity)
  Tunnels         → Connect your infra to Cloudflare without public IPs
  Device Posture  → Is the device compliant? (OS version, disk encryption, etc.)
```

### Gateway Network Policies
```
WHO (Identity):
  - AD groups (DBA team, Developers, DevOps)
  - Email addresses
  - Device posture (serial number, OS version)

WHAT (Destination):
  - IP ranges (10.0.0.0/8 = internal network)
  - Hostnames (db.internal.company.com)
  - Ports (3306 = MySQL, 5432 = PostgreSQL)

ACTION:
  - Allow
  - Block
  - Audit (log only)

Example rules:
  DBA-group + destination 172.16.8.0/24 + port 3306 → Allow
  Everyone + destination 10.0.0.0/8 → Block (default deny)
  DevOps-group + destination * + port 22 → Allow
```

### Cloudflare Tunnels (cloudflared)
```
Traditional: Open port 443 on server → Public internet can reach it
Tunnel: Server connects OUTBOUND to Cloudflare → No open ports

┌─── Your Server ─────┐          ┌─── Cloudflare ─────┐
│                      │          │                      │
│  cloudflared agent  ─┼── OUTBOUND ──▶  Tunnel endpoint │
│  (connects to CF)    │          │                      │
│  No inbound ports!   │          │  Only authenticated  │
│                      │          │  users reach through │
└──────────────────────┘          └──────────────────────┘

# Install cloudflared
brew install cloudflare/cloudflare/cloudflared

# Create tunnel
cloudflared tunnel create my-tunnel

# Route traffic
cloudflared tunnel route dns my-tunnel internal-app.mycompany.com

# Run tunnel (connects to CF)
cloudflared tunnel run my-tunnel
```

### Zero Trust as IaC (Terraform)
```hcl
# Manage gateway policies with Terraform
resource "cloudflare_teams_rule" "allow_dba_mysql" {
  account_id  = var.account_id
  name        = "Allow-DB-DBA-Prod-MySQL"
  description = "DBA team access to production MySQL"
  precedence  = 100
  action      = "allow"
  enabled     = true

  traffic = "net.dst.ip in {172.16.8.0/24} and net.dst.port == 3306"
  identity = "any(identity.groups.name[*] in {\"DBA-Team\"})"
}
```

---

## Workers (Serverless at the Edge)

```javascript
// Runs at 300+ Cloudflare locations, <1ms cold start
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    // A/B testing at the edge
    if (url.pathname === '/') {
      const variant = Math.random() < 0.5 ? 'a' : 'b';
      return fetch(`https://origin.com/variant-${variant}`);
    }
    
    return fetch(request);
  }
}
```

Use cases: A/B testing, auth at edge, URL rewrites, API caching, geolocation routing.

---

## Cloudflare API

```bash
# Verify token
curl "https://api.cloudflare.com/client/v4/user/tokens/verify" \
  -H "Authorization: Bearer $CF_TOKEN"

# List zones
curl "https://api.cloudflare.com/client/v4/zones" \
  -H "Authorization: Bearer $CF_TOKEN"

# Create DNS record
curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer $CF_TOKEN" \
  -d '{
    "type": "A",
    "name": "app",
    "content": "1.2.3.4",
    "proxied": true
  }'

# List Zero Trust gateway rules
curl "https://api.cloudflare.com/client/v4/accounts/{account_id}/gateway/rules" \
  -H "Authorization: Bearer $CF_TOKEN"
```

---

## Cloudflare vs AWS Services

| Feature | Cloudflare | AWS Equivalent |
|---------|-----------|---------------|
| DNS | Cloudflare DNS | Route 53 |
| CDN | Cloudflare CDN | CloudFront |
| WAF | Cloudflare WAF | AWS WAF |
| DDoS | Always-on, free | AWS Shield |
| Zero Trust/VPN | WARP + Access | Client VPN / Verified Access |
| Tunnels | cloudflared | None (need public IPs) |
| Edge compute | Workers | Lambda@Edge / CloudFront Functions |
| SSL/TLS | Free, automatic | ACM (free) + CloudFront |

Many companies use BOTH: Cloudflare for DNS/CDN/WAF, AWS for infrastructure.

---

*Cloudflare = the shield between your users and your servers! 🌐*
