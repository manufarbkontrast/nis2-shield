# NIS2 Shield

NIS2-Compliance-SaaS for German Mittelstand (50-500 employees).

## What is this?

A SaaS platform that guides German companies through NIS2 compliance:
- Guided risk assessment
- BSI incident reporting (24h/72h/30d workflows)
- Policy document generator (10 templates)
- Secret management with AES-256-GCM encryption
- Compliance dashboard with scoring
- Management training tracker

## Tech Stack

- **Backend:** Django 5.x (Python)
- **Frontend:** SvelteKit (TypeScript)
- **Database:** PostgreSQL 16
- **Styling:** Tailwind CSS
- **Payment:** Stripe
- **Hosting:** Hetzner Cloud (Germany)

Based on [CISO Assistant](https://github.com/intuitem/ciso-assistant-community) (AGPLv3).

## Project Structure

```
nis2-shield/
├── prompts/          # Agent orchestration prompts (6 streams)
├── docs/             # Architecture & deployment docs
├── docker/           # Docker Compose files
├── backend/          # Django backend (coming)
├── frontend/         # SvelteKit frontend (coming)
└── marketing/        # Next.js landing page (coming)
```

## Getting Started

See `prompts/` directory for the 6 implementation streams.
Each stream is a self-contained prompt for Claude Code agents.

## License

AGPL-3.0 (inherited from CISO Assistant)
