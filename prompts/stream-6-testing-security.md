# Stream 6: Testing & Security Audit

## Kontext

Lies `/CLAUDE.md` für Projektkontext.
Dieser Stream läuft NACH Streams 2+3 (Backend + Frontend müssen existieren).
Ziel: 80%+ Coverage, keine CRITICAL/HIGH Security Issues.

---

## 6.1 Backend Testing

**Agent: tdd-guide (model=sonnet) + test-automator (model=sonnet)**
**Skills: `django-tdd`, `python-testing`, `tdd-workflow`, `python-testing-patterns`**

### Unit Tests

Jede Django App braucht vollständige Unit Tests:

```
backend/apps/tenants/tests/
├── test_models.py         # Tenant, TenantUser, PlanLimit
├── test_middleware.py      # TenantMiddleware
├── test_managers.py        # TenantAwareManager
├── test_api.py            # Alle API Endpoints
└── test_plan_limits.py    # Limit-Durchsetzung

backend/apps/billing/tests/
├── test_checkout.py       # Stripe Checkout Session
├── test_webhooks.py       # Alle Webhook Events
├── test_subscription.py   # Trial, Upgrade, Downgrade, Cancel
└── test_trial.py          # Trial-Ablauf-Logik

backend/apps/bsi_reporting/tests/
├── test_incident.py       # CRUD
├── test_reports.py        # Report-Erstellung, Fristen
├── test_immutability.py   # Submitted Reports nicht änderbar
├── test_pdf.py            # PDF-Generierung
└── test_timeline.py       # Timeline-Einträge

backend/apps/documents/tests/
├── test_templates.py      # Alle 10 Templates laden
├── test_generation.py     # Dokument aus Template erstellen
├── test_pdf.py            # PDF-Export
├── test_versioning.py     # Neue Version erstellen
└── test_finalization.py   # Finalisieren = immutable

backend/apps/secrets/tests/
├── test_crypto.py         # Encrypt/Decrypt korrekt
├── test_tenant_key.py     # Tenant-Key Derivation
├── test_crud.py           # Secret CRUD
├── test_audit_log.py      # Jeder Zugriff wird geloggt
├── test_plan_limits.py    # Limit-Durchsetzung
├── test_soft_delete.py    # Soft-Delete + Restore
└── test_no_leak.py        # Secret-Werte NICHT in Responses wo sie nicht hingehören
```

### Integration Tests

```python
# test_full_onboarding.py
# 1. User registriert sich
# 2. E-Mail Verification
# 3. Tenant wird erstellt (Trial)
# 4. User loggt ein
# 5. Dashboard zeigt Score 0%
# 6. Risikoanalyse starten
# → Alles in einer Testsuite, echte DB

# test_full_incident_workflow.py
# 1. Incident erstellen
# 2. 3 Draft-Reports automatisch angelegt
# 3. Erstmeldung ausfüllen + submitten
# 4. Report ist immutable
# 5. PDF generieren
# 6. Folgemeldung ausfüllen + submitten
# 7. Abschlussbericht
# 8. Incident als resolved markieren

# test_billing_flow.py
# 1. Trial-Tenant erstellt
# 2. Trial läuft ab
# 3. Tenant wird deaktiviert
# 4. Stripe Checkout → Webhook → Tenant aktiviert
# 5. Plan-Tier korrekt gesetzt
# 6. Plan-Limits aktiv

# test_tenant_isolation_cross_check.py
# 1. Tenant A: Erstelle Secret, Incident, Document
# 2. Tenant B: Versuche auf A's Daten zuzugreifen → 403/leer
# 3. Wiederhole für ALLE Endpoints
```

### Coverage-Ziel
- Gesamt: >= 80%
- crypto.py: >= 95% (sicherheitskritisch!)
- middleware.py: >= 90%
- webhook handlers: >= 90%

---

## 6.2 Frontend Testing

**Agent: test-automator (model=sonnet)**
**Skills: `e2e-testing-patterns`, `playwright`, `vitest-testing`**

### E2E Tests (Playwright)

```
frontend/tests/e2e/
├── registration.spec.ts    # Registrierung komplett durchspielen
├── login.spec.ts           # Login + Logout
├── dashboard.spec.ts       # Dashboard lädt, Score sichtbar
├── risk-wizard.spec.ts     # Risikoanalyse 8 Steps durchspielen
├── incidents.spec.ts       # Vorfall melden, Report erstellen
├── documents.spec.ts       # Dokument aus Template erstellen
├── secrets.spec.ts         # Secret erstellen, anzeigen, kopieren
├── billing.spec.ts         # Pricing → Checkout (Stripe Test Mode)
└── responsive.spec.ts      # Kritische Flows auf 375px Viewport
```

### Component Tests (Vitest)

```
frontend/tests/unit/
├── ComplianceScore.test.ts   # Score-Berechnung + Farblogik
├── AlertBanner.test.ts       # Fristen-Warnungen
├── MeasureCard.test.ts       # Status-Anzeige
├── MaskedValue.test.ts       # Maskierung + Auto-Clear
├── PlanLimitBadge.test.ts    # Limit-Anzeige
└── RiskMatrix.test.ts        # Risiko-Berechnung
```

---

## 6.3 Security Audit

**Agent: security-reviewer (model=sonnet)**
**Skills: `security-review`, `django-security`, `api-security-hardening`**

### Checkliste (JEDER Punkt muss bestätigt werden)

#### Authentication
- [ ] Passwörter mit Argon2 oder bcrypt gehasht
- [ ] Mindestens 12 Zeichen Passwort-Policy
- [ ] Rate Limiting auf Login (max 5 Versuche/Minute)
- [ ] Session Timeout (8h inaktiv → Logout)
- [ ] CSRF Token auf allen State-ändernden Requests
- [ ] JWT Token Expiry (max 1h)
- [ ] Refresh Token Rotation

#### Authorization
- [ ] Tenant-Isolation: Jeder Query durch TenantAwareManager
- [ ] RBAC: Owner > Admin > Member > Viewer korrekt durchgesetzt
- [ ] API: Kein Endpoint ohne Authentication
- [ ] Admin-Panel: Nur für Plattform-Admin (nicht Tenant-Admin)

#### Data Security
- [ ] Secrets: AES-256-GCM korrekt implementiert
- [ ] Secrets: Nonce nie wiederverwendet
- [ ] Secrets: Master Key nur in Umgebungsvariable
- [ ] Secrets: Werte NICHT in Django Logs
- [ ] Secrets: Werte NICHT in Error Messages/Tracebacks
- [ ] Secrets: Werte NICHT in Sentry/Error Tracking
- [ ] DB: Alle Queries parametrisiert (kein SQL Injection)
- [ ] XSS: Alle User-Inputs escaped
- [ ] File Upload: Validiert und begrenzt (nur PDF bei Importen)

#### Infrastructure
- [ ] Docker: Container laufen NICHT als root
- [ ] Docker: Keine Secrets in Docker Images
- [ ] SSL/TLS: Nur TLS 1.2+ (kein TLS 1.0/1.1)
- [ ] HSTS Header aktiv
- [ ] X-Frame-Options: DENY
- [ ] X-Content-Type-Options: nosniff
- [ ] Content-Security-Policy konfiguriert
- [ ] Referrer-Policy: strict-origin-when-cross-origin
- [ ] CORS: Nur eigene Domains erlaubt

#### Dependencies
- [ ] `pip audit` — keine bekannten Vulnerabilities
- [ ] `npm audit` — keine bekannten Vulnerabilities
- [ ] Keine veralteten Dependencies mit bekannten CVEs
- [ ] Dependabot oder Renovate für automatische Updates

#### Stripe
- [ ] Webhook-Signatur wird verifiziert
- [ ] Stripe Secret Key nur serverseitig
- [ ] Publishable Key im Frontend (kein Secret Key!)
- [ ] PCI Compliance via Stripe (wir speichern KEINE Kartendaten)

#### DSGVO
- [ ] Datenlöschung möglich (Account löschen → alle Daten weg)
- [ ] Datenexport möglich (JSON/CSV Export)
- [ ] Logging proportional (nicht mehr als nötig)
- [ ] AVV verfügbar
- [ ] TOMs dokumentiert
- [ ] Cookie-Banner korrekt (nur technisch notwendig)

---

## 6.4 Performance Baseline

**Agent: performance-engineer (model=sonnet)**
**Skills: `performance-optimization`, `web-performance-optimization`**

### Benchmarks die eingehalten werden müssen

```
Backend API:
- GET /api/health/           < 50ms
- GET /api/compliance/score/ < 200ms
- GET /api/secrets/          < 200ms (Liste, ohne Werte)
- GET /api/secrets/{id}/     < 300ms (mit Entschlüsselung)
- POST /api/incidents/       < 500ms
- GET /api/documents/{id}/pdf/ < 3000ms (PDF-Generierung)

Frontend:
- Dashboard First Load       < 2s (LCP)
- Navigation zwischen Seiten < 500ms
- Risk Wizard Step-Wechsel   < 300ms

Marketing (Landing Page):
- LCP                        < 2.5s
- FID                        < 100ms
- CLS                        < 0.1
- Lighthouse Score           > 90
```

### Load Testing (optional, Phase 2)

```
- 50 concurrent Users → Kein Error, < 500ms p95
- 100 concurrent API Calls → < 1s p95
- Simulation: 50 Tenants, je 5 Users, normale Nutzung
```

---

## 6.5 Code Review

**Agent: code-reviewer (model=sonnet)**
**Skills: `code-review`, `code-review-excellence`**

Review ALLER eigenen Django Apps und Frontend-Erweiterungen:

1. Code Quality:
   - Immutable Patterns eingehalten?
   - Funktionen < 50 Zeilen?
   - Dateien < 800 Zeilen?
   - Klare Namensgebung?
   - Keine Deep Nesting (> 4 Levels)?

2. Error Handling:
   - Alle Fehler explizit behandelt?
   - User-freundliche Fehlermeldungen?
   - Keine swallowed Exceptions?

3. Security:
   - Keine hardcoded Secrets?
   - Input Validation an Systemgrenzen?
   - Alle Security-Checklist Punkte aus 6.3?

4. Testing:
   - Coverage >= 80%?
   - Edge Cases getestet?
   - Negative Tests (was soll NICHT gehen)?

---

## Definition of Done — Stream 6

- [ ] Backend Test-Coverage >= 80%
- [ ] crypto.py Coverage >= 95%
- [ ] Alle E2E Tests grün
- [ ] Security Checkliste: ALLE Punkte ✅
- [ ] pip audit: 0 known vulnerabilities
- [ ] npm audit: 0 known vulnerabilities
- [ ] Performance Benchmarks eingehalten
- [ ] Code Review: Keine CRITICAL/HIGH Issues offen
- [ ] Kein Secret-Wert in Logs/Errors/Responses (verifiziert)

## Ausführung

```
Starte Stream 6 NACH Streams 2+3.

1. PARALLEL:
   → 6.1 Backend Tests (tdd-guide + test-automator)
   → 6.2 Frontend Tests (test-automator)
   → 6.3 Security Audit (security-reviewer)

2. DANACH:
   → 6.4 Performance Baseline (performance-engineer)
   → 6.5 Code Review (code-reviewer)
```
