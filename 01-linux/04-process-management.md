# Linux — Process Management — Deep Dive

---

## WHAT IS A PROCESS

```
Process = a running program

Every process has:
- PID (Process ID): unique number
- PPID (Parent PID): who started it
- User: who owns it
- State: running, sleeping, stopped, zombie
- CPU/Memory usage
```

---

## VIEWING PROCESSES

```bash
# ps — snapshot of current processes
ps                              # Your processes only
ps aux                          # ALL processes (most common)
#  USER  PID  %CPU %MEM    VSZ   RSS TTY  STAT  START  TIME COMMAND
#  root    1   0.0  0.1 169316 11200 ?    Ss    Jan15  0:05 /sbin/init
#  nginx 1234  0.2  0.5  45000 20000 ?    S     10:30  0:10 nginx: worker

ps aux | grep nginx              # Find specific process
ps -ef                           # Full format listing
ps -ef --forest                  # Show parent-child tree

# top — live process monitor
top                              # Real-time view (q to quit)
#  Key shortcuts inside top:
#  P = sort by CPU
#  M = sort by Memory
#  k = kill a process (enter PID)
#  q = quit
#  1 = show individual CPUs

# htop — better version of top (install: apt install htop)
htop                             # Color-coded, mouse support

# Other useful commands
pgrep nginx                      # Find PID by name
pstree                           # Process tree
uptime                           # System uptime + load average
```

---

## MANAGING PROCESSES

```bash
# Start process in background
./long-script.sh &               # & = run in background
nohup ./script.sh &              # Keep running after logout
nohup ./script.sh > output.log 2>&1 &  # With log file

# Job control
jobs                             # List background jobs
fg %1                            # Bring job 1 to foreground
bg %1                            # Resume job 1 in background
Ctrl+Z                           # Suspend current process
Ctrl+C                           # Kill current process

# Kill processes
kill 1234                        # Graceful stop (SIGTERM)
kill -9 1234                     # Force kill (SIGKILL) — last resort
kill -HUP 1234                   # Reload config (SIGHUP)
killall nginx                    # Kill all processes by name
pkill -f "python app.py"        # Kill by command pattern

# Signals
┌──────────┬──────────┬──────────────────────────────────────┐
│ Signal   │ Number   │ What it does                         │
├──────────┼──────────┼──────────────────────────────────────┤
│ SIGTERM  │ 15       │ Graceful shutdown (default kill)     │
│ SIGKILL  │ 9        │ Force kill (cannot be caught)        │
│ SIGHUP   │ 1        │ Reload config / hangup               │
│ SIGINT   │ 2        │ Interrupt (Ctrl+C)                   │
│ SIGSTOP  │ 19       │ Pause process (Ctrl+Z)               │
│ SIGCONT  │ 18       │ Resume paused process                │
└──────────┴──────────┴──────────────────────────────────────┘
```

---

## SYSTEMD & SERVICES

```
systemd = init system that manages services on modern Linux

┌─── Service Lifecycle ────────────────────────────────────────┐
│                                                               │
│  ┌──────────┐  start   ┌─────────┐  stop    ┌──────────┐   │
│  │ Inactive │ ────────►│ Active  │ ────────►│ Inactive │   │
│  │ (dead)   │          │(running)│          │ (dead)   │   │
│  └──────────┘          └─────────┘          └──────────┘   │
│                             │                                │
│                             │ restart                        │
│                             ▼                                │
│                        Stop + Start                          │
│                                                               │
│  enabled  = starts automatically on boot                     │
│  disabled = does NOT start on boot                           │
└───────────────────────────────────────────────────────────────┘
```

### systemctl Commands (Use These Daily)
```bash
# Service management
sudo systemctl start nginx       # Start service
sudo systemctl stop nginx        # Stop service
sudo systemctl restart nginx     # Stop + Start
sudo systemctl reload nginx      # Reload config (no downtime)
sudo systemctl status nginx      # Check status (VERY common)

# Boot behavior
sudo systemctl enable nginx      # Start on boot
sudo systemctl disable nginx     # Don't start on boot
sudo systemctl is-enabled nginx  # Check if enabled

# List services
systemctl list-units --type=service              # Running services
systemctl list-units --type=service --state=failed  # Failed services
systemctl list-unit-files --type=service         # All services

# View logs for a service
journalctl -u nginx              # All logs for nginx
journalctl -u nginx -f           # Follow logs (like tail -f)
journalctl -u nginx --since "1 hour ago"  # Recent logs
journalctl -u nginx --since today         # Today's logs
journalctl -u nginx -n 50        # Last 50 lines
```

### Creating a Custom Service
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload     # Reload after creating/editing service
sudo systemctl enable myapp      # Enable on boot
sudo systemctl start myapp       # Start it
```

---

## RESOURCE MONITORING

```bash
# CPU
top                              # Live CPU usage
mpstat                           # CPU statistics
uptime                           # Load average (1, 5, 15 min)
# Load average: 1.0 = 1 CPU fully used
# 4 CPUs → load 4.0 = fully used, > 4.0 = overloaded

# Memory
free -h                          # Memory usage
#              total   used   free   shared  buff/cache  available
# Mem:          8Gi    2Gi    1Gi     100Mi      5Gi       5Gi
# Swap:         2Gi    0Gi    2Gi
#
# "available" is what matters (free + reclaimable cache)

# Disk
df -h                            # Disk space per filesystem
df -h /                          # Root partition only
du -sh /var/log                  # Size of a directory
du -sh /var/log/* | sort -rh | head -10  # Top 10 largest in /var/log
lsblk                            # List block devices (disks)
fdisk -l                         # Detailed disk info

# Network
ss -tlnp                         # Listening ports (modern)
netstat -tlnp                    # Listening ports (legacy)
iftop                            # Live network traffic
ip addr show                     # IP addresses
ip route show                    # Routing table

# I/O
iostat                           # Disk I/O statistics
iotop                            # Live I/O by process
```

---

## CHECKLIST
- [ ] Use ps aux and top to find resource-hungry processes
- [ ] Kill a process gracefully (SIGTERM) and forcefully (SIGKILL)
- [ ] Manage services with systemctl (start, stop, enable, status)
- [ ] Read service logs with journalctl
- [ ] Create a custom systemd service file
- [ ] Monitor CPU, memory, and disk usage
