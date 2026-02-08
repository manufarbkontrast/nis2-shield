# Stream 2: Django Backend — Eigene Apps

## Kontext

Lies zuerst `/CLAUDE.md` im Repository Root für Projektkontext.
Dieser Stream implementiert alle eigenen Django-Apps die CISO Assistant erweitern.
CISO Assistant Code wird NICHT verändert — wir bauen Addons drumherum.

**Vorbedingung:** Stream 1 muss abgeschlossen sein (Repository + Docker läuft lokal).

---

## 2.1 Multi-Tenant Architektur

**Agent: backend-architect (model=opus)**
**Skills: `django-patterns`, `database-schema-design`, `postgresql`**

### Erstelle Django App: `backend/apps/tenants/`

**TDD-Pflicht: Tests ZUERST schreiben.**
**Agent für Tests: tdd-guide (model=sonnet)**
**Skills: `django-tdd`, `tdd-workflow`, `python-testing`**

### Models (tenants/models.py)

```python
# IMMUTABLE PATTERN: Alle Updates erzeugen neue Objekte oder nutzen
# Django's QuerySet.update() statt save() auf bestehenden Instanzen.

class PlanTier(models.TextChoices):
    TRIAL = "trial", "14-Tage Test"
    STARTER = "starter", "Starter"
    PROFESSIONAL = "professional", "Professional"
    ENTERPRISE = "enterprise", "Enterprise"

class Tenant:
    id: UUID (primary key, auto)
    name: str (Firmenname)
    slug: str (unique, für Subdomain/URL)
    plan_tier: PlanTier (default=TRIAL)
    is_active: bool (default=True)
    trial_ends_at: datetime (nullable)
    stripe_customer_id: str (nullable)
    industry_sector: str (NIS2-Sektor)
    employee_count: int
    created_at: datetime (auto)
    updated_at: datetime (auto)

class TenantUser:
    id: UUID
    tenant: FK(Tenant)
    user: FK(User)  # Django's auth.User
    role: str (choices: owner/admin/member/viewer)
    invited_at: datetime
    accepted_at: datetime (nullable)
    is_active: bool

class PlanLimit:
    tier: PlanTier (unique)
    max_users: int
    max_secrets: int
    max_frameworks: int
    max_documents: int
    has_supplier_assessment: bool
    has_multi_framework: bool
    has_api_access: bool
    has_sso: bool
    has_priority_support: bool
```

### Plan-Limits Werte

| Feature | Trial | Starter | Professional | Enterprise |
|---------|-------|---------|-------------|------------|
| max_users | 2 | 5 | 15 | Unlimited (9999) |
| max_secrets | 10 | 50 | 500 | Unlimited |
| max_frameworks | 1 | 1 | 3 | Unlimited |
| max_documents | 5 | 10 | 10 | 10 |
| has_supplier_assessment | No | No | Yes | Yes |
| has_multi_framework | No | No | Yes | Yes |
| has_api_access | No | No | No | Yes |
| has_sso | No | No | No | Yes |

### Middleware (tenants/middleware.py)

```python
# TenantMiddleware:
# 1. Extrahiert Tenant aus Request (via authenticated user's tenant membership)
# 2. Setzt request.tenant
# 3. Alle nachfolgenden Queries filtern automatisch nach diesem Tenant
# 4. Wenn User keinem Tenant zugeordnet → 403
# 5. Wenn Tenant nicht aktiv → 402 (Payment Required)
```

### Managers (tenants/managers.py)

```python
# TenantAwareManager:
# Überschreibt get_queryset() um automatisch nach request.tenant zu filtern.
# JEDES Model das tenant-spezifisch ist muss diesen Manager nutzen.
```

### API Endpoints (tenants/api/)

```
POST   /api/v1/tenants/                   # Tenant erstellen (nur bei Registration)
GET    /api/v1/tenants/current/            # Aktuellen Tenant abrufen
PATCH  /api/v1/tenants/current/            # Tenant-Profil updaten
GET    /api/v1/tenants/current/users/      # Team-Mitglieder auflisten
POST   /api/v1/tenants/current/users/invite/ # Einladung senden
DELETE /api/v1/tenants/current/users/{id}/ # Mitglied entfernen
GET    /api/v1/tenants/current/plan/       # Plan & Limits anzeigen
GET    /api/v1/tenants/current/usage/      # Aktuelle Nutzung vs. Limits
```

### Tests die ZUERST geschrieben werden

```python
# tests/test_tenant_isolation.py
# - Erstelle 2 Tenants mit je einem User
# - Tenant A erstellt Daten
# - User von Tenant B darf diese Daten NICHT sehen
# - User von Tenant B bekommt leere Liste oder 403

# tests/test_plan_limits.py
# - Tenant mit Starter Plan
# - Erstelle 50 Secrets → OK
# - Erstelle Secret #51 → 403 mit Meldung "Plan-Limit erreicht"

# tests/test_tenant_middleware.py
# - Request ohne Auth → 401
# - Request mit Auth aber ohne Tenant → 403
# - Request mit Auth + Tenant → request.tenant gesetzt

# tests/test_tenant_lifecycle.py
# - Trial erstellen → trial_ends_at = now + 14 Tage
# - Trial abgelaufen + kein Upgrade → is_active = False
# - Upgrade auf Starter → is_active = True, plan_tier = starter
```

---

## 2.2 NIS2-Framework vervollständigen

**Agent: search-specialist (model=sonnet) → dann python-pro (model=sonnet)**
**Skills: `django-patterns`, `python-code-style`**

### Phase A: Recherche

Prüfe im CISO Assistant Source Code:
1. Wo liegen Compliance-Frameworks? (vermutlich als YAML/JSON in einem `library/` Ordner)
2. Ist NIS2 bereits als Framework enthalten?
3. Welches Format nutzt CISO Assistant für Framework-Definitionen?
4. Wie werden Controls/Maßnahmen definiert?

### Phase B: NIS2-Framework erstellen/ergänzen

Erstelle das vollständige NIS2-Framework im CISO Assistant Format:

```yaml
# Struktur (angepasst an CISO Assistant's Format):

framework:
  name: "NIS2 (§30 BSIG)"
  description: "NIS2-Umsetzungsgesetz — Risikomanagementmaßnahmen"
  locale: de
  version: "1.0"

requirement_groups:
  - id: nis2-art21-1
    name: "Risikoanalyse und Sicherheitskonzepte"
    description: "Konzepte für die Risikoanalyse und Sicherheit für Informationssysteme"
    requirements:
      - id: nis2-art21-1-a
        name: "Asset-Inventar"
        description: "Vollständige Erfassung aller IT-Systeme und -Dienste"
        guidance: "Erstellen Sie eine Liste aller Hardware, Software, Cloud-Dienste..."
        priority: critical

  - id: nis2-art21-2
    name: "Bewältigung von Sicherheitsvorfällen"
    # ... BSI-Meldepflichten 24h/72h/30d

  - id: nis2-art21-3
    name: "Business Continuity Management"
    # ... Backup, Notfallplan, Krisenmanagement

  - id: nis2-art21-4
    name: "Sicherheit der Lieferkette"
    # ... Lieferanten-Bewertung

  - id: nis2-art21-5
    name: "Sicherheit bei Beschaffung, Entwicklung und Wartung"

  - id: nis2-art21-6
    name: "Bewertung der Wirksamkeit"
    # ... Regelmäßige Überprüfung der Maßnahmen

  - id: nis2-art21-7
    name: "Cyberhygiene und Schulungen"
    # ... Mitarbeiter-Awareness, GF-Schulungspflicht

  - id: nis2-art21-8
    name: "Kryptografie und Verschlüsselung"
    # ... Hier kommt Secret Management rein!

  - id: nis2-art21-9
    name: "Personalsicherheit und Zugriffskontrolle"
    # ... RBAC, MFA, Onboarding/Offboarding

  - id: nis2-art21-10
    name: "Multi-Faktor-Authentifizierung"
```

Ergänze für jede Requirement:
- Deutsche Beschreibung (verständlich für Nicht-ITler)
- Praxisbeispiel (was bedeutet das für einen Metallbauer?)
- Mapping zu ISO 27001 Annex A Controls (wenn möglich)
- Priorität: critical / high / medium / low

---

## 2.3 BSI-Meldeformular-Workflow

**Agent: backend-architect (model=sonnet)**
**Skills: `django-patterns`, `workflow-orchestration-patterns`, `pdf`**
**TDD Agent: tdd-guide (model=sonnet)**

### Erstelle Django App: `backend/apps/bsi_reporting/`

### Models

```python
class IncidentType(models.TextChoices):
    RANSOMWARE = "ransomware", "Ransomware / Verschlüsselungstrojaner"
    DATA_BREACH = "data_breach", "Datenleck / Datenabfluss"
    DDOS = "ddos", "DDoS-Angriff"
    PHISHING = "phishing", "Phishing (erfolgreich)"
    UNAUTHORIZED_ACCESS = "unauthorized_access", "Unberechtigter Zugriff"
    MALWARE = "malware", "Schadsoftware (sonstige)"
    SUPPLY_CHAIN = "supply_chain", "Lieferketten-Angriff"
    OTHER = "other", "Sonstiges"

class IncidentSeverity(models.TextChoices):
    CRITICAL = "critical", "Kritisch"
    HIGH = "high", "Hoch"
    MEDIUM = "medium", "Mittel"
    LOW = "low", "Niedrig"

class ReportType(models.TextChoices):
    INITIAL = "initial", "Erstmeldung (24h)"
    UPDATE = "update", "Folgemeldung (72h)"
    FINAL = "final", "Abschlussbericht (30 Tage)"

class ReportStatus(models.TextChoices):
    DRAFT = "draft", "Entwurf"
    SUBMITTED = "submitted", "Eingereicht"

class SecurityIncident:
    id: UUID
    tenant: FK(Tenant)
    incident_type: IncidentType
    severity: IncidentSeverity
    title: str
    description: text
    detected_at: datetime
    affected_systems: JSONField (Liste von Systemnamen)
    estimated_impact: text
    immediate_actions_taken: text
    is_resolved: bool (default=False)
    resolved_at: datetime (nullable)
    created_by: FK(User)
    created_at: datetime

class IncidentReport:
    id: UUID
    incident: FK(SecurityIncident)
    report_type: ReportType
    status: ReportStatus
    content: JSONField (strukturiertes Formular)
    submitted_at: datetime (nullable)
    due_at: datetime  # Automatisch berechnet
    pdf_file: FileField (nullable)
    created_by: FK(User)
    created_at: datetime

class IncidentTimelineEntry:
    id: UUID
    incident: FK(SecurityIncident)
    action: str
    details: text
    user: FK(User)
    created_at: datetime
```

### Fristenlogik

```python
# Bei Erstellung eines SecurityIncident:
# 1. Erstmeldung (initial) due_at = detected_at + 24 Stunden
# 2. Folgemeldung (update) due_at = detected_at + 72 Stunden
# 3. Abschlussbericht (final) due_at = detected_at + 30 Tage

# Automatische Reminder (via Django Management Command oder Celery):
# - 4h vor Frist: E-Mail Warnung "Erstmeldung in 4 Stunden fällig"
# - 1h vor Frist: E-Mail Eskalation "DRINGEND: Erstmeldung in 1 Stunde fällig"
# - Nach Frist: E-Mail "ÜBERFÄLLIG: Erstmeldung nicht eingereicht"
```

### PDF-Export

Jeder IncidentReport muss als PDF exportierbar sein:
- BSI-konformes Layout
- Firmenlogo + Adresse
- Alle Pflichtfelder
- Zeitstempel der Einreichung
- Wasserzeichen "ENTWURF" bei status=draft

### API Endpoints

```
POST   /api/v1/incidents/                           # Neuen Vorfall melden
GET    /api/v1/incidents/                           # Alle Vorfälle (Tenant)
GET    /api/v1/incidents/{id}/                       # Vorfall-Details
PATCH  /api/v1/incidents/{id}/                       # Vorfall updaten
POST   /api/v1/incidents/{id}/reports/               # Bericht erstellen
GET    /api/v1/incidents/{id}/reports/               # Alle Berichte zum Vorfall
GET    /api/v1/incidents/{id}/reports/{rid}/          # Bericht-Details
PATCH  /api/v1/incidents/{id}/reports/{rid}/          # Bericht bearbeiten (nur draft)
POST   /api/v1/incidents/{id}/reports/{rid}/submit/   # Bericht einreichen (→ submitted, immutable)
GET    /api/v1/incidents/{id}/reports/{rid}/pdf/       # PDF Export
GET    /api/v1/incidents/{id}/timeline/               # Timeline
GET    /api/v1/incidents/overdue/                     # Überfällige Meldungen
```

### Tests ZUERST

```python
# test_incident_creation.py
# - Vorfall erstellen → 3 Reports automatisch als draft angelegt
# - due_at korrekt berechnet (24h, 72h, 30d)

# test_incident_submit.py
# - Draft-Report submitten → status=submitted, submitted_at gesetzt
# - Submitted Report bearbeiten → 403 (immutable!)

# test_incident_pdf.py
# - PDF generieren → gültige PDF-Datei
# - PDF enthält alle Pflichtfelder

# test_incident_overdue.py
# - Vorfall mit abgelaufener Frist → erscheint in /overdue/

# test_incident_tenant_isolation.py
# - Tenant A erstellt Vorfall
# - Tenant B kann ihn NICHT sehen
```

---

## 2.4 Dokumenten-Generator

**Agent: python-pro (model=sonnet)**
**Skills: `pdf`, `django-patterns`, `python-design-patterns`**

### Erstelle Django App: `backend/apps/documents/`

### Models

```python
class DocumentTemplate:
    id: UUID
    slug: str (unique)
    name: str (deutsch)
    description: text
    category: str (policy/plan/form/report)
    template_file: str (Pfad zum Jinja2 Template)
    required_plan: PlanTier (ab welchem Plan verfügbar)
    sort_order: int

class GeneratedDocument:
    id: UUID
    tenant: FK(Tenant)
    template: FK(DocumentTemplate)
    title: str
    version: int (default=1)
    content: JSONField (ausgefüllte Felder)
    pdf_file: FileField (nullable)
    status: str (draft/final)
    created_by: FK(User)
    created_at: datetime
    finalized_at: datetime (nullable)
```

### 10 Pflicht-Templates

Erstelle Jinja2-Templates für jedes Dokument. Alle auf Deutsch.
Platzhalter werden aus Tenant-Profil + User-Input befüllt.

```
1. informationssicherheitsrichtlinie.html
   → Übergeordnete IT-Sicherheitsrichtlinie
   → Platzhalter: Firmenname, GF-Name, Geltungsbereich, Datum

2. passwort_richtlinie.html
   → Mindestlänge, Komplexität, Rotation, MFA
   → Platzhalter: Firmenname, Mindestlänge (Empfehlung: 12), Rotationsintervall

3. backup_richtlinie.html
   → Backup-Strategie, Aufbewahrung, Wiederherstellung
   → Platzhalter: Backup-Frequenz, Aufbewahrungsdauer, Verantwortlicher

4. notfallhandbuch.html
   → Rollen, Kommunikationskette, Wiederanlauf-Plan
   → Platzhalter: Notfall-Team, Telefonnummern, Prioritätssysteme

5. zugriffskontroll_richtlinie.html
   → Need-to-Know, RBAC, Onboarding/Offboarding
   → Platzhalter: Rollen-Definitionen, Genehmigungsprozess

6. kryptografie_richtlinie.html
   → Verschlüsselungsstandards, Schlüsselverwaltung
   → Platzhalter: Verschlüsselungsalgorithmen, Schlüsselrotation

7. lieferanten_bewertung.html
   → Formular zur Bewertung von IT-Dienstleistern
   → Platzhalter: Lieferantenname, Bewertungskriterien, Risiko-Score

8. schulungsnachweis_gf.html
   → Nachweis der GF-Schulung (4h in 3 Jahren)
   → Platzhalter: Name, Datum, Thema, Dauer, Trainer

9. verzeichnis_verarbeitungstaetigkeiten.html
   → DSGVO Art. 30 — Bonus-Feature
   → Platzhalter: Verarbeitungstätigkeit, Zweck, Betroffene, Empfänger

10. nis2_compliance_report.html
    → Gesamtbericht: Status aller 10 Maßnahmen
    → Automatisch befüllt aus Compliance-Dashboard Daten
```

### API Endpoints

```
GET    /api/v1/documents/templates/              # Alle verfügbaren Templates
POST   /api/v1/documents/                        # Neues Dokument aus Template
GET    /api/v1/documents/                        # Alle Dokumente (Tenant)
GET    /api/v1/documents/{id}/                   # Dokument-Details
PATCH  /api/v1/documents/{id}/                   # Dokument bearbeiten (nur draft)
POST   /api/v1/documents/{id}/finalize/          # Dokument finalisieren (immutable)
GET    /api/v1/documents/{id}/pdf/               # PDF Export
POST   /api/v1/documents/{id}/new-version/       # Neue Version erstellen (kopiert content)
```

---

## 2.5 Secret Management

**Agent: security-reviewer (model=sonnet) → backend-architect (model=sonnet)**
**Skills: `secrets-management`, `django-security`, `python-design-patterns`**

### Erstelle Django App: `backend/apps/secrets/`

### Verschlüsselungsarchitektur

```
SECRETS_MASTER_KEY (Umgebungsvariable, 32 Byte Hex)
    ↓
HKDF (SHA-256) mit tenant_id als Context
    ↓
Tenant-Key (einzigartig pro Tenant)
    ↓
AES-256-GCM Verschlüsselung jedes einzelnen Secret-Werts
    ↓
Gespeichert in DB: encrypted_value (bytes) + nonce (bytes)
```

### Models

```python
class SecretFolder:
    id: UUID
    tenant: FK(Tenant)
    name: str
    parent: FK(SecretFolder, nullable)  # Hierarchische Ordner
    created_by: FK(User)
    created_at: datetime

class Secret:
    id: UUID
    tenant: FK(Tenant)
    folder: FK(SecretFolder, nullable)
    name: str
    secret_type: str (password/api_key/certificate/token/connection_string/other)
    encrypted_value: BinaryField  # AES-256-GCM verschlüsselt
    nonce: BinaryField            # Für AES-GCM
    description: text (optional, NICHT verschlüsselt)
    url: str (optional, zugehörige URL)
    expires_at: datetime (nullable)
    created_by: FK(User)
    created_at: datetime
    updated_at: datetime
    is_deleted: bool (default=False)  # Soft-Delete
    deleted_at: datetime (nullable)

class SecretVersion:
    id: UUID
    secret: FK(Secret)
    encrypted_value: BinaryField
    nonce: BinaryField
    version_number: int
    created_by: FK(User)
    created_at: datetime

class SecretAccessLog:
    id: UUID
    tenant: FK(Tenant)
    secret: FK(Secret, nullable)  # nullable weil Secret gelöscht sein kann
    secret_name: str              # Denormalisiert für Audit
    user: FK(User)
    action: str (view/create/update/delete/restore)
    ip_address: GenericIPAddressField
    user_agent: str
    created_at: datetime
```

### Crypto Service (secrets/crypto.py)

```python
# Nutze: cryptography (Python Library)
# KEIN eigenes Krypto bauen!

# derive_tenant_key(master_key: bytes, tenant_id: UUID) -> bytes
#     → HKDF(SHA256, master_key, info=tenant_id.bytes) → 32 Byte Key

# encrypt_secret(tenant_key: bytes, plaintext: str) -> tuple[bytes, bytes]
#     → AES-256-GCM encrypt → (ciphertext, nonce)

# decrypt_secret(tenant_key: bytes, ciphertext: bytes, nonce: bytes) -> str
#     → AES-256-GCM decrypt → plaintext
```

### API Endpoints

```
GET    /api/v1/secrets/folders/                  # Ordnerstruktur
POST   /api/v1/secrets/folders/                  # Ordner erstellen
GET    /api/v1/secrets/                          # Alle Secrets (ohne Werte!)
POST   /api/v1/secrets/                          # Secret erstellen
GET    /api/v1/secrets/{id}/                     # Secret-Details (mit entschlüsseltem Wert)
PUT    /api/v1/secrets/{id}/                     # Secret updaten (neue Version)
DELETE /api/v1/secrets/{id}/                     # Soft-Delete
POST   /api/v1/secrets/{id}/restore/             # Wiederherstellen
GET    /api/v1/secrets/{id}/versions/            # Versionshistorie (ohne Werte)
GET    /api/v1/secrets/{id}/versions/{v}/        # Spezifische Version (mit Wert)
GET    /api/v1/secrets/audit-log/                # Zugriffsprotokolle
GET    /api/v1/secrets/expiring/                 # Bald ablaufende Secrets
POST   /api/v1/secrets/import/                   # CSV/JSON Bulk-Import
```

### Sicherheitsregeln

```
1. GET /secrets/ listet Secrets OHNE encrypted_value (nur Metadaten)
2. GET /secrets/{id}/ liefert entschlüsselten Wert UND schreibt Audit-Log
3. Secret-Werte NIEMALS in:
   - Django Logs
   - Error Messages
   - Sentry/Error Tracking
   - API Error Responses
4. Rate Limiting: Max 60 Secret-Reads pro Minute pro User
5. Plan-Limit Check: Vor jedem POST /secrets/ prüfen
6. Tenant-Isolation: TenantAwareManager auf allen Querysets
```

---

## 2.6 Stripe Billing

**Agent: payment-integration (model=sonnet)**
**Skills: `stripe-integration`, `payment-gateway-integration`, `django-patterns`**

### Erstelle Django App: `backend/apps/billing/`

### Models

```python
class Subscription:
    id: UUID
    tenant: OneToOne(Tenant)
    stripe_subscription_id: str (nullable)
    stripe_customer_id: str
    plan_tier: PlanTier
    status: str (trialing/active/past_due/canceled/unpaid)
    current_period_start: datetime
    current_period_end: datetime
    cancel_at_period_end: bool (default=False)
    created_at: datetime

class Payment:
    id: UUID
    tenant: FK(Tenant)
    stripe_payment_intent_id: str
    amount_cents: int
    currency: str (default="eur")
    payment_type: str (setup_fee/recurring)
    status: str (succeeded/failed/pending)
    created_at: datetime

class Invoice:
    id: UUID
    tenant: FK(Tenant)
    stripe_invoice_id: str
    amount_cents: int
    status: str (paid/open/void)
    pdf_url: str (Stripe hosted invoice PDF)
    period_start: datetime
    period_end: datetime
    created_at: datetime
```

### Stripe Products Setup (erstelle via Stripe API oder Dashboard)

```python
# Stripe Products:
# - "NIS2 Shield Starter"
# - "NIS2 Shield Professional"
# - "NIS2 Shield Enterprise"

# Stripe Prices (für jedes Product):
# 1. Einmalige Setup-Fee (one_time):
#    - Starter: 49900 cents (499 EUR)
#    - Professional: 99900 cents (999 EUR)
#    - Enterprise: 199900 cents (1.999 EUR)
#
# 2. Monatliche Subscription (recurring, monthly):
#    - Starter: 9900 cents (99 EUR)
#    - Professional: 19900 cents (199 EUR)
#    - Enterprise: 39900 cents (399 EUR)
```

### Checkout Flow

```python
# 1. User klickt "Starter Plan kaufen" auf Pricing-Seite
# 2. Backend erstellt Stripe Checkout Session:
#    - line_items: [setup_fee (one_time), subscription (recurring)]
#    - mode: "subscription" (Stripe erlaubt one_time + subscription zusammen)
#    - success_url: /billing/success?session_id={CHECKOUT_SESSION_ID}
#    - cancel_url: /pricing
# 3. User wird zu Stripe Checkout weitergeleitet
# 4. Nach Zahlung → Stripe Webhook → tenant aktivieren
```

### Webhook Handler (billing/webhooks.py)

```python
# POST /api/v1/billing/webhooks/stripe/

# Events die wir verarbeiten:
# - checkout.session.completed → Tenant aktivieren, Subscription erstellen
# - invoice.paid → Payment erstellen, Zugang bestätigen
# - invoice.payment_failed → Warning-Email, nach 3x → Tenant suspenden
# - customer.subscription.updated → Plan-Änderung speichern
# - customer.subscription.deleted → Tenant auf expired setzen

# WICHTIG:
# - Webhook-Signatur verifizieren (stripe.Webhook.construct_event)
# - Idempotent: Gleiches Event 2x verarbeiten = kein Problem
# - Async: Webhook muss in <5s antworten (200 OK), schwere Arbeit async
```

### API Endpoints

```
POST   /api/v1/billing/checkout/                 # Checkout Session erstellen
GET    /api/v1/billing/subscription/             # Aktuelle Subscription
POST   /api/v1/billing/portal/                   # Stripe Customer Portal URL
GET    /api/v1/billing/invoices/                 # Rechnungshistorie
POST   /api/v1/billing/webhooks/stripe/          # Stripe Webhook (kein Auth!)
```

### Trial-Logik

```python
# Bei Registrierung:
# - Tenant erstellen mit plan_tier=TRIAL, trial_ends_at=now+14days
# - KEIN Stripe Customer erstellt (keine Kreditkarte nötig)
#
# Daily Cron Job (check_trials):
# - Alle Tenants mit plan_tier=TRIAL und trial_ends_at < now
# - E-Mail: "Ihre Testphase ist abgelaufen. Upgraden Sie jetzt."
# - Nach 3 Tagen: is_active = False (Zugang gesperrt, Daten bleiben)
# - Nach 30 Tagen inaktiv: Daten löschen (DSGVO)
```

---

## Definition of Done — Stream 2

- [ ] `apps/tenants/` — Multi-Tenant mit Row-Level Isolation, Tests pass
- [ ] `apps/bsi_reporting/` — Incident-Workflow mit 24h/72h/30d Fristen, PDF Export, Tests pass
- [ ] `apps/documents/` — 10 Templates, PDF Export, Versionierung, Tests pass
- [ ] `apps/secrets/` — AES-256-GCM Encryption, Audit-Log, Plan-Limits, Tests pass
- [ ] `apps/billing/` — Stripe Checkout, Webhooks, Trial-Logik, Tests pass
- [ ] NIS2-Framework vollständig (alle 10 Maßnahmen-Gruppen)
- [ ] Alle Tests grün, Coverage >= 80%
- [ ] Kein SECRET in Logs oder Error Messages
- [ ] Alle APIs dokumentiert (Docstrings + OpenAPI)

## Ausführung

```
Starte Stream 2. Reihenfolge:

1. ZUERST: 2.1 Multi-Tenant (alles andere hängt davon ab)
   → Agents: backend-architect (opus) + tdd-guide (sonnet)

2. DANN PARALLEL:
   → 2.2 NIS2 Framework: search-specialist + python-pro
   → 2.3 BSI Meldungen: backend-architect + tdd-guide
   → 2.4 Dokumente: python-pro
   → 2.5 Secrets: security-reviewer + backend-architect + tdd-guide
   → 2.6 Billing: payment-integration

3. DANACH: Code Review aller Apps
   → Agent: code-reviewer
```
