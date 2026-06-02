# Linux — Cron Jobs & Scheduling — Deep Dive

---

## CRONTAB

```bash
# Edit your crontab
crontab -e                       # Edit current user's crontab
sudo crontab -e                  # Edit root's crontab
crontab -l                       # List current crontab
crontab -r                       # Remove all cron jobs (careful!)

# Format:
# ┌───── minute (0-59)
# │ ┌───── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌───── month (1-12)
# │ │ │ │ ┌───── day of week (0-6, Sun=0)
# │ │ │ │ │
# * * * * *  command
```

---

## EXAMPLES

```bash
# Every minute
* * * * *  /opt/scripts/health-check.sh

# Every 5 minutes
*/5 * * * *  /opt/scripts/monitor.sh

# Every hour at minute 0
0 * * * *  /opt/scripts/sync.sh

# Daily at 2 AM
0 2 * * *  /opt/scripts/backup.sh

# Daily at midnight
0 0 * * *  /opt/scripts/cleanup.sh

# Every Monday at 9 AM
0 9 * * 1  /opt/scripts/weekly-report.sh

# Every Sunday at midnight
0 0 * * 0  /opt/scripts/weekly-cleanup.sh

# First day of every month at 3 AM
0 3 1 * *  /opt/scripts/monthly-report.sh

# Every weekday (Mon-Fri) at 8 AM
0 8 * * 1-5  /opt/scripts/daily-check.sh

# Every 15 minutes during business hours
*/15 8-17 * * 1-5  /opt/scripts/business-monitor.sh
```

---

## BEST PRACTICES

```bash
# Always redirect output (otherwise cron emails it)
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Use full paths (cron has minimal PATH)
0 * * * * /usr/bin/python3 /opt/scripts/check.py

# Set PATH at top of crontab
PATH=/usr/local/bin:/usr/bin:/bin
0 2 * * * backup.sh >> /var/log/backup.log 2>&1

# Set MAILTO for error notifications
MAILTO=ops@example.com
0 2 * * * /opt/scripts/backup.sh

# Lock to prevent overlapping runs
* * * * * flock -n /tmp/job.lock /opt/scripts/long-job.sh
```

---

## SYSTEM CRON DIRECTORIES

```
/etc/crontab           ← System crontab (includes user field)
/etc/cron.d/           ← Drop-in cron files
/etc/cron.daily/       ← Scripts run daily
/etc/cron.hourly/      ← Scripts run hourly
/etc/cron.weekly/      ← Scripts run weekly
/etc/cron.monthly/     ← Scripts run monthly

# Just drop a script in /etc/cron.daily/ and it runs daily
sudo cp backup.sh /etc/cron.daily/
sudo chmod +x /etc/cron.daily/backup.sh
```

---

## SYSTEMD TIMERS (Modern Alternative)

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```
```bash
sudo systemctl enable backup.timer
sudo systemctl start backup.timer
systemctl list-timers                    # List all timers
```

---

## CHECKLIST
- [ ] Create a crontab entry for a daily backup at 2 AM
- [ ] Redirect cron output to a log file
- [ ] Use flock to prevent overlapping cron jobs
