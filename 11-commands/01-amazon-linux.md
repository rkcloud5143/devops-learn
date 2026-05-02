# Amazon Linux — Command Reference 🐧

Essential commands for managing Amazon Linux 2 / Amazon Linux 2023 servers.

---

## System Information

```bash
# OS version
cat /etc/os-release
cat /etc/system-release

# Kernel version
uname -r
uname -a

# Hostname
hostname
hostnamectl

# Uptime & load
uptime
w

# CPU info
lscpu
cat /proc/cpuinfo

# Memory info
free -h
cat /proc/meminfo

# Disk info
lsblk
fdisk -l
df -h
du -sh /path

# System architecture
arch
```

---

## Package Management (yum / dnf)

```bash
# ─── Amazon Linux 2 (yum) ───
yum update -y                          # Update all packages
yum install -y nginx                   # Install package
yum remove nginx                       # Remove package
yum list installed                     # List installed packages
yum list available | grep docker       # Search available
yum info nginx                         # Package details
yum clean all                          # Clear cache
yum history                            # Install/update history
yum downgrade package                  # Downgrade to previous version

# Enable extras (Amazon Linux 2)
amazon-linux-extras list               # List available topics
amazon-linux-extras install docker     # Install from extras
amazon-linux-extras install epel       # Enable EPEL repo

# ─── Amazon Linux 2023 (dnf) ───
dnf update -y
dnf install -y nginx
dnf remove nginx
dnf list installed
dnf search docker
dnf info nginx
dnf clean all
dnf history

# RPM (low-level package manager)
rpm -qa                                # List all installed
rpm -qi nginx                          # Package info
rpm -ql nginx                          # List files in package
rpm -ivh package.rpm                   # Install from file
rpm -e package                         # Remove
```

---

## User & Group Management

```bash
# Users
useradd deploy                         # Create user
useradd -m -s /bin/bash deploy         # With home dir and shell
userdel deploy                         # Delete user
userdel -r deploy                      # Delete user + home dir
passwd deploy                          # Set password
usermod -aG docker deploy              # Add to group
usermod -L deploy                      # Lock account
usermod -U deploy                      # Unlock account
id deploy                              # Show UID, GID, groups
whoami                                 # Current user

# Groups
groupadd devops                        # Create group
groupdel devops                        # Delete group
groups deploy                          # Show user's groups
getent group docker                    # List group members

# Sudo
visudo                                 # Edit sudoers safely
echo "deploy ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/deploy

# Switch user
su - deploy                            # Login shell
sudo -u deploy command                 # Run as user
```

---

## File & Directory Operations

```bash
# Navigation
pwd                                    # Current directory
cd /path                               # Change directory
cd -                                   # Previous directory
cd ~                                   # Home directory

# Listing
ls -la                                 # Detailed + hidden
ls -lh                                 # Human-readable sizes
ls -lt                                 # Sort by time
ls -lS                                 # Sort by size
tree -L 2                              # Tree view (install: yum install tree)

# Create
mkdir -p /path/to/dir                  # Create with parents
touch file.txt                         # Create empty file

# Copy / Move / Delete
cp -r source dest                      # Copy recursive
mv old new                             # Move/rename
rm -rf dir                             # Remove recursive (CAREFUL!)

# Links
ln -s /target /link                    # Symbolic link
ln /target /link                       # Hard link
readlink -f /link                      # Resolve symlink

# Find
find / -name "*.log" -mtime +7         # Files older than 7 days
find / -size +100M -type f             # Files > 100MB
find / -user deploy -type f            # Files owned by user
find /var/log -name "*.log" -delete    # Find and delete
locate filename                        # Fast search (updatedb first)

# File content
cat file                               # Display file
head -n 20 file                        # First 20 lines
tail -n 50 file                        # Last 50 lines
tail -f /var/log/messages              # Follow log in real-time
less file                              # Paginated view
wc -l file                             # Count lines
diff file1 file2                       # Compare files
```

---

## Permissions

```bash
# Change permissions
chmod 755 file                         # rwxr-xr-x
chmod 644 file                         # rw-r--r--
chmod +x script.sh                     # Add execute
chmod -R 755 dir                       # Recursive

# Change ownership
chown user:group file
chown -R deploy:deploy /app

# Special permissions
chmod u+s file                         # SUID
chmod g+s dir                          # SGID
chmod +t dir                           # Sticky bit

# ACLs
getfacl file                           # View ACLs
setfacl -m u:deploy:rwx file           # Set ACL
```

---

## Process Management

```bash
# View processes
ps aux                                 # All processes
ps -ef                                 # Full format
ps aux | grep nginx                    # Find specific
top                                    # Real-time (q to quit)
htop                                   # Better top (install: yum install htop)
pgrep -a nginx                         # Find by name
pidof nginx                            # Get PID

# Kill processes
kill PID                               # Graceful (SIGTERM)
kill -9 PID                            # Force (SIGKILL)
pkill nginx                            # Kill by name
killall nginx                          # Kill all by name

# Background / Foreground
command &                              # Run in background
nohup command &                        # Survive logout
jobs                                   # List background jobs
fg %1                                  # Bring to foreground
bg %1                                  # Send to background
disown %1                              # Detach from terminal

# Resource usage
top -bn1 | head -20                    # Snapshot
pidstat 1 5                            # Per-process stats
strace -p PID                          # System calls
lsof -p PID                            # Open files by process
```

---

## Service Management (systemd)

```bash
# Service control
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx                 # Reload config without restart
systemctl status nginx
systemctl enable nginx                 # Start at boot
systemctl disable nginx                # Don't start at boot
systemctl is-active nginx
systemctl is-enabled nginx

# List services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service

# Logs
journalctl -u nginx                    # Service logs
journalctl -u nginx -f                 # Follow
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since today
journalctl -xe                         # Recent errors
journalctl --disk-usage                # Log disk usage
journalctl --vacuum-time=7d            # Clean old logs

# Reload systemd after unit file changes
systemctl daemon-reload
```

---

## Networking

```bash
# IP & interfaces
ip addr                                # Show IPs
ip addr show eth0                      # Specific interface
ip link                                # Interface status
ifconfig                               # Legacy (net-tools)

# Routing
ip route                               # Routing table
ip route add 10.0.0.0/8 via 192.168.1.1
traceroute host                        # Trace path
mtr host                               # Better traceroute

# DNS
nslookup domain.com
dig domain.com
dig +short domain.com
host domain.com
cat /etc/resolv.conf                   # DNS config

# Connectivity
ping -c 4 host                         # 4 pings
curl -v https://example.com            # HTTP request
curl -s -o /dev/null -w "%{http_code}" URL  # Just status code
wget https://example.com/file          # Download file
telnet host 80                         # Test TCP port
nc -zv host 80                         # Test port (netcat)

# Ports & connections
ss -tuln                               # Listening ports
ss -tunap                              # All connections with process
ss -s                                  # Connection summary
netstat -tuln                          # Legacy
lsof -i :80                            # What's on port 80
lsof -i -P -n                         # All network connections

# Firewall (firewalld — Amazon Linux 2023)
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

# iptables (Amazon Linux 2)
iptables -L -n -v                      # List rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables-save > /etc/sysconfig/iptables

# Network troubleshooting
tcpdump -i eth0 port 80                # Capture traffic
tcpdump -i eth0 -w capture.pcap        # Save to file
nmap -sT host                          # Port scan
```

---

## Disk & Storage

```bash
# Disk usage
df -h                                  # Filesystem usage
df -i                                  # Inode usage
du -sh /path                           # Directory size
du -sh /* | sort -h                    # Largest directories
ncdu /                                 # Interactive (install: yum install ncdu)

# Block devices
lsblk                                  # List block devices
blkid                                  # Block device IDs
fdisk -l                               # Partition table

# Mount
mount /dev/xvdf /data                  # Mount device
umount /data                           # Unmount
mount -a                               # Mount all in fstab
cat /etc/fstab                         # Persistent mounts

# EBS volume (attach new volume)
lsblk                                  # Find new device (e.g., xvdf)
file -s /dev/xvdf                      # Check if formatted
mkfs -t xfs /dev/xvdf                  # Format (XFS)
mkdir /data
mount /dev/xvdf /data
echo "/dev/xvdf /data xfs defaults,nofail 0 2" >> /etc/fstab

# Extend EBS volume (after resizing in AWS)
lsblk                                  # Check current size
growpart /dev/xvda 1                   # Grow partition
xfs_growfs /data                       # Grow XFS filesystem
# or
resize2fs /dev/xvda1                   # Grow ext4 filesystem

# LVM
pvs                                    # Physical volumes
vgs                                    # Volume groups
lvs                                    # Logical volumes
lvextend -L +10G /dev/vg/lv            # Extend LV
```

---

## SSH & Remote Access

```bash
# Connect
ssh user@host
ssh -i key.pem ec2-user@ip
ssh -p 2222 user@host                  # Custom port

# Key management
ssh-keygen -t ed25519                  # Generate key
ssh-copy-id user@host                  # Copy public key
cat ~/.ssh/authorized_keys             # View authorized keys

# SCP (copy files)
scp file.txt user@host:/path           # Upload
scp user@host:/path/file.txt .         # Download
scp -r dir user@host:/path             # Recursive

# Rsync (better for large transfers)
rsync -avz /local/ user@host:/remote/
rsync -avz --delete /local/ user@host:/remote/  # Mirror (delete extra)
rsync -avz --progress /local/ user@host:/remote/

# SSH tunnel
ssh -L 8080:localhost:80 user@host     # Local port forward
ssh -D 1080 user@host                  # SOCKS proxy

# SSH config (~/.ssh/config)
# Host myserver
#   HostName 10.0.1.50
#   User ec2-user
#   IdentityFile ~/.ssh/my-key.pem
# Then: ssh myserver
```

---

## Text Processing

```bash
# grep
grep "error" /var/log/messages         # Search
grep -i "error" file                   # Case insensitive
grep -r "pattern" /path                # Recursive
grep -c "error" file                   # Count matches
grep -v "debug" file                   # Exclude pattern
grep -E "error|warn" file              # Regex (egrep)
grep -A 3 -B 2 "error" file           # Context lines

# sed
sed 's/old/new/g' file                 # Replace (print)
sed -i 's/old/new/g' file             # Replace (in-place)
sed -n '10,20p' file                   # Print lines 10-20
sed '/pattern/d' file                  # Delete matching lines

# awk
awk '{print $1, $3}' file             # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd      # Custom delimiter
awk '$3 > 100 {print $0}' file        # Conditional
awk '{sum+=$1} END {print sum}' file   # Sum column

# sort & uniq
sort file                              # Sort lines
sort -n file                           # Numeric sort
sort -r file                           # Reverse
sort -u file                           # Unique
sort file | uniq -c | sort -rn         # Count occurrences

# cut
cut -d: -f1 /etc/passwd               # Cut by delimiter
cut -c1-10 file                        # Cut by character position

# tr
echo "HELLO" | tr 'A-Z' 'a-z'         # Lowercase
cat file | tr -d '\r'                  # Remove carriage returns

# xargs
find . -name "*.log" | xargs rm       # Delete found files
cat urls.txt | xargs -I {} curl {}    # Process each line
```

---

## Compression & Archives

```bash
# tar
tar czf archive.tar.gz /path          # Create gzip archive
tar cjf archive.tar.bz2 /path         # Create bzip2 archive
tar xzf archive.tar.gz                # Extract gzip
tar xzf archive.tar.gz -C /dest       # Extract to directory
tar tzf archive.tar.gz                # List contents

# gzip
gzip file                             # Compress (replaces file)
gunzip file.gz                        # Decompress
zcat file.gz                          # View without extracting

# zip
zip -r archive.zip /path              # Create zip
unzip archive.zip                     # Extract
unzip -l archive.zip                  # List contents
```

---

## Cron Jobs

```bash
# Edit crontab
crontab -e                             # Edit current user's crontab
crontab -l                             # List crontab
crontab -r                             # Remove crontab
crontab -u deploy -e                   # Edit another user's crontab

# Format: minute hour day month weekday command
# ┌───── minute (0-59)
# │ ┌───── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌───── month (1-12)
# │ │ │ │ ┌───── day of week (0-7, 0=Sun)
# │ │ │ │ │
# * * * * * command

0 3 * * *   /opt/scripts/backup.sh     # Daily at 3am
*/5 * * * * /opt/scripts/health.sh     # Every 5 minutes
0 0 * * 0   /opt/scripts/weekly.sh     # Weekly (Sunday midnight)
0 9 1 * *   /opt/scripts/monthly.sh    # Monthly (1st at 9am)

# System cron directories
/etc/cron.d/                           # Drop-in cron files
/etc/cron.daily/                       # Daily scripts
/etc/cron.hourly/                      # Hourly scripts
/etc/cron.weekly/                      # Weekly scripts
```

---

## AWS CLI (on Amazon Linux)

```bash
# Already installed on Amazon Linux
aws --version

# Configure
aws configure                          # Set credentials
aws configure --profile prod           # Named profile
aws sts get-caller-identity            # Who am I?

# EC2
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table
aws ec2 start-instances --instance-ids i-1234567890
aws ec2 stop-instances --instance-ids i-1234567890

# S3
aws s3 ls                              # List buckets
aws s3 ls s3://bucket/prefix/          # List objects
aws s3 cp file s3://bucket/            # Upload
aws s3 cp s3://bucket/file .           # Download
aws s3 sync /local s3://bucket/path    # Sync
aws s3 rm s3://bucket/file             # Delete

# Logs
aws logs tail /ecs/my-service --follow
aws logs filter-log-events --log-group-name /ecs/my-service --filter-pattern "ERROR"

# SSM (connect without SSH)
aws ssm start-session --target i-1234567890

# ECR
aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ca-central-1.amazonaws.com

# EKS
aws eks update-kubeconfig --name my-cluster --region ca-central-1
```

---

## Troubleshooting Cheat Sheet

### Finding Service Logs

```bash
# ─── Step 1: Check systemd logs ───
journalctl -u tomcat                   # All logs for service
journalctl -u tomcat -f                # Follow live
journalctl -u tomcat --since "1 hour ago"
journalctl -u tomcat -p err            # Errors only

# Don't know the service name?
systemctl list-units --type=service | grep tomcat

# ─── Step 2: Find where service writes log files ───
systemctl cat tomcat                   # View unit file — look for log paths

# ─── Step 3: Find open log files (BEST trick) ───
lsof -p $(pidof java) 2>/dev/null | grep "\.log"   # Tomcat runs on Java
lsof -p $(pgrep nginx) | grep log                   # Nginx
# Shows EXACTLY which files the process is writing to

# ─── Step 4: Search disk for log files ───
find /var/log -name "*tomcat*"
find / -name "*.log" 2>/dev/null | grep tomcat
```

### Common Service Log Locations

```
Service       Log Location
─────────     ──────────────────────────────────────────────
Tomcat        /opt/tomcat/logs/catalina.out
              /var/log/tomcat/
Nginx         /var/log/nginx/access.log
              /var/log/nginx/error.log
Apache/httpd  /var/log/httpd/access_log
              /var/log/httpd/error_log
MySQL         /var/log/mysqld.log
PostgreSQL    /var/log/postgresql/
Docker        docker logs <container>
              /var/lib/docker/containers/<id>/<id>-json.log
SSH           /var/log/secure (Amazon Linux / RHEL)
              /var/log/auth.log (Ubuntu)
System        /var/log/messages (Amazon Linux / RHEL)
              /var/log/syslog (Ubuntu)
Kernel        dmesg  or  /var/log/dmesg
Cron          /var/log/cron
Cloud-init    /var/log/cloud-init.log
              /var/log/cloud-init-output.log
User data     /var/log/cloud-init-output.log
```

### General Troubleshooting

```bash
# Server slow?
uptime                                 # Load average
top                                    # CPU/memory hogs
iostat -x 1 3                          # Disk I/O
vmstat 1 5                             # System stats
dmesg | tail                           # Kernel messages

# Disk full?
df -h                                  # Which filesystem?
du -sh /* | sort -h                    # What's using space?
find / -size +100M -type f 2>/dev/null # Large files
journalctl --vacuum-time=3d            # Clean old logs

# Can't connect?
ping host                              # Network reachable?
ss -tuln | grep PORT                   # Service listening?
systemctl status service               # Service running?
iptables -L -n                         # Firewall blocking?
cat /var/log/secure                    # Auth failures

# Service won't start?
systemctl status service               # Status + recent logs
journalctl -u service -n 50            # More logs
systemctl cat service                  # View unit file
```

---

*Keep this open in a terminal tab — you'll use it daily! 🐧*
