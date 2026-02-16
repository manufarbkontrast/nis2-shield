# NIS2 Shield

NIS2-Compliance-SaaS fuer den deutschen Mittelstand (50-500 Mitarbeiter). Gefuehrte Risikoanalyse, BSI-Meldepflichten, Dokumenten-Generator, Secret Management und Compliance-Dashboard.

Basiert auf [CISO Assistant](https://github.com/intuitem/ciso-assistant-community) (AGPLv3) mit proprietaeren Erweiterungen fuer deutsche NIS2-Anforderungen.

---

## Architektur

```mermaid
graph TB
    subgraph Frontend["Frontend (SvelteKit)"]
        Dashboard["Compliance Dashboard<br/>Score + 10 Massnahmen"]
        Wizard["Risiko-Wizard<br/>8 Schritte, 30 Min"]
        Incidents["BSI Meldungen<br/>24h / 72h / 30d"]
        Docs["Dokumenten-Generator<br/>10 Templates"]
        Secrets["Secret Manager<br/>AES-256-GCM"]
        Billing["Billing<br/>Stripe Checkout"]
    end

    subgraph Backend["Backend (Django 5.x)"]
        TenantApp["tenants<br/>Multi-Tenant + Rollen"]
        BillingApp["billing<br/>Stripe Subscriptions"]
        BSIApp["bsi_reporting<br/>Meldepflichten"]
        DocsApp["documents<br/>Jinja2 Templates"]
        SecretsApp["secrets<br/>Verschluesselung"]
    end

    subgraph Infra["Infrastruktur (Hetzner DE)"]
        Nginx["Nginx<br/>Reverse Proxy + TLS 1.3"]
        Postgres["PostgreSQL 16<br/>Multi-Tenant Isolation"]
        Redis["Redis 7<br/>Cache"]
    end

    subgraph External["Externe Services"]
        Stripe["Stripe<br/>Payments"]
        Email["SMTP<br/>Transaktions-E-Mails"]
        Sentry["Sentry<br/>Error Tracking"]
    end

    subgraph Marketing["Landing Page (Next.js 15)"]
        LP["nis2shield.de<br/>SEO + Blog"]
    end

    Frontend --> Nginx
    Nginx --> Backend
    Backend --> Postgres
    Backend --> Redis
    Backend --> Stripe
    Backend --> Email
    Backend --> Sentry
    LP --> Stripe
```

## Multi-Tenant Architektur

```mermaid
flowchart TB
    A["HTTP Request"] --> B["Nginx<br/>Rate Limiting"]
    B --> C["Django Middleware"]
    C --> D["TenantMiddleware<br/>request.tenant extrahieren"]
    D --> E["Authentication<br/>JWT / Session"]
    E --> F["Authorization<br/>Rolle pruefen (owner/admin/member/viewer)"]
    F --> G["TenantAwareManager<br/>QuerySet automatisch gefiltert"]
    G --> H["PostgreSQL<br/>WHERE tenant_id = ?"]
    H --> I["Response<br/>Nur eigene Daten"]
```

## BSI-Meldeworkflow

```mermaid
sequenceDiagram
    participant M as Mitarbeiter
    participant App as NIS2 Shield
    participant DB as PostgreSQL
    participant Mail as E-Mail
    participant BSI as BSI (extern)

    M->>App: Sicherheitsvorfall melden
    App->>DB: Incident erstellen
    App->>DB: 3 Berichts-Entwuerfe anlegen

    Note over App: Erstmeldung (24h Frist)
    App->>Mail: Erinnerung 4h vorher
    App->>Mail: Erinnerung 1h vorher
    M->>App: Erstmeldung ausfuellen + absenden
    App->>DB: Status: submitted (unveraenderlich)
    M->>BSI: PDF exportieren + uebermitteln

    Note over App: Folgemeldung (72h Frist)
    App->>Mail: Erinnerungen senden
    M->>App: Folgemeldung ausfuellen + absenden

    Note over App: Abschlussbericht (30 Tage Frist)
    App->>Mail: Erinnerungen senden
    M->>App: Abschlussbericht + absenden
    App-->>M: Alle Meldungen abgeschlossen
```

## Secret Management

```mermaid
flowchart TB
    A["SECRETS_MASTER_KEY<br/>(32 Bytes, Env-Variable)"] --> B["HKDF-SHA256<br/>context = tenant_id"]
    B --> C["Tenant-Key<br/>(einzigartig pro Mandant)"]
    C --> D["AES-256-GCM<br/>pro Secret"]
    D --> E["Verschluesselter Wert<br/>+ Nonce in DB"]

    F["Secret abrufen"] --> G{"Berechtigung?"}
    G -->|Ja| H["Entschluesselung<br/>+ Audit-Log"]
    G -->|Nein| I["403 Forbidden"]

    H --> J["Wert an Client<br/>(nur Detail-Endpoint)"]

    style A fill:#ff6b6b,color:#fff
    style E fill:#51cf66,color:#fff
```

## Pricing-Modell

```mermaid
graph LR
    subgraph Trial["Trial (14 Tage)"]
        T1["Kostenlos"]
        T2["2 User"]
        T3["10 Secrets"]
    end

    subgraph Starter["Starter (50-100 MA)"]
        S1["499 EUR Setup<br/>+ 99 EUR/Monat"]
        S2["5 User"]
        S3["50 Secrets"]
    end

    subgraph Pro["Professional (100-250 MA)"]
        P1["999 EUR Setup<br/>+ 199 EUR/Monat"]
        P2["15 User"]
        P3["500 Secrets"]
        P4["Multi-Framework"]
    end

    subgraph Enterprise["Enterprise (250-500 MA)"]
        E1["1.999 EUR Setup<br/>+ 399 EUR/Monat"]
        E2["Unbegrenzt User"]
        E3["Unbegrenzt Secrets"]
        E4["API-Zugang"]
    end

    Trial -->|Upgrade| Starter
    Starter -->|Upgrade| Pro
    Pro -->|Upgrade| Enterprise
```

## User Journey

```mermaid
stateDiagram-v2
    [*] --> Registrierung: E-Mail + Passwort
    Registrierung --> Betroffenheitscheck: Firma anlegen
    Betroffenheitscheck --> Dashboard: "Ja, betroffen"
    Betroffenheitscheck --> Info: "Nicht betroffen"

    Dashboard --> Risikoanalyse: Wizard starten (30 Min)
    Dashboard --> Dokumente: Template auswaehlen
    Dashboard --> Vorfallmeldung: Incident melden
    Dashboard --> Secrets: Passwoerter verwalten

    Risikoanalyse --> Dashboard: Score aktualisiert
    Dokumente --> Dashboard: Massnahme auf Gruen
    Vorfallmeldung --> BSIMeldung: 3 Berichte
    BSIMeldung --> Dashboard: Fristen eingehalten

    state Trial {
        Dashboard --> TrialEnde: 14 Tage
        TrialEnde --> Upgrade: Stripe Checkout
        Upgrade --> Dashboard: Plan aktiviert
    }
```

---

## Tech-Stack

| Komponente | Technologie |
|---|---|
| Backend | Django 5.x (Python 3.12+) |
| Frontend | SvelteKit (TypeScript) + Tailwind CSS |
| Marketing | Next.js 15 (App Router) |
| Datenbank | PostgreSQL 16 |
| Cache | Redis 7 |
| Payment | Stripe (Subscriptions + Checkout) |
| Verschluesselung | AES-256-GCM + HKDF-SHA256 |
| Hosting | Hetzner Cloud (Nuernberg, DSGVO-konform) |
| Reverse Proxy | Nginx + TLS 1.3 |
| CI/CD | GitHub Actions |
| Container | Docker + Docker Compose |
| Basis | CISO Assistant (AGPLv3) |

---

## Features

- **Compliance Dashboard**: Score (0-100%), 10 NIS2-Massnahmen mit Ampelstatus
- **Risiko-Wizard**: 8-Schritt-Analyse (Assets, Bedrohungen, Luecken, Massnahmenplan)
- **BSI-Meldepflichten**: 24h/72h/30d Workflow mit Fristen-Tracking + E-Mail-Erinnerungen
- **Dokumenten-Generator**: 10 Pflicht-Vorlagen (IT-Sicherheit, Passwort, Backup, Incident Response...)
- **Secret Management**: AES-256-GCM Verschluesselung, Versionierung, Audit-Log
- **Multi-Tenant**: Row-Level Isolation, Rollen (Owner/Admin/Member/Viewer)
- **Stripe Billing**: Trial (14 Tage), 3 Preisstufen, Setup-Gebuehr + monatlich
- **GF-Training-Tracker**: Geschaeftsfuehrer-Schulung (4h in 3 Jahren)

---

## Voraussetzungen

- Python 3.12+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 16
- Stripe Account
- SMTP-Server (Transaktions-E-Mails)

---

## Schnellstart

```bash
# 1. Repository klonen
git clone https://github.com/manufarbkontrast/nis2-shield.git
cd nis2-shield

# 2. Umgebungsvariablen
cp .env.example .env
# .env mit Werten befuellen

# 3. Infrastruktur starten
docker compose -f docker/docker-compose.dev.yml up db redis

# 4. Backend
cd backend
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:8000

# 5. Frontend
cd frontend
npm install
npm run dev
```

---

## Umgebungsvariablen

```env
# Django
DJANGO_SECRET_KEY=xxx
DJANGO_ALLOWED_HOSTS=app.nis2shield.de,localhost

# Datenbank
DATABASE_URL=postgres://user:pass@host:5432/nis2shield

# Redis
REDIS_URL=redis://redis:6379/0

# Secret Management
SECRETS_MASTER_KEY=xxx  # 32-Byte Hex-String

# Stripe
STRIPE_SECRET_KEY=sk_xxx
STRIPE_PUBLISHABLE_KEY=pk_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# E-Mail
EMAIL_HOST=smtp.zoho.eu
EMAIL_HOST_USER=info@nis2shield.de
EMAIL_HOST_PASSWORD=xxx
DEFAULT_FROM_EMAIL=NIS2 Shield <info@nis2shield.de>

# URLs
APP_URL=https://app.nis2shield.de
MARKETING_URL=https://nis2shield.de
```

---

## Projektstruktur

```
nis2-shield/
├── prompts/                  # 6 Implementation-Streams
│   ├── stream-1-infrastructure.md
│   ├── stream-2-backend.md
│   ├── stream-3-frontend.md
│   ├── stream-4-landing-seo.md
│   ├── stream-5-legal.md
│   └── stream-6-testing-security.md
├── backend/                  # Django (Python)
│   └── apps/
│       ├── tenants/          # Multi-Tenant + Plan-Limits
│       ├── billing/          # Stripe Subscriptions
│       ├── bsi_reporting/    # 24h/72h/30d Meldungen
│       ├── documents/        # 10 NIS2-Templates
│       └── secrets/          # AES-256-GCM Verschluesselung
├── frontend/                 # SvelteKit (TypeScript)
├── marketing/                # Next.js 15 Landing Page
├── docker/
│   ├── docker-compose.dev.yml
│   ├── docker-compose.prod.yml
│   ├── backup.sh
│   └── nginx/nginx.conf
└── .github/workflows/        # CI + Deploy
```

---

## Lizenz

AGPL-3.0 (geerbt von CISO Assistant)
