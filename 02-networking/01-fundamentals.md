# Networking — Fundamentals

## How the Internet Works (Simplified)
```
User types example.com
        │
        ▼
┌──────────────┐    "What IP is example.com?"    ┌───────────┐
│   Browser    │ ──────────────────────────────► │  DNS Server│
│              │ ◄────────────────────────────── │            │
└──────────────┘    "It's 93.184.216.34"         └───────────┘
        │
        ▼  Connects to 93.184.216.34:443 (HTTPS)
┌──────────────┐                                 ┌───────────┐
│   Browser    │ ──────── TCP Connection ──────► │ Web Server│
│              │ ◄──────── HTML Response ─────── │ (EC2)     │
└──────────────┘                                 └───────────┘
```

## TCP vs UDP
```
TCP (Transmission Control Protocol)
- Reliable, ordered delivery
- Used for: HTTP, SSH, databases
- Think: registered mail (confirmed delivery)

UDP (User Datagram Protocol)
- Fast, no guarantee of delivery
- Used for: DNS, video streaming, gaming
- Think: throwing a paper airplane (hope it arrives)
```
