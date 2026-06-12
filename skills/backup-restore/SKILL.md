---
name: backup-restore
description: Backup and restore crucial Hermes configs, profiles, skills, and data. Push backups to git with weekly cron.
---

# Backup & Restore

Create dated tarball backups of your Hermes setup and push them to a git remote.

## Backup sources

| Source | Path (configurable) | Included |
|--------|---------------------|----------|
| Hermes home | `$HERMES_HOME` or `~/.hermes/` | config, SOUL, profiles, skills |
| Data directories | e.g. `/opt/obsidian-vault`, `/opt/data/scripts` | Add yours in the script |

## Excluded (secrets / temp)

- `*_key.txt`, `*_credentials`
- `.env` / `.env.*`
- `__pycache__` / `*.pyc`
- `.git/` directories
- `node_modules/`

## Automated backup script

`~/.hermes/scripts/backup.sh` (after install) — creates tarball, commits to a `backups/` directory in your git repo, pushes.

Run manually:

```bash
bash ~/.hermes/scripts/backup.sh
```

Cron schedule (weekly, Sunday 2AM):

```bash
cronjob(action='create', script='~/.hermes/scripts/backup.sh', no_agent=True, schedule='0 2 * * 0')
```

## What the script does

1. Creates a timestamped tarball: `backups/hermes-backup-YYYY-MM-DD-HHMMSS.tar.gz`
2. Saves a manifest alongside it listing sources and sizes
3. `git add`, `git commit`, `git push`
4. Prunes backups older than 4 weeks

## Restore procedure

1. List available backups:
   ```bash
   ls backups/hermes-backup-*.tar.gz
   ```

2. Extract to a temp directory:
   ```bash
   tar xzf backups/hermes-backup-YYYY-MM-DD-HHMMSS.tar.gz -C /tmp/restore/
   ```

3. Copy back what's needed:
   ```bash
   cp -r /tmp/restore/config.yaml $HERMES_HOME/
   cp -r /tmp/restore/profiles/* $HERMES_HOME/profiles/
   cp -r /tmp/restore/skills/* $HERMES_HOME/skills/
   ```

4. Restore cron jobs from the manifest (re-create via cronjob tool).

5. Clean up: `rm -rf /tmp/restore`

## Configuration

Edit the top of `backup.sh`:

- `REPO_DIR` — path to your git repository
- `SOURCES` — array of paths to include in backup
- `RETENTION_WEEKS` — how long to keep old backups (default: 4)

## Pitfalls

- **Secrets are NOT backed up.** After restore, manually re-add API keys and credentials.
- **Git push fails if offline.** Backup still exists locally; push retries next week.
- **Backups older than retention period are auto-pruned.** Adjust `RETENTION_WEEKS` in the script.
- **Script assumes a git repo exists** at `REPO_DIR` with a configured remote.
- **Don't restore a config file while Hermes is running.** Restart Hermes after config restore.
