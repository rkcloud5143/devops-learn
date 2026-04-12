# Linux — Disk & Storage Management — Deep Dive

---

## DISK CONCEPTS

```
Physical Disk → Partitions → Filesystem → Mount Point

┌─── /dev/sda (disk) ──────────────────────────────────────────┐
│                                                               │
│  /dev/sda1 (partition 1) → ext4 filesystem → mounted at /    │
│  /dev/sda2 (partition 2) → swap                              │
│                                                               │
└───────────────────────────────────────────────────────────────┘

On AWS EC2:
/dev/xvda or /dev/nvme0n1 = root EBS volume
/dev/xvdf or /dev/nvme1n1 = additional EBS volume
```

---

## VIEWING DISK INFO

```bash
# List block devices (disks and partitions)
lsblk
# NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# xvda    202:0    0   20G  0 disk
# └─xvda1 202:1    0   20G  0 part /
# xvdf    202:80   0  100G  0 disk

# Disk space usage
df -h                            # All filesystems
# Filesystem  Size  Used  Avail  Use%  Mounted on
# /dev/xvda1   20G   8G    12G   40%  /
# /dev/xvdf   100G  50G    50G   50%  /data

df -h /                          # Root only
df -i                            # Inode usage (can run out!)

# Directory sizes
du -sh /var/log                  # Total size of /var/log
du -sh /var/log/* | sort -rh | head  # Largest subdirectories
du -sh /* 2>/dev/null | sort -rh     # Largest top-level dirs
```

---

## MOUNTING & UNMOUNTING

```bash
# Mount a new EBS volume (after attaching in AWS console)
sudo lsblk                              # See the new disk
sudo file -s /dev/xvdf                  # Check if has filesystem
sudo mkfs -t ext4 /dev/xvdf            # Create filesystem (FIRST TIME ONLY!)
sudo mkdir /data                         # Create mount point
sudo mount /dev/xvdf /data              # Mount it

# Make it persist across reboots (/etc/fstab)
# Add this line to /etc/fstab:
/dev/xvdf  /data  ext4  defaults,nofail  0  2

# Verify fstab is correct (IMPORTANT — bad fstab = server won't boot)
sudo mount -a                            # Mount all from fstab

# Unmount
sudo umount /data                        # Unmount
```

---

## DISK TROUBLESHOOTING

```bash
# Disk full? Find what's using space
df -h                                    # Which filesystem is full?
du -sh /* 2>/dev/null | sort -rh | head  # Find largest directories
find / -size +100M -type f 2>/dev/null   # Find large files
journalctl --disk-usage                  # Journal log size
sudo journalctl --vacuum-size=100M       # Trim journal logs

# Common culprits:
# /var/log/     → old log files (set up logrotate)
# /tmp/         → temp files
# /var/lib/docker/ → docker images/containers
# docker system prune -a  → clean docker

# Inodes full (df shows space but can't create files)
df -i                                    # Check inode usage
find / -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head
# ^ finds directories with most files
```

---

## SWAP

```
Swap = disk space used as "overflow" memory when RAM is full

┌─── RAM (8 GB) ───────────────────────────────────────────────┐
│  Running processes use RAM                                    │
│  When RAM is full → least-used pages move to swap             │
└───────────────────────────────────────────────────────────────┘
         │ overflow
         ▼
┌─── Swap (2 GB on disk) ─────────────────────────────────────┐
│  Much slower than RAM                                        │
│  Prevents out-of-memory (OOM) kills                          │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Check swap
free -h                          # Shows swap usage
swapon --show                    # Show swap devices

# Create swap file (if none exists)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## LOG ROTATION

```
Problem: logs grow forever → disk fills up
Solution: logrotate (runs daily via cron)

# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily              # Rotate daily
    rotate 7           # Keep 7 rotated files
    compress           # Gzip old logs
    delaycompress      # Don't compress most recent rotated
    missingok          # Don't error if log missing
    notifempty         # Don't rotate empty files
    create 644 ubuntu ubuntu  # Permissions for new log
    postrotate
        systemctl reload myapp  # Reload app after rotation
    endscript
}

# Test logrotate
sudo logrotate -d /etc/logrotate.d/myapp   # Dry run
sudo logrotate -f /etc/logrotate.d/myapp   # Force run
```

---

## CHECKLIST
- [ ] Check disk usage with df -h and du -sh
- [ ] Mount an EBS volume and add to /etc/fstab
- [ ] Troubleshoot a full disk (find large files, clean logs)
- [ ] Understand swap and when it's needed
