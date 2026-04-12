# Networking — CIDR & Subnets

## IP Addresses & CIDR (You MUST know this for AWS VPC)
```
IP Address:  192.168.1.0
Subnet Mask: 255.255.255.0  =  /24

CIDR Notation Examples:
┌──────────────────┬───────────┬──────────────────────────────┐
│ CIDR             │ # of IPs  │ Use Case                     │
├──────────────────┼───────────┼──────────────────────────────┤
│ 10.0.0.0/16      │ 65,536    │ VPC (entire network)         │
│ 10.0.1.0/24      │ 256       │ Subnet (one AZ)              │
│ 10.0.1.0/28      │ 16        │ Small subnet                 │
│ 0.0.0.0/0        │ All IPs   │ "Anywhere" (internet access) │
└──────────────────┴───────────┴──────────────────────────────┘

Quick trick: /24 = 256 IPs, /16 = 65536 IPs
Each +1 in CIDR halves the IPs: /24=256, /25=128, /26=64, /27=32, /28=16
```

## Checklist
- [ ] Understand CIDR notation — be able to calculate subnet sizes
