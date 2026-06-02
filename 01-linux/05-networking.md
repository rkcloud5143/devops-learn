# Linux — Networking — Deep Dive

---

## NETWORK CONFIGURATION

```bash
# View IP addresses
ip addr show                     # All interfaces
ip addr show eth0                # Specific interface
hostname -I                      # Just the IP(s)

# Output explained:
# eth0: <BROADCAST,MULTICAST,UP>
#     inet 10.0.1.50/24          ← Private IP + subnet
#     inet6 fe80::1/64           ← IPv6 link-local

# View routing table
ip route show
# default via 10.0.1.1 dev eth0  ← Default gateway
# 10.0.1.0/24 dev eth0           ← Local subnet

# View DNS configuration
cat /etc/resolv.conf
# nameserver 10.0.0.2            ← VPC DNS server (AWS)
```

---

## CONNECTIVITY TESTING

```bash
# Ping — test if host is reachable
ping google.com                  # Continuous (Ctrl+C to stop)
ping -c 4 google.com             # Send 4 packets only
ping 10.0.2.50                   # Test internal connectivity

# Traceroute — show network path
traceroute google.com            # Show each hop
traceroute -n google.com         # Skip DNS lookup (faster)

# DNS lookup
nslookup example.com             # Query DNS
dig example.com                  # Detailed DNS query
dig +short example.com           # Just the IP
host example.com                 # Simple lookup

# Test specific port
telnet 10.0.2.50 5432            # Test PostgreSQL port
nc -zv 10.0.2.50 5432            # Netcat port test (better)
nc -zv 10.0.2.50 80-443          # Test port range

# curl — HTTP requests (VERY common in DevOps)
curl http://localhost:3000                # GET request
curl -I http://example.com               # Headers only
curl -s http://localhost:3000/health      # Silent mode
curl -X POST -d '{"key":"val"}' \
  -H "Content-Type: application/json" \
  http://localhost:3000/api               # POST with JSON
curl -o file.zip http://example.com/file  # Download file
curl -w "%{http_code}" -s -o /dev/null \
  http://localhost:3000/health            # Just status code

# wget — download files
wget http://example.com/file.tar.gz      # Download
wget -q -O - http://example.com          # Output to stdout
```

---

## PORTS & CONNECTIONS

```bash
# Show listening ports
ss -tlnp                         # TCP listening ports with process
# State  Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
# LISTEN 0       128     0.0.0.0:80          0.0.0.0:*          nginx
# LISTEN 0       128     0.0.0.0:22          0.0.0.0:*          sshd
# LISTEN 0       128     127.0.0.1:5432      0.0.0.0:*          postgres

ss -tlnp | grep :80              # Check if port 80 is in use
ss -tunap                        # All connections (TCP + UDP)

# Legacy (still works)
netstat -tlnp                    # Same as ss -tlnp
netstat -an | grep ESTABLISHED   # Active connections

# What process is using a port?
sudo lsof -i :80                 # What's on port 80?
sudo fuser 80/tcp                # PID using port 80
```

---

## FIREWALL (iptables / firewalld)

```
On AWS: Security Groups handle most firewall rules
On the OS level: iptables or firewalld

┌─── iptables (low-level) ─────────────────────────────────────┐
│                                                               │
│  Chains:                                                     │
│  INPUT   → traffic coming IN to this server                  │
│  OUTPUT  → traffic going OUT from this server                │
│  FORWARD → traffic passing THROUGH (routing)                 │
│                                                               │
│  sudo iptables -L -n                    # List rules         │
│  sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT         │
│  sudo iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT │
│  sudo iptables -A INPUT -j DROP         # Drop everything else│
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌─── firewalld (RHEL/Amazon Linux — higher level) ─────────────┐
│                                                               │
│  sudo firewall-cmd --list-all                                │
│  sudo firewall-cmd --add-port=80/tcp --permanent             │
│  sudo firewall-cmd --add-service=http --permanent            │
│  sudo firewall-cmd --reload                                  │
│                                                               │
└───────────────────────────────────────────────────────────────┘

┌─── ufw (Ubuntu — simplest) ──────────────────────────────────┐
│                                                               │
│  sudo ufw enable                                             │
│  sudo ufw allow 22                     # Allow SSH           │
│  sudo ufw allow 80                     # Allow HTTP          │
│  sudo ufw allow from 10.0.0.0/8       # Allow VPC range     │
│  sudo ufw status                       # Show rules          │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## /etc/hosts (Local DNS Override)

```bash
# /etc/hosts — resolves BEFORE DNS
127.0.0.1   localhost
10.0.2.50   db.internal
10.0.2.100  cache.internal
10.0.1.25   api.internal

# Now you can: ping db.internal → 10.0.2.50
# Useful for: testing, local dev, before DNS is set up
```

---

## NETWORK TROUBLESHOOTING FLOW

```
Can't connect to a service? Follow this:

1. Is the service running?
   └── systemctl status nginx

2. Is it listening on the right port?
   └── ss -tlnp | grep :80

3. Can you reach it locally?
   └── curl http://localhost:80

4. Can you reach it from another server?
   └── nc -zv 10.0.1.50 80

5. Is DNS resolving?
   └── nslookup myapp.com

6. Is the route correct?
   └── ip route show
   └── traceroute 10.0.1.50

7. Is the firewall blocking?
   └── sudo iptables -L -n
   └── Check AWS Security Group

8. Is the NACL blocking?
   └── Check AWS NACL (subnet level)
```

---

## CHECKLIST
- [ ] Check IP, routes, and DNS on a Linux server
- [ ] Test connectivity with ping, curl, nc, traceroute
- [ ] Find what's listening on a port with ss -tlnp
- [ ] Troubleshoot a connection failure step by step
