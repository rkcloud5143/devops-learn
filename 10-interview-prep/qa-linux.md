# Linux Q&A (150+ Questions)

Basic, Advanced, and Scenario-based questions for Linux.

---

## Basic

**Q1: What is Linux?**
A: An open-source, Unix-like operating system kernel. Combined with GNU tools, it forms a complete OS (GNU/Linux). Used in servers, containers, embedded systems, and Android.

**Q2: What's the difference between Linux and Unix?**
A: Unix is proprietary (Solaris, AIX, HP-UX). Linux is open-source and free. Linux is "Unix-like" — similar commands and philosophy but different codebase.

**Q3: What is a Linux distribution?**
A: A complete OS built on the Linux kernel with package managers, tools, and desktop environments. Examples: Ubuntu, CentOS, RHEL, Debian, Amazon Linux.

**Q4: What is the root user?**
A: The superuser with UID 0. Has unrestricted access to everything. Use `sudo` instead of logging in as root.

**Q5: What does `sudo` do?**
A: Executes a command as another user (default: root). Configured in `/etc/sudoers`. Safer than logging in as root.

**Q6: How do you check the current user?**
A: `whoami` or `id`

**Q7: How do you switch users?**
A: `su - username` (with login shell) or `su username` (without)

**Q8: What is the home directory?**
A: User's personal directory. Usually `/home/username`. Root's home is `/root`. Represented by `~`.

**Q9: What does `pwd` do?**
A: Print Working Directory — shows your current location in the filesystem.

**Q10: How do you list files?**
A: `ls` (basic), `ls -l` (detailed), `ls -la` (include hidden), `ls -lh` (human-readable sizes)

**Q11: What are hidden files in Linux?**
A: Files starting with a dot (`.bashrc`, `.ssh`). Not shown by default in `ls`.

**Q12: How do you create a directory?**
A: `mkdir dirname` or `mkdir -p path/to/nested/dir` (creates parents)

**Q13: How do you remove a directory?**
A: `rmdir dirname` (empty only) or `rm -r dirname` (recursive, including contents)

**Q14: How do you copy files?**
A: `cp source dest` or `cp -r source dest` (recursive for directories)

**Q15: How do you move/rename files?**
A: `mv oldname newname` (rename) or `mv file /path/to/dest` (move)

**Q16: How do you delete files?**
A: `rm filename` or `rm -f filename` (force, no prompt)

**Q17: What does `cat` do?**
A: Concatenates and displays file contents. `cat file.txt`

**Q18: How do you view the beginning of a file?**
A: `head filename` (first 10 lines) or `head -n 20 filename` (first 20)

**Q19: How do you view the end of a file?**
A: `tail filename` (last 10 lines) or `tail -f filename` (follow in real-time — great for logs)

**Q20: What does `less` do?**
A: Paginated file viewer. Navigate with arrows, search with `/`, quit with `q`.

**Q21: How do you search for text in a file?**
A: `grep "pattern" filename` or `grep -r "pattern" directory` (recursive)

**Q22: What does `grep -i` do?**
A: Case-insensitive search.

**Q23: What does `grep -v` do?**
A: Inverts match — shows lines that DON'T match the pattern.

**Q24: How do you count lines in a file?**
A: `wc -l filename`

**Q25: What does `wc` stand for?**
A: Word Count. `-l` lines, `-w` words, `-c` bytes, `-m` characters.

**Q26: How do you find files by name?**
A: `find /path -name "filename"` or `find . -name "*.log"` (wildcards)

**Q27: How do you find files modified in the last 24 hours?**
A: `find /path -mtime -1`

**Q28: What does `locate` do?**
A: Searches a pre-built database for files. Faster than `find` but may be outdated. Update with `updatedb`.

**Q29: What is a symbolic link?**
A: A pointer to another file (like a shortcut). Created with `ln -s target linkname`.

**Q30: What is a hard link?**
A: Another name for the same file data (same inode). Created with `ln target linkname`. Can't cross filesystems.

**Q31: What's the difference between soft and hard links?**
A: Soft link points to a path (breaks if target moves). Hard link points to data (works even if original name deleted). Hard links can't cross filesystems or link directories.

**Q32: What are file permissions in Linux?**
A: Read (r=4), Write (w=2), Execute (x=1) for Owner, Group, Others. Example: `rwxr-xr--` = 754.

**Q33: How do you change file permissions?**
A: `chmod 755 file` (numeric) or `chmod u+x file` (symbolic)

**Q34: What does `chmod 777` mean?**
A: Everyone can read, write, and execute. Generally a bad idea for security.

**Q35: How do you change file ownership?**
A: `chown user:group filename` or `chown -R user:group directory` (recursive)

**Q36: What is umask?**
A: Default permission mask for new files. `umask 022` means new files get 644, directories get 755.

**Q37: What is `/etc/passwd`?**
A: User account information. Format: `username:x:UID:GID:comment:home:shell`

**Q38: What is `/etc/shadow`?**
A: Encrypted passwords and password policies. Only readable by root.

**Q39: What is `/etc/group`?**
A: Group definitions. Format: `groupname:x:GID:members`

**Q40: How do you add a user?**
A: `useradd username` or `adduser username` (interactive on Debian/Ubuntu)

**Q41: How do you delete a user?**
A: `userdel username` or `userdel -r username` (also removes home directory)

**Q42: How do you add a user to a group?**
A: `usermod -aG groupname username` (-a = append, -G = supplementary groups)

**Q43: How do you check which groups a user belongs to?**
A: `groups username` or `id username`

**Q44: What is the difference between primary and secondary groups?**
A: Primary group (GID in /etc/passwd) is default for new files. Secondary groups (in /etc/group) provide additional access.

**Q45: What does `passwd` do?**
A: Changes user password. Root can change any user's password: `passwd username`

**Q46: How do you lock a user account?**
A: `usermod -L username` or `passwd -l username`

**Q47: What is a process?**
A: A running instance of a program with its own PID, memory space, and resources.

**Q48: How do you list running processes?**
A: `ps aux` (all processes) or `ps -ef` (full format)

**Q49: What does `top` do?**
A: Real-time process viewer. Shows CPU, memory, and process list. Press `q` to quit.

**Q50: What is `htop`?**
A: Enhanced version of `top` with colors, mouse support, and easier navigation.

**Q51: How do you kill a process?**
A: `kill PID` (graceful SIGTERM) or `kill -9 PID` (force SIGKILL)

**Q52: What's the difference between SIGTERM and SIGKILL?**
A: SIGTERM (15) asks process to terminate gracefully (can be caught). SIGKILL (9) forces immediate termination (can't be caught).

**Q53: How do you find a process by name?**
A: `pgrep processname` or `ps aux | grep processname`

**Q54: How do you kill a process by name?**
A: `pkill processname` or `killall processname`

**Q55: What is a daemon?**
A: A background process that runs continuously, usually started at boot. Examples: sshd, httpd, cron.

**Q56: How do you run a process in the background?**
A: Append `&`: `command &` or use `nohup command &` to survive logout.

**Q57: What does `nohup` do?**
A: Runs a command immune to hangups. Process continues after you log out.

**Q58: How do you bring a background job to foreground?**
A: `fg` or `fg %jobnumber`

**Q59: How do you list background jobs?**
A: `jobs`

**Q60: What is PID 1?**
A: The init process (systemd on modern systems). Parent of all processes. If it dies, system crashes.

**Q61: What is a zombie process?**
A: A process that finished but its parent hasn't read its exit status. Shows as `Z` in `ps`. Harmless but indicates parent bug.

**Q62: What is an orphan process?**
A: A process whose parent died. Gets adopted by PID 1 (init/systemd).

**Q63: How do you check system uptime?**
A: `uptime`

**Q64: What is load average?**
A: Average number of processes waiting for CPU over 1, 5, and 15 minutes. Load > number of CPUs = overloaded.

**Q65: How do you check memory usage?**
A: `free -h` (human-readable)

**Q66: What is swap space?**
A: Disk space used as virtual memory when RAM is full. Slower than RAM.

**Q67: How do you check disk usage?**
A: `df -h` (filesystem usage) or `du -sh directory` (directory size)

**Q68: What does `df` stand for?**
A: Disk Free.

**Q69: What does `du` stand for?**
A: Disk Usage.

**Q70: How do you find large files?**
A: `find / -size +100M -type f` (files over 100MB)

**Q71: What is an inode?**
A: Data structure storing file metadata (permissions, owner, timestamps, data block pointers). Each file has one inode.

**Q72: How do you check inode usage?**
A: `df -i`

**Q73: What happens when you run out of inodes?**
A: Can't create new files even if disk space is available. Common with many small files.

**Q74: What is `/dev/null`?**
A: A special file that discards all data written to it. "Black hole" for output.

**Q75: How do you redirect output to a file?**
A: `command > file` (overwrite) or `command >> file` (append)

**Q76: How do you redirect stderr?**
A: `command 2> file` or `command 2>&1` (stderr to stdout)

**Q77: What does `2>&1` mean?**
A: Redirect file descriptor 2 (stderr) to file descriptor 1 (stdout).

**Q78: What is a pipe?**
A: Connects stdout of one command to stdin of another. `command1 | command2`

**Q79: What does `tee` do?**
A: Reads stdin and writes to both stdout and a file. `command | tee file.txt`

**Q80: What is stdin, stdout, stderr?**
A: Standard input (0), standard output (1), standard error (2). The three default I/O streams.

---

## Advanced

**Q81: What is the Linux boot process?**
A: BIOS/UEFI → Bootloader (GRUB) → Kernel loads → initramfs → systemd (PID 1) → Services start → Login prompt

**Q82: What is GRUB?**
A: Grand Unified Bootloader. Loads the kernel and initramfs. Config at `/boot/grub/grub.cfg`.

**Q83: What is initramfs?**
A: Initial RAM filesystem. Temporary root filesystem with drivers needed to mount the real root.

**Q84: What is systemd?**
A: Modern init system and service manager. Manages services, logging, networking, and more.

**Q85: How do you start/stop a service with systemd?**
A: `systemctl start servicename` / `systemctl stop servicename`

**Q86: How do you enable a service to start at boot?**
A: `systemctl enable servicename`

**Q87: How do you check service status?**
A: `systemctl status servicename`

**Q88: How do you view systemd logs?**
A: `journalctl` or `journalctl -u servicename` (specific service)

**Q89: What is a systemd unit file?**
A: Configuration file defining a service, socket, mount, etc. Located in `/etc/systemd/system/` or `/lib/systemd/system/`.

**Q90: How do you reload systemd after changing a unit file?**
A: `systemctl daemon-reload`

**Q91: What is cron?**
A: Job scheduler for running commands at specified times.

**Q92: Where are cron jobs defined?**
A: User crontabs (`crontab -e`), `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.daily/`, etc.

**Q93: What does `* * * * *` mean in cron?**
A: Every minute. Format: minute hour day-of-month month day-of-week.

**Q94: How do you run a job every day at 3am?**
A: `0 3 * * * /path/to/script`

**Q95: What is `/etc/fstab`?**
A: Filesystem table. Defines what filesystems to mount at boot and how.

**Q96: How do you mount a filesystem?**
A: `mount /dev/sdb1 /mnt/data` or `mount -a` (mount all in fstab)

**Q97: How do you unmount a filesystem?**
A: `umount /mnt/data` (note: umount, not unmount)

**Q98: What is LVM?**
A: Logical Volume Manager. Allows flexible disk management — resize volumes, span multiple disks, snapshots.

**Q99: What are the components of LVM?**
A: Physical Volumes (PV) → Volume Groups (VG) → Logical Volumes (LV)

**Q100: How do you extend an LVM volume?**
A: `lvextend -L +10G /dev/vg/lv` then `resize2fs /dev/vg/lv` (for ext4)

**Q101: What is RAID?**
A: Redundant Array of Independent Disks. Combines multiple disks for performance and/or redundancy.

**Q102: What's the difference between RAID 0, 1, 5, 10?**
A: RAID 0 = striping (fast, no redundancy). RAID 1 = mirroring (redundancy). RAID 5 = striping with parity (1 disk can fail). RAID 10 = mirrored stripes (fast + redundant).

**Q103: What is the `/proc` filesystem?**
A: Virtual filesystem exposing kernel and process information. `/proc/cpuinfo`, `/proc/meminfo`, `/proc/PID/`.

**Q104: What is the `/sys` filesystem?**
A: Virtual filesystem exposing kernel objects, devices, and drivers.

**Q105: How do you check CPU info?**
A: `cat /proc/cpuinfo` or `lscpu`

**Q106: How do you check kernel version?**
A: `uname -r` or `cat /proc/version`

**Q107: What is a kernel module?**
A: Loadable code that extends kernel functionality without reboot. Drivers are often modules.

**Q108: How do you list loaded kernel modules?**
A: `lsmod`

**Q109: How do you load/unload a kernel module?**
A: `modprobe modulename` / `modprobe -r modulename`

**Q110: What is SELinux?**
A: Security-Enhanced Linux. Mandatory Access Control (MAC) system. Enforces security policies beyond traditional permissions.

**Q111: What are SELinux modes?**
A: Enforcing (blocks violations), Permissive (logs but allows), Disabled.

**Q112: How do you check SELinux status?**
A: `getenforce` or `sestatus`

**Q113: What is AppArmor?**
A: Alternative to SELinux (used by Ubuntu/Debian). Profile-based mandatory access control.

**Q114: What is iptables?**
A: Linux firewall. Filters packets based on rules in chains (INPUT, OUTPUT, FORWARD).

**Q115: How do you list iptables rules?**
A: `iptables -L -n -v`

**Q116: How do you allow SSH in iptables?**
A: `iptables -A INPUT -p tcp --dport 22 -j ACCEPT`

**Q117: What is firewalld?**
A: Dynamic firewall manager (RHEL/CentOS). Uses zones and services. Frontend to iptables/nftables.

**Q118: What is nftables?**
A: Modern replacement for iptables. Unified framework for packet filtering.

**Q119: How do you check open ports?**
A: `ss -tuln` or `netstat -tuln`

**Q120: What does `ss` stand for?**
A: Socket Statistics. Modern replacement for netstat.

**Q121: How do you check network interfaces?**
A: `ip addr` or `ifconfig` (deprecated)

**Q122: How do you add an IP address to an interface?**
A: `ip addr add 192.168.1.10/24 dev eth0`

**Q123: How do you check routing table?**
A: `ip route` or `route -n`

**Q124: How do you add a static route?**
A: `ip route add 10.0.0.0/8 via 192.168.1.1`

**Q125: What is `/etc/hosts`?**
A: Local DNS override. Maps hostnames to IPs before querying DNS.

**Q126: What is `/etc/resolv.conf`?**
A: DNS resolver configuration. Lists nameservers to query.

**Q127: How do you test DNS resolution?**
A: `nslookup hostname`, `dig hostname`, or `host hostname`

**Q128: How do you test network connectivity?**
A: `ping hostname` (ICMP) or `telnet hostname port` (TCP)

**Q129: What does `traceroute` do?**
A: Shows the path packets take to reach a destination. Each hop is a router.

**Q130: What is MTU?**
A: Maximum Transmission Unit. Largest packet size that can be sent without fragmentation. Default: 1500 bytes.

**Q131: What is TCP vs UDP?**
A: TCP = connection-oriented, reliable, ordered (HTTP, SSH). UDP = connectionless, fast, no guarantee (DNS, video streaming).

**Q132: What is a socket?**
A: Endpoint for network communication. Combination of IP address and port.

**Q133: What is SSH?**
A: Secure Shell. Encrypted protocol for remote login and command execution. Port 22.

**Q134: How do you generate SSH keys?**
A: `ssh-keygen -t ed25519` or `ssh-keygen -t rsa -b 4096`

**Q135: Where are SSH keys stored?**
A: Private key: `~/.ssh/id_ed25519`. Public key: `~/.ssh/id_ed25519.pub`. Authorized keys: `~/.ssh/authorized_keys`.

**Q136: What is SSH agent forwarding?**
A: Allows using local SSH keys on remote servers. `ssh -A user@host`. Security risk if remote host is compromised.

**Q137: How do you copy files over SSH?**
A: `scp file user@host:/path` or `rsync -avz file user@host:/path`

**Q138: What is rsync?**
A: Efficient file sync tool. Only transfers differences. `rsync -avz source dest`

**Q139: What does `rsync -avz` mean?**
A: -a = archive (preserves permissions, timestamps). -v = verbose. -z = compress during transfer.

**Q140: What is `/etc/ssh/sshd_config`?**
A: SSH server configuration. Controls authentication methods, ports, allowed users.

**Q141: How do you disable root SSH login?**
A: Set `PermitRootLogin no` in `/etc/ssh/sshd_config`, then restart sshd.

**Q142: How do you disable password authentication for SSH?**
A: Set `PasswordAuthentication no` in `/etc/ssh/sshd_config`. Requires key-based auth.

---

## Scenario-Based

**Q143: A server is running out of disk space. How do you investigate?**
A:
```bash
df -h                           # Check which filesystem is full
du -sh /* 2>/dev/null | sort -h # Find large directories
find / -size +100M -type f      # Find large files
journalctl --disk-usage         # Check journal size
docker system df                # If Docker is installed
```
Common culprits: logs, old kernels, Docker images, temp files.

**Q144: A process is using 100% CPU. How do you handle it?**
A:
```bash
top                             # Identify the process
ps aux | grep <PID>             # Get details
strace -p <PID>                 # See what it's doing
kill <PID>                      # Graceful stop
kill -9 <PID>                   # Force if needed
```
Investigate why before killing in production.

**Q145: You can't SSH into a server. What do you check?**
A:
1. Network: `ping server` — is it reachable?
2. Port: `telnet server 22` — is SSH port open?
3. Service: Is sshd running? (check via console)
4. Firewall: iptables/security groups blocking?
5. SSH config: Check `/etc/ssh/sshd_config`
6. Disk space: Full disk can prevent login
7. Auth: Check `~/.ssh/authorized_keys` permissions

**Q146: A cron job isn't running. How do you debug?**
A:
1. Check cron syntax: `crontab -l`
2. Check cron service: `systemctl status cron`
3. Check logs: `grep CRON /var/log/syslog`
4. Check script permissions: Is it executable?
5. Check PATH: Cron has minimal PATH
6. Check output: Redirect to file to see errors
7. Test manually: Run the command as the cron user

**Q147: A service won't start. How do you troubleshoot?**
A:
```bash
systemctl status servicename    # Check status and recent logs
journalctl -u servicename -n 50 # More detailed logs
systemctl cat servicename       # View unit file
/path/to/binary --help          # Check if binary works
```
Common issues: missing dependencies, permission errors, port already in use, config syntax error.

**Q148: Server is slow. Where do you start?**
A:
```bash
uptime                          # Load average
top / htop                      # CPU and memory hogs
iostat -x 1                     # Disk I/O
vmstat 1                        # Memory and swap
ss -s                           # Connection count
dmesg | tail                    # Kernel errors
```
Identify bottleneck: CPU, memory, disk I/O, or network.

**Q149: You need to find which process is listening on port 8080.**
A: `ss -tlnp | grep 8080` or `lsof -i :8080`

**Q150: A file was accidentally deleted. Can you recover it?**
A: If process still has it open: `lsof | grep deleted`, then `cp /proc/<PID>/fd/<FD> /recovered/file`. Otherwise, need filesystem recovery tools (extundelete) or backups. Prevention: regular backups.

**Q151: How do you check if a server was rebooted unexpectedly?**
A:
```bash
last reboot                     # Reboot history
uptime                          # Current uptime
journalctl --list-boots         # Boot history
dmesg | head                    # Boot messages
cat /var/log/wtmp               # Login records
```

**Q152: How do you find all files owned by a specific user?**
A: `find / -user username 2>/dev/null`

**Q153: How do you find all SUID files (security audit)?**
A: `find / -perm -4000 -type f 2>/dev/null`

**Q154: A user can't write to a directory they should have access to. What do you check?**
A:
1. Directory permissions: `ls -la`
2. User's groups: `id username`
3. ACLs: `getfacl directory`
4. SELinux/AppArmor: `ls -Z`, check audit log
5. Filesystem: Is it mounted read-only? `mount | grep <fs>`
6. Disk space: Is it full?

**Q155: How do you securely transfer a file between two servers?**
A: `scp file user@host:/path` or `rsync -avz -e ssh file user@host:/path`. For large transfers, rsync is better (resumable, only transfers changes).

---

*Master these and you'll handle any Linux situation thrown at you! 🐧*
