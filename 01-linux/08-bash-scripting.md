# Linux — Bash Scripting — Deep Dive

---

## SCRIPT BASICS

```bash
#!/bin/bash
# ^ shebang line — tells OS to use bash

# Make script executable
chmod +x script.sh
./script.sh                      # Run it
bash script.sh                   # Or run with bash directly
```

---

## VARIABLES

```bash
# Assign (NO spaces around =)
NAME="DevOps"
PORT=3000
LOG_FILE="/var/log/app.log"

# Use
echo "Hello $NAME"
echo "Running on port ${PORT}"   # Braces for clarity

# Command substitution
TODAY=$(date +%Y-%m-%d)
HOSTNAME=$(hostname)
IP=$(curl -s ifconfig.me)
FILE_COUNT=$(ls /tmp | wc -l)

# Read-only
readonly DB_HOST="prod-db.internal"

# Environment variables (available to child processes)
export APP_ENV="production"
```

---

## CONDITIONALS

```bash
# if/elif/else
if [ "$ENV" = "production" ]; then
    echo "Deploying to PROD"
elif [ "$ENV" = "staging" ]; then
    echo "Deploying to STAGING"
else
    echo "Unknown environment"
fi

# File tests
if [ -f "/etc/nginx/nginx.conf" ]; then   # File exists
if [ -d "/var/log" ]; then                 # Directory exists
if [ -r "$FILE" ]; then                    # File is readable
if [ -w "$FILE" ]; then                    # File is writable
if [ -x "$FILE" ]; then                    # File is executable
if [ -s "$FILE" ]; then                    # File is not empty
if [ ! -f "$FILE" ]; then                 # File does NOT exist

# String tests
if [ -z "$VAR" ]; then                     # String is empty
if [ -n "$VAR" ]; then                     # String is not empty
if [ "$A" = "$B" ]; then                   # Strings are equal
if [ "$A" != "$B" ]; then                  # Strings not equal

# Number tests
if [ "$COUNT" -eq 0 ]; then               # Equal
if [ "$COUNT" -ne 0 ]; then               # Not equal
if [ "$COUNT" -gt 10 ]; then              # Greater than
if [ "$COUNT" -lt 5 ]; then               # Less than
if [ "$COUNT" -ge 10 ]; then              # Greater or equal
if [ "$COUNT" -le 5 ]; then               # Less or equal

# Combine conditions
if [ "$ENV" = "prod" ] && [ "$READY" = "true" ]; then   # AND
if [ "$ENV" = "dev" ] || [ "$ENV" = "test" ]; then       # OR
```

---

## LOOPS

```bash
# For loop
for server in web1 web2 web3; do
    echo "Checking $server..."
    ping -c 1 "$server"
done

# For loop with range
for i in {1..10}; do
    echo "Iteration $i"
done

# For loop with command output
for file in /var/log/*.log; do
    echo "Size of $file: $(du -sh "$file" | awk '{print $1}')"
done

# C-style for loop
for ((i=0; i<5; i++)); do
    echo "Count: $i"
done

# While loop
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Read file line by line
while IFS= read -r line; do
    echo "Processing: $line"
done < servers.txt

# Infinite loop (useful for monitoring)
while true; do
    curl -s http://localhost:3000/health || echo "APP DOWN!"
    sleep 10
done
```

---

## FUNCTIONS

```bash
# Define function
check_service() {
    local service=$1                     # Local variable
    if systemctl is-active --quiet "$service"; then
        echo "✓ $service is running"
        return 0
    else
        echo "✗ $service is DOWN!"
        return 1
    fi
}

# Call function
check_service nginx
check_service docker

# Function with return value
get_cpu_usage() {
    top -bn1 | grep "Cpu(s)" | awk '{print $2}'
}
CPU=$(get_cpu_usage)
echo "CPU: ${CPU}%"
```

---

## INPUT & ARGUMENTS

```bash
# Script arguments
# ./deploy.sh production v2.1.0
echo "Environment: $1"          # production
echo "Version: $2"              # v2.1.0
echo "Script name: $0"          # ./deploy.sh
echo "All args: $@"             # production v2.1.0
echo "Arg count: $#"            # 2

# Validate arguments
if [ $# -lt 2 ]; then
    echo "Usage: $0 <environment> <version>"
    exit 1
fi

# Read user input
read -p "Enter environment (dev/prod): " ENV
read -sp "Enter password: " PASSWORD    # -s = silent (hidden)
echo ""
```

---

## ERROR HANDLING

```bash
# Exit on error
set -e                           # Exit immediately if command fails
set -u                           # Exit if undefined variable used
set -o pipefail                  # Exit if any pipe command fails
set -euo pipefail                # All three (USE THIS in every script)

# Exit codes
exit 0                           # Success
exit 1                           # General error
$?                               # Exit code of last command

# Trap — run cleanup on exit
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/deploy-*
}
trap cleanup EXIT                # Run cleanup when script exits

# Error handling pattern
deploy() {
    echo "Deploying..."
    if ! docker pull myapp:latest; then
        echo "ERROR: Failed to pull image"
        exit 1
    fi
    echo "Deploy successful"
}
```

---

## REAL-WORLD SCRIPTS

### Health Check Script
```bash
#!/bin/bash
set -euo pipefail

SERVICES=("nginx" "docker" "myapp")
FAILED=0

for svc in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$svc"; then
        echo "✓ $svc"
    else
        echo "✗ $svc is DOWN"
        FAILED=$((FAILED + 1))
    fi
done

if [ $FAILED -gt 0 ]; then
    echo "WARNING: $FAILED service(s) down"
    exit 1
fi
echo "All services healthy"
```

### Log Cleanup Script
```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
DAYS=7

echo "Cleaning logs older than $DAYS days in $LOG_DIR"
find "$LOG_DIR" -name "*.log" -mtime +$DAYS -delete
echo "Done. Current disk usage:"
df -h /
```

### Backup Script
```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
DATE=$(date +%Y-%m-%d_%H%M)
DB_NAME="myapp"

mkdir -p "$BACKUP_DIR"
pg_dump "$DB_NAME" | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Keep only last 7 backups
ls -t "$BACKUP_DIR"/${DB_NAME}_*.sql.gz | tail -n +8 | xargs rm -f

echo "Backup complete: ${DB_NAME}_${DATE}.sql.gz"
aws s3 cp "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz" s3://my-backups/
```

---

## CHECKLIST
- [ ] Write a script with arguments, conditionals, and loops
- [ ] Use set -euo pipefail in every script
- [ ] Write a health check script for 3+ services
- [ ] Write a backup script that uploads to S3
- [ ] Use functions to organize code
