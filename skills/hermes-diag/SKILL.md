---
name: hermes-diag
description: One-shot Hermes diagnostics — check running profiles, active crons, disk, config validity, gateway states, and recent errors.
---

# Hermes Diagnostics

Single command to assess overall Hermes system health.

## Quick diagnostic

```bash
bash ~/.hermes/scripts/hermes-diag.sh
```

This checks everything below and prints a clean summary.

## What it checks

### 1. Profile & gateway status

List each profile and its gateway state:

```bash
for p in $HERMES_HOME/profiles/*/; do
  name=$(basename "$p")
  state_file="${p}gateway_state.json"
  if [ -f "$state_file" ]; then
    state=$(python3 -c "import json; d=json.load(open('${state_file}')); print(d.get('gateway_state','unknown'))")
    echo "${name}: gateway=${state}"
  else
    echo "${name}: no gateway state"
  fi
done
```

### 2. Active crons

```bash
cronjob(action='list')
```

### 3. Disk usage

```bash
df -h /
```

### 4. Config validity

See `config-validator` skill.

### 5. Memory & uptime

```bash
free -h
uptime
```

### 6. Recent errors (logs)

```bash
grep -i "error\|traceback\|exception" $HERMES_HOME/profiles/*/logs/gateway* 2>/dev/null | tail -20
```

## Output format

```
Status: ✅ (all clear) or ⚠️ (warnings) or ❌ (errors)

Site          Status  Detail
───           ──────  ──────
Config        ✅      main + N profiles valid
Gateway       ✅      default=running, ...
Crons         ✅      N active, 0 failed
Disk          ✅      /  42% used
Memory        ⚠️      78% used
Errors        ✅      no recent errors
```

## Automated cron

For periodic health checks, schedule as a cron:

```bash
bash ~/.hermes/scripts/hermes-diag.sh
```

## Configuration

The script uses `$HERMES_HOME` (defaults to `~/.hermes`) to locate profiles, configs, and logs. Set the env var if your Hermes home is elsewhere.

## Pitfalls

- Log paths depend on profile directories under `$HERMES_HOME/profiles/`.
- Gateway status in `gateway_state.json` may lag a few seconds behind reality.
- The disk check inspects root (`/`) — adjust if Hermes data is on a separate volume.
