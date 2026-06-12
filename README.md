# Hermes Skills

Community skills for [Hermes Agent](https://hermes-agent.nousresearch.com) — the open-source AI agent by Nous Research.

## Skills

| Skill | Description |
|-------|-------------|
| [system-health](skills/system-health/) | VPS health monitoring: memory, CPU, disk stats with optional email reports |
| [backup-restore](skills/backup-restore/) | Dated tarball backups of Hermes configs/profiles/skills, pushed to git |
| [hermes-diag](skills/hermes-diag/) | One-shot diagnostics: gateways, config, disk, memory, errors |
| [config-validator](skills/config-validator/) | Validate all Hermes YAML configs for syntax and structure |

## Installation

Add the tap:

```bash
hermes skills tap add vijays365/hermes-skills
hermes skills install vijays365/hermes-skills/system-health
```

Or install individual skills directly:

```bash
hermes skills install vijays365/hermes-skills/system-health
hermes skills install vijays365/hermes-skills/backup-restore
hermes skills install vijays365/hermes-skills/hermes-diag
hermes skills install vijays365/hermes-skills/config-validator
```

## Requirements

- Hermes Agent (any recent version)
- Linux (scripts use `/proc`, `free`, `df` — Linux-specific)
- Python 3 for script-based skills

## License

MIT
