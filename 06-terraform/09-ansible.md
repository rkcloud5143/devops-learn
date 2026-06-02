# Ansible — Configuration Management 🔧

Automate server setup, app deployment, and configuration.

---

## What is Ansible?

```
Terraform creates infrastructure (EC2, VPC, RDS).
Ansible CONFIGURES infrastructure (install packages, edit files, start services).

┌─── Terraform ──────────┐     ┌─── Ansible ────────────────────┐
│                         │     │                                 │
│  Creates:               │     │  Configures:                    │
│  - EC2 instances        │────▶│  - Install Docker, Nginx        │
│  - VPCs, subnets        │     │  - Copy config files            │
│  - RDS databases        │     │  - Start services               │
│  - S3 buckets           │     │  - Create users                 │
│                         │     │  - Apply security hardening     │
│  "Build the kitchen"    │     │  "Set up the kitchen"           │
└─────────────────────────┘     └─────────────────────────────────┘
```

---

## How Ansible Works

```
┌─── Your Laptop (Control Node) ──────────────────┐
│                                                   │
│  ansible-playbook deploy.yml                     │
│       │                                           │
│       │  SSH (no agent needed!)                   │
│       │                                           │
│       ├──→ Server 1: Install Nginx, copy config  │
│       ├──→ Server 2: Install Nginx, copy config  │
│       └──→ Server 3: Install Nginx, copy config  │
│                                                   │
│  Key difference from Chef/Puppet:                │
│  ✅ Agentless — just needs SSH                   │
│  ✅ Push-based — you run it when you want        │
│  ✅ YAML — easy to read                          │
└───────────────────────────────────────────────────┘
```

---

## Core Concepts

### Inventory (Which Servers)
```ini
# inventory.ini
[webservers]
web1.example.com
web2.example.com
10.0.1.50

[databases]
db1.example.com

[production:children]
webservers
databases

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/my-key.pem
```

Or dynamic inventory from AWS:
```bash
# Automatically discovers EC2 instances by tags
ansible-inventory -i aws_ec2.yml --list
```

### Playbook (What To Do)
```yaml
# deploy.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes                    # Run as root (sudo)

  vars:
    app_port: 8080
    nginx_version: "1.24"

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install Nginx
      apt:
        name: "nginx={{ nginx_version }}*"
        state: present

    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Restart Nginx

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Add user to docker group
      user:
        name: ubuntu
        groups: docker
        append: yes

    - name: Start and enable Docker
      service:
        name: docker
        state: started
        enabled: yes

    - name: Deploy application container
      docker_container:
        name: my-app
        image: "my-app:{{ app_version }}"
        ports:
          - "{{ app_port }}:8080"
        restart_policy: always

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

### Templates (Jinja2)
```nginx
# templates/nginx.conf.j2
server {
    listen 80;
    server_name {{ ansible_hostname }};

    location / {
        proxy_pass http://localhost:{{ app_port }};
        proxy_set_header Host $host;
    }
}
```

### Roles (Reusable Components)
```
roles/
├── nginx/
│   ├── tasks/main.yml
│   ├── handlers/main.yml
│   ├── templates/nginx.conf.j2
│   ├── files/
│   ├── vars/main.yml
│   └── defaults/main.yml
├── docker/
│   ├── tasks/main.yml
│   └── handlers/main.yml
└── security/
    └── tasks/main.yml
```

```yaml
# site.yml — use roles
---
- hosts: webservers
  become: yes
  roles:
    - security
    - docker
    - nginx
```

---

## Common Modules

```yaml
# Package management
- apt: name=nginx state=present          # Debian/Ubuntu
- yum: name=nginx state=present          # RHEL/CentOS

# Files
- copy: src=file.txt dest=/etc/file.txt
- template: src=config.j2 dest=/etc/config
- file: path=/data state=directory mode='0755'
- lineinfile: path=/etc/hosts line="10.0.1.5 myapp"

# Services
- service: name=nginx state=started enabled=yes
- systemd: name=nginx state=restarted daemon_reload=yes

# Users
- user: name=deploy groups=docker shell=/bin/bash
- authorized_key: user=deploy key="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

# Docker
- docker_container: name=app image=my-app:latest ports=["8080:8080"]
- docker_image: name=my-app source=pull

# Commands
- command: /opt/scripts/deploy.sh
- shell: cat /etc/os-release | grep VERSION
```

---

## Ansible Commands

```bash
# Run playbook
ansible-playbook deploy.yml

# Run with specific inventory
ansible-playbook -i inventory.ini deploy.yml

# Run on specific hosts
ansible-playbook deploy.yml --limit webservers

# Dry run (check mode)
ansible-playbook deploy.yml --check

# Run with extra variables
ansible-playbook deploy.yml -e "app_version=v2.0"

# Ad-hoc commands (quick one-liners)
ansible webservers -m ping                          # Test connectivity
ansible webservers -m shell -a "uptime"             # Run command
ansible webservers -m apt -a "name=nginx state=present" --become

# Encrypt secrets
ansible-vault encrypt secrets.yml
ansible-vault decrypt secrets.yml
ansible-vault edit secrets.yml
ansible-playbook deploy.yml --ask-vault-pass
```

---

## Ansible vs Terraform

| Feature | Ansible | Terraform |
|---------|---------|-----------|
| Purpose | Configuration management | Infrastructure provisioning |
| Approach | Procedural (do this, then that) | Declarative (desired state) |
| State | Stateless | State file |
| Agent | Agentless (SSH) | Agentless (API) |
| Idempotent | Yes (most modules) | Yes |
| Best for | Server config, app deploy | Cloud resources |

**Use both together:** Terraform creates EC2 → Ansible configures it.

---

## When to Use Ansible in 2024+

```
✅ Still relevant for:
  - Configuring VMs/EC2 instances
  - On-premises servers
  - Network device configuration
  - One-time setup tasks
  - Legacy application deployment

⚠️ Less relevant for:
  - Kubernetes (use Helm/ArgoCD instead)
  - Immutable infrastructure (use Packer + Terraform)
  - Container-based apps (use Docker/K8s)
```

---

*Ansible = SSH into servers and configure them, at scale, with YAML! 🔧*
