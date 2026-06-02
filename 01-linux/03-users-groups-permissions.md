# Linux — Users, Groups & Permissions — Deep Dive

---

## USERS

```
┌─── Linux Users ──────────────────────────────────────────────┐
│                                                               │
│  root (UID 0):                                               │
│  - Superuser, can do ANYTHING                                │
│  - Like AWS root account — avoid using directly              │
│  - Use sudo instead                                          │
│                                                               │
│  System users (UID 1-999):                                   │
│  - Created for services (nginx, mysql, docker)               │
│  - Cannot login interactively                                │
│  - Example: www-data runs nginx                              │
│                                                               │
│  Regular users (UID 1000+):                                  │
│  - Human users (ubuntu, ec2-user, your-name)                 │
│  - Can login, have home directory                            │
│                                                               │
│  Key files:                                                  │
│  /etc/passwd  → user accounts (name:x:UID:GID:info:home:shell)│
│  /etc/shadow  → encrypted passwords (only root can read)     │
│  /etc/group   → group definitions                            │
└───────────────────────────────────────────────────────────────┘
```

### User Commands
```bash
# View current user
whoami                          # ubuntu
id                              # uid=1000(ubuntu) gid=1000(ubuntu) groups=...

# Create user
sudo useradd -m -s /bin/bash devops    # -m=create home, -s=set shell
sudo passwd devops                      # Set password

# Modify user
sudo usermod -aG docker devops         # Add to docker group
sudo usermod -s /bin/zsh devops        # Change shell
sudo usermod -L devops                 # Lock account (disable login)
sudo usermod -U devops                 # Unlock account

# Delete user
sudo userdel devops                    # Delete user
sudo userdel -r devops                 # Delete user + home directory

# Switch user
su - devops                            # Switch to devops user
sudo -i                                # Switch to root
exit                                   # Go back
```

---

## GROUPS

```
┌─── Linux Groups ─────────────────────────────────────────────┐
│                                                               │
│  Primary group: assigned at user creation (usually same name)│
│  Secondary groups: additional groups user belongs to         │
│                                                               │
│  Example:                                                    │
│  User "devops" → primary: devops, secondary: docker, sudo   │
│                                                               │
│  Common groups:                                              │
│  ┌──────────┬────────────────────────────────────────────┐  │
│  │ sudo     │ Can run commands as root (Ubuntu)           │  │
│  │ wheel    │ Can run commands as root (RHEL/Amazon Linux)│  │
│  │ docker   │ Can run docker commands without sudo        │  │
│  │ www-data │ Web server group                            │  │
│  │ adm      │ Can read log files                          │  │
│  └──────────┴────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

### Group Commands
```bash
sudo groupadd devteam                  # Create group
sudo groupdel devteam                  # Delete group
sudo usermod -aG devteam devops        # Add user to group
groups devops                          # List user's groups
cat /etc/group | grep devteam          # View group members
```

---

## PERMISSIONS — Deep Dive

```
ls -la output:
-rwxr-xr-- 1 ubuntu devops 4096 Jan 15 10:30 deploy.sh
│├──┤├──┤├──┤  │      │       │
│ │   │   │    │      │       └── file size
│ │   │   │    │      └── group owner
│ │   │   │    └── user owner
│ │   │   └── others permissions (r--)
│ │   └── group permissions (r-x)
│ └── user/owner permissions (rwx)
└── file type (- = file, d = directory, l = symlink)

Permission meanings:
┌──────┬──────────────────────┬──────────────────────────┐
│      │ For FILES            │ For DIRECTORIES          │
├──────┼──────────────────────┼──────────────────────────┤
│ r(4) │ Read file contents   │ List directory contents  │
│ w(2) │ Modify file          │ Create/delete files in it│
│ x(1) │ Execute as program   │ Enter directory (cd)     │
└──────┴──────────────────────┴──────────────────────────┘
```

### Numeric (Octal) Permissions
```
Calculate: add up r(4) + w(2) + x(1)

┌───────┬───────┬──────────────────────────────────────────┐
│ Number│ Perms │ Meaning                                  │
├───────┼───────┼──────────────────────────────────────────┤
│ 7     │ rwx   │ Read + Write + Execute                   │
│ 6     │ rw-   │ Read + Write                             │
│ 5     │ r-x   │ Read + Execute                           │
│ 4     │ r--   │ Read only                                │
│ 3     │ -wx   │ Write + Execute                          │
│ 2     │ -w-   │ Write only                               │
│ 1     │ --x   │ Execute only                             │
│ 0     │ ---   │ No permissions                           │
└───────┴───────┴──────────────────────────────────────────┘

Common permission sets:
┌───────┬──────────────────────────────────────────────────┐
│ 755   │ rwxr-xr-x  Scripts, directories (owner full,    │
│       │            others read+execute)                   │
│ 644   │ rw-r--r--  Config files (owner read/write,       │
│       │            others read only)                      │
│ 600   │ rw-------  Private keys, secrets (owner only)    │
│ 700   │ rwx------  Private directories, .ssh/            │
│ 777   │ rwxrwxrwx  NEVER use this (everyone full access) │
│ 400   │ r--------  SSH private keys (read-only, owner)   │
└───────┴──────────────────────────────────────────────────┘
```

### Permission Commands
```bash
# Change permissions
chmod 755 script.sh              # Numeric
chmod u+x script.sh              # Add execute for user
chmod g-w file.txt               # Remove write for group
chmod o-rwx secret.key           # Remove all for others
chmod -R 755 /var/www/           # Recursive

# Change ownership
chown ubuntu file.txt            # Change user owner
chown ubuntu:devops file.txt     # Change user and group
chown -R ubuntu:ubuntu /app/     # Recursive
chgrp devops file.txt            # Change group only

# Special permissions
chmod u+s /usr/bin/program       # SUID: run as file owner
chmod g+s /shared/               # SGID: inherit group
chmod +t /tmp/                   # Sticky bit: only owner can delete
```

### SUDO
```bash
# Run command as root
sudo apt update                  # Single command as root
sudo -i                          # Full root shell
sudo -u postgres psql            # Run as specific user

# Sudoers file (/etc/sudoers — edit with visudo ONLY)
# Format: user  host=(runas) commands
ubuntu  ALL=(ALL) NOPASSWD: ALL  # ubuntu can sudo anything, no password
devops  ALL=(ALL) /usr/bin/systemctl  # devops can only sudo systemctl

# Check sudo access
sudo -l                          # List what you can sudo
```

---

## SSH (Secure Shell)

```
┌─── SSH Connection ───────────────────────────────────────────┐
│                                                               │
│  Your laptop ──── SSH (port 22) ────► EC2 instance           │
│                   encrypted tunnel                           │
│                                                               │
│  Authentication methods:                                     │
│  1. Key pair (recommended):                                  │
│     Private key (on your laptop) + Public key (on server)    │
│  2. Password (less secure, disabled by default on EC2)       │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### SSH Commands
```bash
# Connect to server
ssh ubuntu@54.23.1.100                    # Using default key
ssh -i ~/.ssh/my-key.pem ubuntu@54.23.1.100  # Specific key
ssh -p 2222 ubuntu@server.com             # Custom port

# Key management
ssh-keygen -t ed25519 -C "your@email.com" # Generate key pair
ssh-copy-id ubuntu@server.com             # Copy public key to server
cat ~/.ssh/id_ed25519.pub                 # View public key

# SSH config file (~/.ssh/config) — saves typing
# Host myserver
#     HostName 54.23.1.100
#     User ubuntu
#     IdentityFile ~/.ssh/my-key.pem
#
# Then just: ssh myserver

# File permissions for SSH (MUST be correct or SSH refuses)
chmod 700 ~/.ssh/                         # Directory
chmod 600 ~/.ssh/id_ed25519              # Private key
chmod 644 ~/.ssh/id_ed25519.pub          # Public key
chmod 600 ~/.ssh/authorized_keys         # Server-side

# SCP — copy files over SSH
scp file.txt ubuntu@server:/tmp/          # Upload
scp ubuntu@server:/var/log/app.log .      # Download
scp -r mydir/ ubuntu@server:/opt/         # Upload directory
```

---

## CHECKLIST
- [ ] Create a user, add to groups, set permissions
- [ ] Understand 755, 644, 600 without looking up
- [ ] SSH into an EC2 instance with a key pair
- [ ] Use sudo properly (not everything as root)
