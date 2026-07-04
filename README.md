# flirt-docs

Single source of truth for the **Flirt** product: architecture, scope, data model, API, iOS keyboard rules, AI prompts, and roadmap.

> All documentation is written in **English**. Keep it that way.

## Index

| Document | Purpose |
|---|---|
| [HANDOFF.md](./HANDOFF.md) | **Start here** — current status, decisions, how to run, next steps |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System overview, components, data flow |
| [MVP_SCOPE.md](./MVP_SCOPE.md) | What ships in the first version (and what doesn't) |
| [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md) | PostgreSQL tables and relationships |
| [API_ENDPOINTS.md](./API_ENDPOINTS.md) | REST contract between iOS, keyboard and backend |
| [IOS_KEYBOARD_RULES.md](./IOS_KEYBOARD_RULES.md) | Apple constraints for the keyboard extension |
| [AI_PROMPTS.md](./AI_PROMPTS.md) | Prompt templates, tones, structured output contract |
| [ROADMAP.md](./ROADMAP.md) | Phased delivery plan (v0.1 → v1.0) |
| [APP_STORE_COMPLIANCE.md](./APP_STORE_COMPLIANCE.md) | Apple review risks and mitigations (keyboard, AI content, 17+) |
| [COST_MODEL.md](./COST_MODEL.md) | Unit economics: cost per generation, per-user cost, margin levers |
| [ANALYTICS.md](./ANALYTICS.md) | Event schema, KPIs, validation targets |
| [TESTING_STRATEGY.md](./TESTING_STRATEGY.md) | What's automated vs the physical-device keyboard checklist |
| [CLAUDE.md](./CLAUDE.md) | Working conventions for Claude / contributors |

## Repos

- `flirt-ios` — iOS app + Custom Keyboard Extension (Swift + SwiftUI)
- `flirt-api` — Backend (NestJS + PostgreSQL + Redis), multi-provider AI layer
- `flirt-contracts` — Shared types / JSON schemas (TypeScript)
- `flirt-infra` — Docker Compose (local) → AWS (production)
- `flirt-docs` — This repo
