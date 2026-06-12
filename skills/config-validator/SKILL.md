---
name: config-validator
description: Validate Hermes config YAML files — main config, profile configs, required fields, skill references, and path integrity.
---

# Config Validator

Validate all Hermes configuration YAML files for structural and semantic correctness.

## Quick validation (ad-hoc)

```bash
# Check main config is valid YAML
python3 -c "import yaml; yaml.safe_load(open('$HERMES_HOME/config.yaml')); print('config.yaml: OK')"

# Validate all profile configs
for p in $HERMES_HOME/profiles/*/; do
  name=$(basename "$p")
  if [ -f "${p}config.yaml" ]; then
    python3 -c "import yaml; yaml.safe_load(open('${p}config.yaml')); print('${name}/config.yaml: OK')"
  else
    echo "WARNING: ${name} has no config.yaml"
  fi
done
```

## Validation checks

| Check | What it validates |
|-------|------------------|
| YAML syntax | Every `.yaml`/`.yml` file parses without error |
| Model config | `model.default` and `model.provider` are present |
| Profile existence | Each profile directory has `config.yaml`, `profile.yaml`, `SOUL.md` |
| Gateway states | `gateway_state.json` is valid JSON with expected keys |
| Skills directory | Skills dir exists and is non-empty for profiles that use them |
| Path existence | All paths in configs resolve on disk |

## Automated check (cron-friendly)

`~/.hermes/scripts/validate-config.sh` — runs all checks and reports findings.

```bash
bash ~/.hermes/scripts/validate-config.sh
```

## Output format

Prints OK/WARNING/ERROR per check. Exits 0 if clean, 1 if warnings, 2 if errors.

## Configuration

The script uses `$HERMES_HOME` (defaults to `~/.hermes`) to find configs, profiles, and gateway states. Override the env var for non-standard setups.

## Prerequisites

- Python 3 with `pyyaml` installed: `pip install pyyaml` or use `uv run --with pyyaml python3`
- `uv` (recommended) — used by the script for isolated YAML parsing

## Pitfalls

- `yaml` Python package must be installed. If missing: `pip install pyyaml`.
- Profile configs are partial (inherit from main). Missing optional fields there are OK.
- `gateway_state.json` uses the key `gateway_state`, not `state`.
- The script reads from `$HERMES_HOME` — ensure this env var is set correctly, or edit the script path directly.
