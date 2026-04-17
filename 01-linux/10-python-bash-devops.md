# Python & Bash for DevOps 🐍

Scripting skills every DevOps engineer needs.

---

# Bash Scripting (Essential)

## Variables & Basics
```bash
#!/bin/bash
set -euo pipefail    # Exit on error, undefined vars, pipe failures

# Variables
NAME="production"
COUNT=5
TIMESTAMP=$(date +%Y%m%d-%H%M%S)

# String operations
echo "Deploying to ${NAME}"
echo "Name length: ${#NAME}"
echo "Uppercase: ${NAME^^}"

# Arrays
SERVERS=("web1" "web2" "web3")
echo "First: ${SERVERS[0]}"
echo "All: ${SERVERS[@]}"
echo "Count: ${#SERVERS[@]}"

# Conditionals
if [[ "$NAME" == "production" ]]; then
    echo "⚠️  Production deployment!"
elif [[ "$NAME" == "staging" ]]; then
    echo "Staging deployment"
else
    echo "Dev deployment"
fi

# Loops
for server in "${SERVERS[@]}"; do
    echo "Deploying to $server"
done

for i in {1..5}; do
    echo "Attempt $i"
done

# Functions
deploy() {
    local env=$1
    local version=$2
    echo "Deploying $version to $env"
}
deploy "production" "v2.0"
```

## Real DevOps Scripts

### Health Check Script
```bash
#!/bin/bash
set -euo pipefail

URL="https://myapp.example.com/health"
MAX_RETRIES=5
RETRY_INTERVAL=10

for i in $(seq 1 $MAX_RETRIES); do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$URL" || true)
    if [[ "$STATUS" == "200" ]]; then
        echo "✅ Health check passed"
        exit 0
    fi
    echo "⏳ Attempt $i/$MAX_RETRIES — Status: $STATUS"
    sleep $RETRY_INTERVAL
done

echo "❌ Health check failed after $MAX_RETRIES attempts"
exit 1
```

### Log Cleanup Script
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
DAYS_TO_KEEP=7

echo "Cleaning logs older than $DAYS_TO_KEEP days in $LOG_DIR"

DELETED=$(find "$LOG_DIR" -name "*.log" -mtime +$DAYS_TO_KEEP -delete -print | wc -l)
echo "Deleted $DELETED files"

# Also compress recent logs
find "$LOG_DIR" -name "*.log" -mtime +1 ! -name "*.gz" -exec gzip {} \;
echo "Compressed logs older than 1 day"
```

### Deployment Script
```bash
#!/bin/bash
set -euo pipefail

ENV=${1:?"Usage: $0 <environment> <version>"}
VERSION=${2:?"Usage: $0 <environment> <version>"}
CLUSTER="my-cluster"
NAMESPACE="$ENV"

echo "🚀 Deploying $VERSION to $ENV"

# Update kubeconfig
aws eks update-kubeconfig --name "$CLUSTER" --region ca-central-1

# Deploy with Helm
helm upgrade --install my-app ./helm/my-app \
    --namespace "$NAMESPACE" \
    --set image.tag="$VERSION" \
    --wait --timeout 5m

# Verify
kubectl rollout status deployment/my-app -n "$NAMESPACE" --timeout=5m

echo "✅ Deployment complete"
```

### AWS Resource Cleanup
```bash
#!/bin/bash
set -euo pipefail

echo "Finding unattached EBS volumes..."
VOLUMES=$(aws ec2 describe-volumes \
    --filters Name=status,Values=available \
    --query 'Volumes[].VolumeId' \
    --output text)

if [[ -z "$VOLUMES" ]]; then
    echo "No unattached volumes found"
    exit 0
fi

echo "Found volumes: $VOLUMES"
read -p "Delete these volumes? (y/n) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    for vol in $VOLUMES; do
        echo "Deleting $vol"
        aws ec2 delete-volume --volume-id "$vol"
    done
    echo "✅ Done"
fi
```

---

# Python for DevOps

## Why Python?
```
Bash is great for:  Simple scripts, CLI glue, one-liners
Python is better for: Complex logic, API calls, data processing, error handling

Most DevOps tools have Python SDKs:
  - boto3 (AWS)
  - kubernetes (K8s)
  - requests (HTTP)
  - paramiko (SSH)
  - jinja2 (templating)
```

## AWS with boto3
```python
import boto3
from datetime import datetime, timedelta

# List EC2 instances
ec2 = boto3.client('ec2', region_name='ca-central-1')

response = ec2.describe_instances(
    Filters=[{'Name': 'tag:Environment', 'Values': ['production']}]
)

for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        name = next((t['Value'] for t in instance.get('Tags', []) if t['Key'] == 'Name'), 'N/A')
        print(f"{instance['InstanceId']} | {name} | {instance['State']['Name']}")


# Find old snapshots (> 30 days)
ec2 = boto3.client('ec2')
cutoff = datetime.now(tz=None) - timedelta(days=30)

snapshots = ec2.describe_snapshots(OwnerIds=['self'])['Snapshots']
old = [s for s in snapshots if s['StartTime'].replace(tzinfo=None) < cutoff]

print(f"Found {len(old)} snapshots older than 30 days")
for snap in old:
    print(f"  {snap['SnapshotId']} | {snap['StartTime'].date()} | {snap.get('Description', 'N/A')}")
```

## Kubernetes with Python
```python
from kubernetes import client, config

# Load kubeconfig
config.load_kube_config()
v1 = client.CoreV1Api()

# List pods in namespace
pods = v1.list_namespaced_pod(namespace="production")
for pod in pods.items:
    status = pod.status.phase
    restarts = sum(cs.restart_count for cs in (pod.status.container_statuses or []))
    print(f"{pod.metadata.name} | {status} | Restarts: {restarts}")

# Find pods with high restarts
problem_pods = [
    pod for pod in pods.items
    if any(cs.restart_count > 5 for cs in (pod.status.container_statuses or []))
]
if problem_pods:
    print(f"\n⚠️  {len(problem_pods)} pods with >5 restarts!")
```

## HTTP API Calls
```python
import requests
import json

# Health check with retries
def health_check(url, retries=5, interval=10):
    import time
    for i in range(retries):
        try:
            r = requests.get(url, timeout=5)
            if r.status_code == 200:
                print(f"✅ Healthy: {url}")
                return True
        except requests.exceptions.RequestException as e:
            print(f"⏳ Attempt {i+1}/{retries}: {e}")
        time.sleep(interval)
    print(f"❌ Unhealthy: {url}")
    return False

health_check("https://myapp.example.com/health")


# Slack notification
def notify_slack(webhook_url, message, color="good"):
    payload = {
        "attachments": [{
            "color": color,
            "text": message,
            "ts": int(__import__('time').time())
        }]
    }
    requests.post(webhook_url, json=payload)

notify_slack(
    "https://hooks.slack.com/services/xxx/yyy/zzz",
    "🚀 Deployment v2.0 to production complete",
    "good"
)
```

## File Processing (Logs, CSV, JSON)
```python
import json
import re
from collections import Counter

# Parse logs and find top errors
error_pattern = re.compile(r'ERROR.*?:\s*(.*)')
errors = Counter()

with open('/var/log/app.log') as f:
    for line in f:
        match = error_pattern.search(line)
        if match:
            errors[match.group(1)] += 1

print("Top 10 errors:")
for error, count in errors.most_common(10):
    print(f"  {count:5d} | {error[:80]}")


# Process JSON config
with open('config.json') as f:
    config = json.load(f)

# Update and write back
config['version'] = '2.0.0'
with open('config.json', 'w') as f:
    json.dump(config, f, indent=2)
```

## CLI Tool with argparse
```python
#!/usr/bin/env python3
"""DevOps utility script."""
import argparse
import boto3
import sys

def list_instances(env, region):
    ec2 = boto3.client('ec2', region_name=region)
    response = ec2.describe_instances(
        Filters=[{'Name': 'tag:Environment', 'Values': [env]}]
    )
    for r in response['Reservations']:
        for i in r['Instances']:
            name = next((t['Value'] for t in i.get('Tags', []) if t['Key'] == 'Name'), 'N/A')
            print(f"{i['InstanceId']} | {name} | {i['State']['Name']}")

def main():
    parser = argparse.ArgumentParser(description='DevOps CLI')
    sub = parser.add_subparsers(dest='command')

    # list-instances command
    li = sub.add_parser('list-instances')
    li.add_argument('--env', required=True, choices=['dev', 'staging', 'prod'])
    li.add_argument('--region', default='ca-central-1')

    args = parser.parse_args()
    if args.command == 'list-instances':
        list_instances(args.env, args.region)
    else:
        parser.print_help()

if __name__ == '__main__':
    main()
```

```bash
# Usage
python devops_cli.py list-instances --env prod
python devops_cli.py list-instances --env dev --region us-east-1
```

---

## When to Use Bash vs Python

| Scenario | Use |
|----------|-----|
| Simple file operations | Bash |
| Chaining CLI commands | Bash |
| One-liners | Bash |
| API calls | Python |
| Complex logic / error handling | Python |
| Data processing (JSON, CSV) | Python |
| Reusable CLI tools | Python |
| Cron jobs (simple) | Bash |
| AWS automation | Python (boto3) |

**Rule of thumb:** If the script is > 50 lines or has complex logic, use Python.

---

*Scripting is the glue that holds DevOps together! 🐍*
