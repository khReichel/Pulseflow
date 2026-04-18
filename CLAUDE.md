# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important

The `./repo` directory is a symlink to an external git repository used as analysis input. **Do not read, scan, or explore it.**

## Project Overview

**Calyntro** is a repository analytics platform that extracts and interprets git history to surface software evolution patterns, code quality risks, and team dynamics. It is distributed as pre-built Docker images; the source code for backend and UI lives in separate repositories.

This repository contains the **deployment configuration and tooling** only.

## Common Commands

All operations go through `./manage.sh`:

```bash
./manage.sh up [prod|dev]   # Start services (default: prod)
./manage.sh down            # Stop and remove containers
./manage.sh import [path]   # Run import + analysis (uses config.yaml if path omitted)
./manage.sh refresh         # Atomic swap staging DB → production (zero-downtime)
./manage.sh update          # Rebuild and restart with --remove-orphans
./manage.sh pull-latest     # Pull latest images from ghcr.io
./manage.sh logs            # Stream logs to ./logs/combined_YYYYMMDD.log
```

Generate a draft config for a new repository:

```bash
./scripts/generate_config.sh /path/to/target/repo [--branch BRANCH] [--since YYYY-MM-DD] [-o config/config.yaml]
```

## Architecture

```
http://localhost:8765
        │
   Nginx Gateway (nginx.conf)
   /api/* → backend:8000
   /*     → ui:80
        │
   ┌────┴──────────┐
   │               │
Backend (Python)  UI (Web)
        │
   DuckDB files (./data/)
        ↑
   Importer (on-demand container)
   - Git history extraction
   - Code metrics (cyclomatic/cognitive complexity, LOC)
   - Writes to staging DB (./data/update_data/)
```

`./manage.sh refresh` atomically swaps the staging DB into production and creates a timestamped backup under `./data/backup_YYYYMMDD_HHMMSS/`.

**Container images** (pre-built, pulled from ghcr.io):
- `ghcr.io/khreichel/calyntro-backend:latest`
- `ghcr.io/khreichel/calyntro-ui:latest`

## Configuration

`config/config.yaml` is the active config (gitignored). Use `config/config_example.yaml` as a template.

Key fields:
```yaml
project:
  repo_path: "/repo"         # Mount point inside container — keep as /repo
  branch: "master"
  analysis_since: "YYYY-MM-DD"

analysis:
  components:                # Map directory prefixes to logical modules
  excluded_prefixes:
  excluded_extensions:
  excluded_patterns:         # Regex patterns
  users:                     # Developers with email aliases
  teams:
  team_memberships:          # Historical team assignments with valid_from/valid_to
```

## Data Layout

```
data/
  calyntro.duckdb            # Production analysis DB
  calyntro_config.duckdb     # Config/metadata DB
  update_data/               # Staging DB (written by importer)
  backup_*/                  # Timestamped backups created by refresh
exports/                     # Data export outputs
grafana/data/                # Grafana persistent data
```
