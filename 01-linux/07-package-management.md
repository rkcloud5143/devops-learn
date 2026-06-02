# Linux — Package Management — Deep Dive

---

## PACKAGE MANAGERS

```
┌──────────────────────┬──────────────────────────────────────┐
│ Distro               │ Package Manager                      │
├──────────────────────┼──────────────────────────────────────┤
│ Ubuntu / Debian      │ apt (apt-get)                        │
│ Amazon Linux / RHEL  │ yum / dnf                            │
│ Alpine               │ apk (used in Docker)                 │
└──────────────────────┴──────────────────────────────────────┘
```

### apt (Ubuntu/Debian)
```bash
# Update package list (always do this first)
sudo apt update

# Install
sudo apt install nginx                # Install package
sudo apt install -y nginx             # Auto-yes (for scripts)

# Remove
sudo apt remove nginx                 # Remove package
sudo apt purge nginx                  # Remove + config files
sudo apt autoremove                   # Remove unused dependencies

# Upgrade
sudo apt upgrade                      # Upgrade all packages
sudo apt full-upgrade                 # Upgrade + handle dependencies

# Search
apt search nginx                      # Find packages
apt show nginx                        # Package details
dpkg -l | grep nginx                  # List installed packages
```

### yum / dnf (Amazon Linux / RHEL)
```bash
sudo yum update                       # Update all packages
sudo yum install nginx                # Install
sudo yum remove nginx                 # Remove
sudo yum search nginx                 # Search
yum list installed | grep nginx       # List installed
sudo yum install -y nginx             # Auto-yes

# dnf is the modern replacement for yum (same syntax)
sudo dnf install nginx
```

### apk (Alpine — common in Docker)
```bash
apk update                            # Update index
apk add nginx                         # Install
apk del nginx                         # Remove
apk add --no-cache nginx              # Install without cache (Docker)
```

---

## INSTALLING SOFTWARE FROM SOURCE

```bash
# When package isn't in the repo
wget https://example.com/app-1.0.tar.gz
tar -xzf app-1.0.tar.gz
cd app-1.0
./configure
make
sudo make install

# Or use pre-built binaries
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip
unzip terraform_1.7.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform --version
```

---

## COMMON TOOLS TO INSTALL ON A NEW SERVER

```bash
# Ubuntu
sudo apt update && sudo apt install -y \
  curl wget git vim htop tree \
  net-tools dnsutils jq unzip \
  build-essential

# Amazon Linux
sudo yum install -y \
  curl wget git vim htop tree \
  net-tools bind-utils jq unzip

# What each tool does:
# curl/wget  → HTTP requests / downloads
# git        → Version control
# vim        → Text editor
# htop       → Process monitor
# tree       → Directory tree view
# net-tools  → netstat, ifconfig
# dnsutils   → nslookup, dig
# jq         → JSON processor (VERY useful for AWS CLI output)
# unzip      → Extract zip files
```

---

## CHECKLIST
- [ ] Install and remove packages with apt or yum
- [ ] Know the difference between apt and yum
- [ ] Install a binary tool (like Terraform) manually
