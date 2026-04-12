# Linux — Essential Commands & Why You Need Them

This is your DAY 1 file. Learn these commands first before anything else.
Every command here is something you'll use DAILY as a DevOps engineer.

---

## WHY LINUX?

```
Almost every server in the cloud runs Linux.
AWS EC2, Docker containers, Kubernetes pods — all Linux.

As a DevOps engineer you will:
- SSH into servers to troubleshoot issues
- Read log files to find errors
- Check if services are running
- Monitor CPU, memory, and disk
- Write scripts to automate tasks
- Deploy and configure applications

You CANNOT do this job without Linux commands.
```

---

## 1. KNOW WHERE YOU ARE

```bash
pwd                     # "Print Working Directory" — shows your current location
                        # WHY: you need to know where you are before doing anything
                        # Example output: /home/ubuntu

whoami                  # Shows which user you're logged in as
                        # WHY: are you root? are you the right user?
                        # Example output: ubuntu

hostname                # Shows the server name
                        # WHY: am I on the right server? (prod vs dev?)
                        # Example output: ip-10-0-1-50
```

---

## 2. LOOK AROUND

```bash
ls                      # List files in current directory
                        # WHY: see what's here

ls -l                   # Long format — shows permissions, owner, size, date
                        # WHY: check who owns a file, when it was modified
                        # -rw-r--r-- 1 ubuntu ubuntu 4096 Jan 15 10:30 app.conf

ls -la                  # Same but includes hidden files (start with .)
                        # WHY: config files are often hidden (.env, .ssh, .bashrc)

ls -lh                  # Human-readable sizes (KB, MB, GB instead of bytes)
                        # WHY: quickly see if a log file is 5KB or 5GB
```

---

## 3. MOVE AROUND

```bash
cd /var/log             # Change directory — go to /var/log
                        # WHY: navigate to where logs, configs, or apps live

cd ..                   # Go up one level
                        # WHY: go back to parent directory

cd ~                    # Go to your home directory (/home/ubuntu)
                        # WHY: quick way to get "home"

cd -                    # Go to the PREVIOUS directory you were in
                        # WHY: toggle between two directories quickly
```

---

## 4. READ FILES

```bash
cat file.txt            # Print entire file to screen
                        # WHY: quick look at small files (configs, scripts)
                        # DON'T use on huge files (it floods your screen)

less file.txt           # Scroll through a file (q to quit)
                        # WHY: read large files page by page
                        # Use: arrow keys to scroll, /search to find text, q to quit

head -20 file.txt       # Show first 20 lines
                        # WHY: peek at the beginning of a file

tail -20 file.txt       # Show last 20 lines
                        # WHY: see the most recent entries in a log file

tail -f /var/log/syslog # Follow a log file in REAL-TIME
                        # WHY: watch logs as they happen (debugging live issues)
                        # THIS IS ONE OF THE MOST USED COMMANDS IN DEVOPS
                        # Ctrl+C to stop
```

---

## 5. SEARCH INSIDE FILES

```bash
grep "error" app.log           # Find lines containing "error"
                               # WHY: find errors in log files (most common use)

grep -i "error" app.log        # Case-insensitive (finds Error, ERROR, error)
                               # WHY: you don't know how the app logs errors

grep -r "timeout" /etc/        # Search recursively in all files under /etc/
                               # WHY: find which config file has a setting

grep -c "error" app.log        # Count how many lines match
                               # WHY: "how many errors happened today?"

grep -n "error" app.log        # Show line numbers
                               # WHY: know exactly where the error is
```

---

## 6. CREATE, COPY, MOVE, DELETE

```bash
# Create
touch newfile.txt              # Create an empty file
                               # WHY: create a placeholder, or update timestamp

mkdir mydir                    # Create a directory
mkdir -p a/b/c                 # Create nested directories
                               # WHY: -p creates parent dirs if they don't exist

# Copy
cp file.txt backup.txt         # Copy a file
cp -r mydir/ backup/           # Copy a directory (-r = recursive)
                               # WHY: make backups before changing things

# Move / Rename
mv old.txt new.txt             # Rename a file
mv file.txt /tmp/              # Move file to another directory
                               # WHY: organize files, rename configs

# Delete
rm file.txt                    # Delete a file (NO undo!)
rm -r mydir/                   # Delete a directory and everything inside
rm -rf mydir/                  # Force delete, no confirmation
                               # WHY: cleanup old files, temp data
                               # ⚠️ BE VERY CAREFUL with rm -rf
                               # NEVER run: rm -rf /  (deletes EVERYTHING)
```

---

## 7. CHECK SYSTEM HEALTH

```bash
# CPU and processes
top                            # Live view of CPU, memory, processes
                               # WHY: "why is the server slow?" — check CPU usage
                               # Press q to quit, P to sort by CPU, M by memory

ps aux                         # Snapshot of all running processes
                               # WHY: "is nginx running?" "what's using all the CPU?"

ps aux | grep nginx            # Find a specific process
                               # WHY: check if your app/service is running

# Memory
free -h                        # Show RAM usage (human-readable)
                               # WHY: "is the server running out of memory?"
                               # Look at "available" column, not "free"

# Disk
df -h                          # Show disk space per filesystem
                               # WHY: "is the disk full?" (most common server issue)
                               # If Use% is 90%+ → you have a problem

du -sh /var/log                # Size of a specific directory
                               # WHY: "what's filling up the disk?"

# Uptime
uptime                         # How long server has been running + load average
                               # WHY: did someone reboot it? is it overloaded?
                               # Load average > number of CPUs = overloaded
```

---

## 8. MANAGE SERVICES

```bash
sudo systemctl status nginx    # Is nginx running?
                               # WHY: first thing to check when something is down
                               # Shows: active (running) or inactive (dead)

sudo systemctl start nginx     # Start a service
sudo systemctl stop nginx      # Stop a service
sudo systemctl restart nginx   # Stop + Start (use after config changes)
sudo systemctl reload nginx    # Reload config without stopping (zero downtime)

sudo systemctl enable nginx    # Start automatically on boot
sudo systemctl disable nginx   # Don't start on boot

journalctl -u nginx -f         # Follow service logs in real-time
                               # WHY: see why a service crashed or won't start
journalctl -u nginx --since "1 hour ago"  # Recent logs only
```

---

## 9. NETWORK COMMANDS

```bash
ping google.com                # Test if you can reach a host
                               # WHY: "is the internet working?" "can I reach this server?"
                               # Ctrl+C to stop

curl http://localhost:3000     # Make an HTTP request
                               # WHY: test if your web app is responding
                               # Most used: curl http://localhost/health

curl -I http://example.com     # Show only HTTP headers
                               # WHY: check status code (200 OK? 500 error?)

ss -tlnp                       # Show listening ports and which process owns them
                               # WHY: "is my app listening on port 3000?"
                               # "what's already using port 80?"

ip addr show                   # Show IP addresses
                               # WHY: "what's this server's IP?"

nslookup example.com           # DNS lookup — what IP does this domain point to?
                               # WHY: troubleshoot DNS issues
```

---

## 10. PIPES — THE SUPERPOWER

```
Pipes ( | ) connect commands together.
Output of one command becomes input of the next.
THIS is what makes Linux powerful.
```

```bash
# Find errors in a log and count them
cat app.log | grep "error" | wc -l
# Read file → filter errors → count lines
# Answer: "there were 47 errors"

# Find what's using the most disk space
du -sh /* 2>/dev/null | sort -rh | head -5
# Get sizes → sort largest first → show top 5

# Find which process is using the most CPU
ps aux | sort -k3 -rn | head -5
# List processes → sort by CPU column → top 5

# Find your app's process
ps aux | grep "node" | grep -v grep
# List all → find node → exclude the grep itself

# Check if a port is in use
ss -tlnp | grep :80
# List ports → find port 80
```

---

## 11. SUDO — RUN AS ROOT

```bash
sudo command                   # Run a single command as root
                               # WHY: installing software, editing system configs,
                               # managing services — all need root permissions

sudo apt update                # Update package list (Ubuntu)
sudo yum install nginx         # Install software (Amazon Linux)
sudo vim /etc/nginx/nginx.conf # Edit system config files
sudo systemctl restart nginx   # Restart services

sudo -i                        # Switch to root shell (use sparingly)
exit                           # Go back to normal user

# If you get "Permission denied" → you probably need sudo
```

---

## 12. HELP — WHEN YOU FORGET

```bash
man ls                         # Manual page for any command
                               # Press q to quit, /search to find

ls --help                      # Quick help for a command

history                        # Show your command history
                               # WHY: "what did I run earlier?"

history | grep "docker"        # Find a specific past command

Ctrl+R                         # Reverse search through history
                               # Start typing and it finds matching commands
```

---

## DAILY PRACTICE ORDER

```
Week 1 — Practice these every day until they're muscle memory:

Day 1: pwd, ls, cd, cat, less, head, tail
Day 2: grep, find, pipes (|), wc
Day 3: cp, mv, rm, mkdir, touch, chmod
Day 4: ps, top, free, df, du
Day 5: systemctl (start/stop/status/restart)
Day 6: curl, ping, ss, ip addr
Day 7: Combine everything — SSH into EC2, check health,
       read logs, find errors, restart services
```

---

## CHEAT SHEET — Print This

```
WHERE AM I?     pwd, whoami, hostname
LOOK AROUND     ls -la
MOVE            cd /path, cd .., cd ~
READ FILES      cat, less, head, tail -f
SEARCH          grep "text" file
CREATE          touch file, mkdir dir
COPY/MOVE       cp, mv
DELETE           rm file, rm -r dir
PROCESSES       ps aux, top, kill PID
SERVICES        systemctl status/start/stop/restart
DISK            df -h, du -sh
MEMORY          free -h
NETWORK         ping, curl, ss -tlnp, ip addr
LOGS            tail -f /var/log/syslog, journalctl -u service
PERMISSIONS     chmod 755, chown user:group
INSTALL         sudo apt install (Ubuntu) / sudo yum install (Amazon)
HELP            man command, command --help
```
