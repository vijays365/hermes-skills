---
name: system-health
description: VPS system health monitoring — collect memory, CPU, disk stats and optionally report via email. Includes a standalone script for cron integration.
---

# System Health Monitoring

Monitor VPS resource usage (memory, CPU, disk) and optionally email reports via any SMTP or API-based email service.

## Quick check (ad-hoc)

```bash
# Memory
free -h
echo "---"
# CPU load
cat /proc/loadavg
echo "---"
# Disk
df -h /
echo "---"
# Uptime
uptime
echo "---"
# Top processes by memory
ps aux --sort=-%mem | head -6
```

## Automated reporting script

A standalone Python script collects all stats and sends an email report. Configure paths and recipients at the top of the script.

### What it checks

| Metric | Source | Default threshold |
|--------|--------|-------------------|
| Memory usage | `free -m` | Warn if >85% used |
| Swap usage | `free -m` | Warn if >20% used |
| CPU load (1/5/15min) | `/proc/loadavg` | Warn if 1min > nproc×0.8 |
| Disk usage | `df -h` for `/` | Warn if >80% |
| Uptime | `uptime` | Info only |
| Top 5 memory consumers | `ps` | Info only |

### Script location

`~/.hermes/scripts/system-health-report.py` (after install)

The script uses [AgentMail API](https://docs.agentmail.to) by default. Modify the `send_email()` function to use any other email provider.

### Cron usage

For recurring reports (e.g. twice a week):

```bash
cronjob(action='create', script='~/.hermes/scripts/system-health-report.py', no_agent=True, schedule='0 8 * * 2,5')
```

The script is self-contained — no LLM tokens needed on each run.

## Thresholds and alerts

If any metric exceeds its warning threshold, the subject line gets a ⚠️ prefix. Otherwise it's a ✅.

## Configuration

Edit the top of the script to set:

- `TO_EMAIL` — recipient email address
- `FROM_EMAIL` — your sender inbox (AgentMail address)
- `API_KEY_FILE` — path to your AgentMail API key file
- `THRESHOLDS` — adjust warning levels

## Pitfalls

- The script needs a valid email API key (AgentMail or other). Store it outside version control.
- The recipient email and sender inbox are configurable at the top of the script.
- The script exits 0 on success — cron only reports non-empty stdout on delivery.
