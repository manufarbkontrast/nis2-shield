# NIS2 Shield — Projektkontext

## Übersicht
NIS2-Compliance-SaaS für den deutschen Mittelstand (50-500 MA).
Basiert auf CISO Assistant (AGPLv3 Fork) mit eigenen Erweiterungen.

## Stack
- Backend: Django 5.x (Python 3.12+)
- Frontend: SvelteKit (TypeScript)
- Datenbank: PostgreSQL 16
- Cache: Redis
- Styling: Tailwind CSS
- Payment: Stripe
- Hosting: Hetzner Cloud (Deutschland)
- CI/CD: GitHub Actions

## Architektur-Entscheidungen
- Multi-Tenant via Organization Model + Row-Level Filtering
- Secret Management als eigenes Django-Modul (NICHT Infisical)
- Marketing-Website als separates Next.js Projekt
- Alle eigenen Erweiterungen in `backend/apps/` als separate Django Apps
- CISO Assistant Code wird minimal verändert — eigene Features als Addons

## Pricing-Modell
- Starter (50-100 MA): 499 EUR Setup + 99 EUR/Monat
- Professional (100-250 MA): 999 EUR Setup + 199 EUR/Monat
- Enterprise (250-500 MA): 1.999 EUR Setup + 399 EUR/Monat

## Coding-Standards
- Immutable Patterns: Neue Objekte erstellen, nie mutieren
- Funktionen: < 50 Zeilen
- Dateien: < 800 Zeilen
- Test-Coverage: >= 80%
- TDD: Tests IMMER zuerst schreiben
- Sprache: Code auf Englisch, UI-Strings auf Deutsch
- Error Handling: Explizit, nie schlucken
- Input Validation: An jeder Systemgrenze

## Eigene Django Apps
- `tenants` — Multi-Tenant Logik, Pläne, Limits
- `billing` — Stripe Subscriptions, Invoices
- `bsi_reporting` — BSI-Meldeformular-Workflow (24h/72h/30d)
- `documents` — NIS2-Dokumentenvorlagen + PDF-Export
- `secrets` — Secret Management mit AES-256-GCM

## Branch-Strategie
- `main` — Production (nur via PR)
- `develop` — Staging
- `feature/*` — Feature Branches
- `hotfix/*` — Hotfixes

## Commit-Format
<type>: <beschreibung>

Types: feat, fix, refactor, docs, test, chore, perf, ci

## Streams
Das Projekt wird in 6 Streams aufgeteilt:
1. Infrastruktur & Projektsetup
2. Django Backend (eigene Apps)
3. SvelteKit Frontend
4. Landing Page & SEO (Next.js)
5. Rechtliches & DSGVO
6. Testing & Security Audit
