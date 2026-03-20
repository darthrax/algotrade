---
project_name: 'algotrade'
user_name: 'Darthrax'
date: '2026-03-20T14:51:05+05:30'
sections_completed: ['technology_stack']
existing_patterns_found: 5
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

_Intermediate mode: documented after discovery phase_

### Core Technologies (planned / architecture-specified)
- FastAPI backend (`TBD` exact version)
- Postgres with TimescaleDB for tick/time-series persistence (`TBD`)
- Alembic for DB migrations (`TBD`)
- APScheduler for scheduled jobs (`TBD`)
- Dockerized local development (planned `docker-compose.yml`, `Dockerfile.api`, `Dockerfile.worker`) (`TBD`)

### Key Dependencies (integration & operational constraints)
- ICICI Breeze integration via:
  - WebSocket for live incremental data (`breeze-connect` official SDK, `TBD` version)
  - REST for bootstrap, gap-fill, and fallback polling (`breeze-connect` REST methods, `TBD` version)
- Deployment constraint: static IP mandatory for Breeze API-based trading starting April 2026

### Version/compatibility note for agents
- Until lockfiles/manifests exist in-repo, treat “exact versions” as `TBD` and rely on the architecture contracts and module boundaries as the compatibility source of truth.

## Critical Implementation Rules

_Documented after discovery phase_

