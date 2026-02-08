# Stream 1: Infrastruktur & Projektsetup

## Kontext

Wir bauen **NIS2 Shield** — ein NIS2-Compliance-SaaS für den deutschen Mittelstand.
Basis: Fork von CISO Assistant (github.com/intuitem/ciso-assistant-community).
Stack: Django + SvelteKit + PostgreSQL + Tailwind + Docker auf Hetzner.

Dieses ist Stream 1 von 6. Hier geht es NUR um Infrastruktur, Repository-Setup und Deployment.

---

## Aufgabe 1.1: Repository Fork & Projektstruktur

**Agent: architect (model=opus)**
**Skill: `configure-ecc`, `setup`, `init`**

### Was zu tun ist:

1. Forke CISO Assistant in ein neues lokales Repository `nis2-shield`
2. Analysiere die komplette Projektstruktur von CISO Assistant:
   - Welche Django Apps existieren?
   - Wie ist das Frontend organisiert?
   - Wo liegen Konfigurationsdateien?
   - Wie funktioniert das Build-System?
   - Welche Docker-Files existieren?

3. Erstelle diese zusätzlichen Verzeichnisse für unsere eigenen Erweiterungen:

```
nis2-shield/
├── backend/
│   └── apps/
│       ├── tenants/          # Multi-Tenant (NEU)
│       ├── billing/          # Stripe Integration (NEU)
│       ├── bsi_reporting/    # BSI Meldeformulare (NEU)
│       ├── documents/        # Dokumenten-Generator (NEU)
│       └── secrets/          # Secret Management (NEU)
├── frontend/                 # SvelteKit (existierend, wird erweitert)
├── marketing/                # Next.js Landing Page (NEU, separates Projekt)
├── docker/
│   ├── docker-compose.dev.yml
│   ├── docker-compose.prod.yml
│   └── nginx/
│       └── nginx.conf
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   └── api.md
├── prompts/                  # Alle Stream-Prompts
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── CLAUDE.md                 # Projektkontext für Claude Code
├── .env.example
├── .gitignore
└── README.md
```

4. Erstelle `CLAUDE.md` mit folgendem Inhalt:

```markdown
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
- `bsi_reporting` — BSI-Meldeformular-Workflow
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
```

5. Erstelle `.env.example`:

```env
# Django
DJANGO_SECRET_KEY=change-me-to-random-string
DJANGO_DEBUG=true
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgres://nis2shield:password@localhost:5432/nis2shield
DATABASE_HOST=db
DATABASE_PORT=5432
DATABASE_NAME=nis2shield
DATABASE_USER=nis2shield
DATABASE_PASSWORD=change-me

# Redis
REDIS_URL=redis://redis:6379/0

# Secret Management
SECRETS_MASTER_KEY=change-me-to-32-byte-hex-string

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Email (Transactional)
EMAIL_HOST=smtp.zoho.eu
EMAIL_PORT=587
EMAIL_HOST_USER=info@nis2shield.de
EMAIL_HOST_PASSWORD=change-me
EMAIL_USE_TLS=true
DEFAULT_FROM_EMAIL=NIS2 Shield <info@nis2shield.de>

# App
APP_URL=https://app.nis2shield.de
MARKETING_URL=https://nis2shield.de
```

6. Erstelle `.gitignore` (Python + Node + Docker + IDE):

```
# Python
__pycache__/
*.py[cod]
*.pyo
*.egg-info/
dist/
build/
.eggs/
*.egg
.venv/
venv/
env/

# Node
node_modules/
.svelte-kit/
.next/
.nuxt/

# Environment
.env
.env.local
.env.production

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Docker
docker-compose.override.yml

# OS
.DS_Store
Thumbs.db

# Test
.coverage
htmlcov/
.pytest_cache/

# Build
staticfiles/
media/
```

7. Erstelle GitHub Actions CI Pipeline `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  backend-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install ruff
      - run: ruff check backend/

  backend-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: test_nis2shield
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r backend/requirements.txt
      - run: pip install pytest pytest-django pytest-cov
      - run: pytest backend/ --cov --cov-report=xml --cov-fail-under=80
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test_nis2shield

  frontend-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run lint
      - run: cd frontend && npm run check

  frontend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: cd frontend && npm ci
      - run: cd frontend && npm test
```

---

## Aufgabe 1.2: Docker & Deployment

**Agent: cloud-architect (model=sonnet)**
**Skill: `devops`, `deployment-pipeline-design`, `bun-docker`**

### docker/docker-compose.dev.yml

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: nis2shield
      POSTGRES_USER: nis2shield
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build:
      context: ../backend
      dockerfile: Dockerfile
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ../backend:/app
    ports:
      - "8000:8000"
    env_file:
      - ../.env
    depends_on:
      - db
      - redis

  frontend:
    build:
      context: ../frontend
      dockerfile: Dockerfile
    command: npm run dev -- --host 0.0.0.0
    volumes:
      - ../frontend:/app
      - /app/node_modules
    ports:
      - "5173:5173"
    env_file:
      - ../.env
    depends_on:
      - backend

volumes:
  pgdata:
```

### docker/docker-compose.prod.yml

Erstelle eine Production-Konfiguration mit:
- Nginx als Reverse Proxy mit SSL (Let's Encrypt via Certbot)
- Django mit Gunicorn (4 Workers)
- SvelteKit als Node Server
- PostgreSQL mit Persistent Volume
- Redis mit Passwort
- Automatische Restarts (`restart: always`)
- Health Checks für alle Services
- Resource Limits (Memory, CPU)
- Logging-Konfiguration

### docker/nginx/nginx.conf

Nginx Konfiguration:
- HTTPS redirect
- SSL mit Let's Encrypt
- Proxy zu Backend (api.nis2shield.de → Django:8000)
- Proxy zu Frontend (app.nis2shield.de → SvelteKit:3000)
- Security Headers (HSTS, X-Frame-Options, CSP, etc.)
- Gzip Compression
- Rate Limiting (10 req/s pro IP)
- Static Files Caching

### Backup-Strategie

Erstelle `docker/backup.sh`:
- Täglich: PostgreSQL pg_dump → komprimiert → Hetzner Storage Box
- 7 Tage Rotation (ältere Backups löschen)
- E-Mail Benachrichtigung bei Fehler
- Crontab-Eintrag dokumentieren

### Monitoring

Erstelle `docker/docker-compose.monitoring.yml`:
- Uptime Kuma (self-hosted, Port 3001)
- Monitored Endpoints:
  - https://app.nis2shield.de (Frontend)
  - https://app.nis2shield.de/api/health/ (Backend API)
  - PostgreSQL Port
  - Redis Port
- Alerting via E-Mail

---

## Aufgabe 1.3: Hetzner Server Provisioning

**Agent: cloud-architect (model=sonnet)**
**Skill: `devops`**

### Server-Spezifikation

Phase 1 (0-10 Kunden):
- **1x CX22** (2 vCPU, 4 GB RAM, 40 GB SSD) — 3,79 EUR/Monat
- Alles auf einem Server (App + DB + Redis + Nginx)

Phase 2 (10-50 Kunden):
- **1x CX32** (4 vCPU, 8 GB RAM, 80 GB SSD) — 6,80 EUR/Monat
- Oder: CX22 für App + Managed PostgreSQL (ab 10 EUR/Monat)

### Erstelle `docs/deployment.md`:

Schritt-für-Schritt Anleitung:
1. Hetzner Cloud Account erstellen
2. Server erstellen (CX22, Nürnberg Datacenter, Ubuntu 24.04)
3. SSH Key hinzufügen
4. Firewall konfigurieren (nur 80, 443, 22 von eigener IP)
5. Docker + Docker Compose installieren
6. Repository klonen
7. `.env` konfigurieren
8. `docker compose -f docker/docker-compose.prod.yml up -d`
9. SSL Zertifikat via Certbot
10. DNS konfigurieren
11. Erster Health Check
12. Backup-Cronjob einrichten
13. Monitoring starten

---

## Aufgabe 1.4: Entwicklungsumgebung lokal

**Agent: dx-optimizer (model=haiku)**
**Skill: `setup`**

### Erstelle `docs/development.md`:

Lokale Entwicklung Setup:
1. Repository klonen
2. Python 3.12+ installieren (pyenv empfohlen)
3. Node 20+ installieren (nvm empfohlen)
4. `.env` aus `.env.example` kopieren
5. `docker compose -f docker/docker-compose.dev.yml up db redis`
6. Backend: `cd backend && pip install -r requirements.txt && python manage.py migrate && python manage.py runserver`
7. Frontend: `cd frontend && npm install && npm run dev`
8. Öffne http://localhost:5173

### Pre-Commit Hooks

Erstelle `.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v9.0.0
    hooks:
      - id: eslint
        files: \.(js|ts|svelte)$
```

---

## Definition of Done — Stream 1

- [ ] Repository erstellt mit korrekter Verzeichnisstruktur
- [ ] CLAUDE.md vollständig und korrekt
- [ ] .env.example mit allen nötigen Variablen
- [ ] .gitignore vollständig
- [ ] GitHub Actions CI Pipeline läuft
- [ ] docker-compose.dev.yml startet lokal fehlerfrei
- [ ] docker-compose.prod.yml bereit für Hetzner Deployment
- [ ] nginx.conf mit SSL-Konfiguration
- [ ] Backup-Script erstellt
- [ ] Monitoring-Setup dokumentiert
- [ ] docs/deployment.md vollständig
- [ ] docs/development.md vollständig
- [ ] Pre-Commit Hooks konfiguriert

## Ausführung

```
Starte Stream 1. Nutze folgende Agents in dieser Reihenfolge:

1. architect (model=opus) → Repository-Analyse und CLAUDE.md
2. cloud-architect (model=sonnet) → Docker + Deployment
3. dx-optimizer (model=haiku) → Development Docs

Führe die Aufgaben 1.1 und 1.2 PARALLEL aus.
Aufgabe 1.3 und 1.4 können PARALLEL nach 1.2 starten.
```
