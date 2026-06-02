# Linux — File System & Navigation — Deep Dive

---

## FILE SYSTEM HIERARCHY

```
/ (root — everything starts here)
│
├── /home/           ← User home directories
│   ├── /home/ubuntu/    ← Ubuntu default user
│   └── /home/ec2-user/  ← Amazon Linux default user
│
├── /root/           ← Root user's home (NOT /home/root)
│
├── /etc/            ← System configuration files
│   ├── /etc/nginx/nginx.conf      ← Nginx config
│   ├── /etc/ssh/sshd_config       ← SSH server config
│   ├── /etc/hosts                 ← Local DNS overrides
│   ├── /etc/hostname              ← Machine name
│   ├── /etc/passwd                ← User accounts
│   ├── /etc/shadow                ← Encrypted passwords
│   ├── /etc/group                 ← Group definitions
│   ├── /etc/fstab                 ← Disk mount config
│   ├── /etc/crontab               ← System cron jobs
│   ├── /etc/resolv.conf           ← DNS resolver config
│   ├── /etc/systemd/system/       ← Systemd service files
│   └── /etc/environment           ← System-wide env vars
│
├── /var/            ← Variable data (changes frequently)
│   ├── /var/log/                  ← ALL system logs
│   │   ├── syslog or messages     ← General system log
│   │   ├── auth.log or secure     ← Authentication logs
│   │   ├── nginx/access.log       ← Nginx access log
│   │   ├── nginx/error.log        ← Nginx error log
│   │   └── cloud-init-output.log  ← EC2 user data log
│   ├── /var/www/                  ← Web server files
│   └── /var/lib/                  ← Application state data
│       └── /var/lib/docker/       ← Docker data
│
├── /tmp/            ← Temporary files (cleared on reboot)
├── /opt/            ← Third-party software
├── /usr/            ← User programs and utilities
│   ├── /usr/bin/        ← User commands (ls, grep, etc.)
│   ├── /usr/sbin/       ← System admin commands
│   └── /usr/local/      ← Locally compiled software
│
├── /bin/            ← Essential commands (ls, cp, mv)
├── /sbin/           ← System binaries (iptables, fdisk)
├── /dev/            ← Device files
│   ├── /dev/sda         ← First disk
│   ├── /dev/sda1        ← First partition
│   ├── /dev/null        ← Black hole (discard output)
│   └── /dev/zero        ← Infinite zeros
│
├── /proc/           ← Virtual filesystem (running processes)
│   ├── /proc/cpuinfo    ← CPU information
│   ├── /proc/meminfo    ← Memory information
│   └── /proc/[PID]/     ← Per-process info
│
└── /mnt/ or /media/ ← Mount points for external drives
```

---

## NAVIGATION COMMANDS

```bash
# Where am I?
pwd                          # /home/ubuntu

# List files
ls                           # Basic list
ls -l                        # Long format (permissions, size, date)
ls -la                       # Include hidden files (start with .)
ls -lh                       # Human-readable sizes (KB, MB, GB)
ls -lt                       # Sort by modification time (newest first)
ls -lS                       # Sort by size (largest first)
ls -R                        # Recursive (show subdirectories)

# Change directory
cd /var/log                  # Absolute path
cd ..                        # Go up one level
cd ~                         # Go to home directory
cd -                         # Go to previous directory
cd ../..                     # Go up two levels

# Find files
find / -name "nginx.conf"              # Find by name
find /var/log -name "*.log"            # Find by pattern
find / -name "*.conf" -type f          # Files only
find / -name "config" -type d          # Directories only
find /tmp -mtime +7                    # Modified more than 7 days ago
find / -size +100M                     # Files larger than 100MB
find / -user ubuntu                    # Files owned by ubuntu
find / -perm 777                       # Files with 777 permissions

# Locate (faster than find, uses database)
locate nginx.conf                      # Instant search
sudo updatedb                          # Update the database first

# Which command am I running?
which nginx                  # /usr/sbin/nginx
whereis nginx                # Shows binary, source, man page
type ls                      # Shows if alias, builtin, or file
```

---

## FILE OPERATIONS

```bash
# Create
touch file.txt               # Create empty file
mkdir mydir                  # Create directory
mkdir -p a/b/c               # Create nested directories

# Copy
cp file.txt backup.txt       # Copy file
cp -r dir1/ dir2/            # Copy directory recursively
cp -p file.txt backup.txt    # Preserve permissions and timestamps

# Move / Rename
mv file.txt newname.txt      # Rename
mv file.txt /tmp/            # Move to another directory

# Delete
rm file.txt                  # Delete file
rm -r mydir/                 # Delete directory recursively
rm -rf mydir/                # Force delete (no confirmation) — CAREFUL!
rmdir emptydir/              # Delete empty directory only

# Links
ln -s /path/to/original link_name    # Symbolic link (shortcut)
ln /path/to/original hard_link       # Hard link (same inode)

# View files
cat file.txt                 # Print entire file
head -20 file.txt            # First 20 lines
tail -20 file.txt            # Last 20 lines
tail -f /var/log/syslog      # Follow log in real-time (VERY common)
less file.txt                # Scroll through file (q to quit)
wc -l file.txt               # Count lines
```

---

## TEXT PROCESSING (DevOps Uses These Daily)

```bash
# grep — search inside files
grep "error" app.log                    # Find lines with "error"
grep -i "error" app.log                 # Case-insensitive
grep -r "timeout" /etc/                 # Recursive search
grep -c "error" app.log                 # Count matches
grep -n "error" app.log                 # Show line numbers
grep -v "debug" app.log                 # Exclude lines with "debug"
grep -E "error|warning" app.log         # Multiple patterns (regex)

# awk — column extraction
ps aux | awk '{print $1, $11}'          # Print user and command
df -h | awk '{print $1, $5}'           # Print disk and usage %

# sed — find and replace
sed 's/old/new/g' file.txt              # Replace all occurrences
sed -i 's/old/new/g' file.txt           # Edit file in-place
sed -n '5,10p' file.txt                 # Print lines 5-10

# cut — extract columns
echo "a:b:c" | cut -d: -f2             # Output: b
cat /etc/passwd | cut -d: -f1          # List all usernames

# sort & uniq
sort file.txt                           # Sort lines
sort -r file.txt                        # Reverse sort
sort -n file.txt                        # Numeric sort
sort file.txt | uniq                    # Remove duplicates
sort file.txt | uniq -c | sort -rn     # Count + sort by frequency

# Pipes — chain commands (THE most important concept)
cat /var/log/syslog | grep "error" | wc -l
#   read file       → filter errors  → count them

ps aux | grep nginx | grep -v grep
#   list processes → find nginx → exclude grep itself

df -h | awk '{print $5}' | sort -rn | head -5
#   disk usage → get % column → sort descending → top 5
```

---

## REDIRECTION

```bash
# Output redirection
echo "hello" > file.txt       # Write (overwrite)
echo "world" >> file.txt      # Append

# Error redirection
command 2> errors.log          # Redirect errors only
command > output.log 2>&1      # Redirect both stdout and stderr
command &> all.log             # Same as above (shorthand)

# Discard output
command > /dev/null 2>&1       # Silence everything

# Input redirection
sort < unsorted.txt            # Read from file
mysql < backup.sql             # Feed SQL file to mysql

# Here document
cat << EOF > config.txt
server {
    listen 80;
    server_name myapp.com;
}
EOF
```

---

## CHECKLIST
- [ ] Navigate the file system without looking up commands
- [ ] Use grep + pipes to filter log files
- [ ] Know where config files live (/etc/) and logs live (/var/log/)
- [ ] Use find to locate files by name, size, and age
