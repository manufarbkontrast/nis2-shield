# Stream 3: SvelteKit Frontend

## Kontext

Lies `/CLAUDE.md` für Projektkontext.
CISO Assistant hat bereits ein SvelteKit-Frontend. Wir erweitern es.
**Vorbedingung:** Stream 1 (Infra) abgeschlossen. Stream 2 (Backend APIs) parallel oder davor.

---

## 3.1 Branding & Theme

**Agent: ui-designer (model=sonnet)**
**Skills: `design-system-patterns`, `tailwind-design-system`, `bun-sveltekit`**

Erstelle ein eigenes Theme für NIS2 Shield:

### Farbpalette
```
Primary:    Navy/Dunkelblau (#1e3a5f) — Vertrauen, Sicherheit
Secondary:  Blau (#3b82f6) — Aktionen, Links
Success:    Grün (#22c55e) — Compliant, Erledigt
Warning:    Orange (#f59e0b) — Achtung, bald fällig
Danger:     Rot (#ef4444) — Kritisch, überfällig
Background: Weiß (#ffffff) / Hellgrau (#f8fafc)
Text:       Dunkelgrau (#1e293b)
Muted:      Grau (#64748b)
```

### Typography
- Schriftart: Inter (Google Fonts, kostenlos)
- Headings: font-bold
- Body: font-normal, text-base (16px)
- Small: text-sm (14px)

### Aufgaben
1. Analysiere CISO Assistants Theme-System (Tailwind Config, CSS Variablen)
2. Überschreibe Farben und Fonts mit eigenem Theme
3. Erstelle Logo-Platzhalter (Text "NIS2 Shield" mit Schild-Icon)
4. Stelle sicher: Alle Seiten nutzen das neue Theme
5. Responsive: Mobile-first, funktioniert ab 375px Breite
6. Barrierefreiheit: Kontrastverhältnisse prüfen (WCAG AA)

---

## 3.2 Deutsche Lokalisierung

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `internationalization-i18n`**

1. Prüfe CISO Assistants i18n-System (Paraglide-JS oder i18next)
2. Setze Deutsch als Standard-Sprache
3. Prüfe alle existierenden deutschen Übersetzungen auf:
   - Vollständigkeit (fehlende Keys?)
   - Qualität (korrekte Fachbegriffe?)
   - Konsistenz (gleiche Begriffe für gleiche Konzepte?)
4. Ergänze fehlende Übersetzungen
5. NIS2-spezifische Begriffe:
   - Risk Assessment → Risikobewertung
   - Incident Report → Vorfallsmeldung
   - Control → Maßnahme
   - Policy → Richtlinie
   - Framework → Regelwerk
   - Compliance Score → Konformitätsstatus
   - Secret → Geheimnis/Zugangsdaten
6. Datumsformat: DD.MM.YYYY
7. Zahlenformat: 1.234,56

---

## 3.3 NIS2 Compliance Dashboard

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `frontend-patterns`, `responsive-design`**

Erstelle/erweitere die Haupt-Dashboard-Seite:

### Layout

```
┌──────────────────────────────────────────────────────────┐
│  NIS2 Shield    [Dashboard] [Risiko] [Vorfälle] [Docs]  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐  ┌─────────────────────────────────────┐  │
│  │  Score   │  │  ⚠ BSI-Registrierung: 26 Tage       │  │
│  │   67%    │  │  ⚠ 2 überfällige Maßnahmen          │  │
│  │  ██████░ │  │  ✅ Keine offenen Vorfallsmeldungen  │  │
│  └──────────┘  └─────────────────────────────────────┘  │
│                                                          │
│  Quick Actions:                                          │
│  [Risikoanalyse] [Vorfall melden] [Report] [Secret]     │
│                                                          │
│  NIS2-Maßnahmen:                                        │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │🔴Risiko│ │🟡Vorfall│ │🟢Backup│ │🔴Liefer│           │
│  │analyse │ │mgmt    │ │       │ │kette  │              │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │🟡Beschaf│ │🔴Wirksam│ │🟡Schulun│ │🟢Krypto│          │
│  │fung    │ │keit    │ │gen    │ │grafie │              │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│  ┌────────┐ ┌────────┐                                  │
│  │🟡Zugriff│ │🟢MFA   │                                 │
│  └────────┘ └────────┘                                  │
│                                                          │
│  Letzte Aktivitäten:                                     │
│  • 14:23 — Max Müller hat Passwort-Richtlinie erstellt  │
│  • 12:01 — System: Monatliches Backup erfolgreich       │
│  • Gestern — Anna Schmidt: Risikoanalyse gestartet      │
└──────────────────────────────────────────────────────────┘
```

### Komponenten

1. **ComplianceScore.svelte** — Kreisdiagramm mit Prozent und Farbe
2. **AlertBanner.svelte** — Warnungen/Fristen oben
3. **QuickActions.svelte** — 4 große Buttons
4. **MeasureCard.svelte** — Kachel für jede NIS2-Maßnahme (Status + Klick→Detail)
5. **ActivityFeed.svelte** — Timeline der letzten Aktionen
6. **TeamStatus.svelte** — GF-Schulungsstatus

API-Calls:
- GET /api/v1/compliance/score/ → Score-Daten
- GET /api/v1/compliance/measures/ → Status aller 10 Maßnahmen
- GET /api/v1/incidents/overdue/ → Überfällige Meldungen
- GET /api/v1/activity/ → Letzte Aktivitäten

---

## 3.4 Geführte Risikoanalyse (Wizard)

**Agent: frontend-developer (model=sonnet) + ui-designer (model=sonnet)**
**Skills: `bun-sveltekit`, `frontend-patterns`, `interaction-design`**

### Wizard-Steps

```
Step 1: Willkommen
→ "In 30 Minuten zu Ihrer NIS2-Risikoanalyse"
→ Erklärung was passiert, was am Ende rauskommt

Step 2: Unternehmensprofil
→ Branche (NIS2-Sektoren Dropdown)
→ Mitarbeiterzahl
→ Standorte
→ Bereits vorhanden: ISO 27001? BSI-Grundschutz?

Step 3: IT-Assets erfassen
→ Tabelle: Name, Typ (Server/Client/Cloud/Netzwerk/IoT), Kritikalität
→ Vorschläge je nach Branche (Maschinenbau → CNC, ERP, CAD)
→ [+ Asset hinzufügen] Button

Step 4: Bedrohungen bewerten (pro Asset)
→ Wahrscheinlichkeit: Niedrig (1) / Mittel (2) / Hoch (3) / Sehr hoch (4)
→ Auswirkung: Niedrig (1) / Mittel (2) / Hoch (3) / Kritisch (4)
→ Risiko = Wahrscheinlichkeit × Auswirkung (automatisch)
→ Visuelle Risikomatrix (4×4 Grid, farbcodiert)

Step 5: Bestehende Maßnahmen
→ Checkliste: Was tut ihr bereits?
→ [ ] Firewall aktiv
→ [ ] Antivirensoftware
→ [ ] Regelmäßige Backups
→ [ ] MFA für kritische Systeme
→ [ ] Mitarbeiterschulungen
→ usw.

Step 6: Gap-Analyse (automatisch generiert)
→ "Basierend auf Ihren Angaben fehlen folgende Maßnahmen:"
→ Priorisierte Liste mit Aufwand-Schätzung

Step 7: Maßnahmenplan
→ Empfohlene Reihenfolge
→ Zeitrahmen pro Maßnahme
→ Verantwortlichen zuweisen

Step 8: Zusammenfassung + Export
→ Compliance-Score nach Bewertung
→ [PDF herunterladen]
→ [Im Dashboard speichern]
```

### UX-Anforderungen
- Progressbar oben (Step X von 8)
- Zurück-Button (nichts verloren)
- Zwischenspeichern (localStorage + API)
- Tooltips für Fachbegriffe
- Beispielantworten als Hinweis
- Mobile-fähig

---

## 3.5 Onboarding Flow

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `interaction-design`, `auth-implementation-patterns`**

### Flow

```
1. /register
   → E-Mail + Passwort (min 12 Zeichen)
   → Oder: "Mit Google anmelden" (OAuth, optional Phase 2)
   → Datenschutz + AGB akzeptieren

2. /verify-email
   → "Wir haben Ihnen eine E-Mail gesendet. Bitte bestätigen Sie."
   → Resend-Button nach 60 Sekunden

3. /onboarding/company
   → Firmenname
   → Branche (NIS2-Sektoren Dropdown)
   → Mitarbeiterzahl
   → PLZ (für Hebesatz bei Gewerbesteuer — optional)

4. /onboarding/check
   → "Ist Ihr Unternehmen NIS2-betroffen?"
   → 3 Fragen:
     a) Sektor in NIS2-Liste? (Ja/Nein basierend auf Branche von Step 3)
     b) >= 50 Mitarbeiter ODER > 10 Mio EUR Umsatz?
     c) Sonderfall (KRITIS, DNS, TLD, Vertrauensdienst)?
   → Ergebnis: "Ja, Sie sind betroffen" / "Vermutlich nicht betroffen"

5. /dashboard (wenn betroffen)
   → Welcome-Tour (3 Tooltips: Score, Quick Actions, Maßnahmen)
   → Automatisch: "Starten Sie mit der Risikoanalyse" Prompt

6. /not-affected (wenn nicht betroffen)
   → "Aktuell scheinen Sie nicht betroffen. Prüfen Sie regelmäßig."
   → "Trotzdem IT-Sicherheit verbessern? → Dashboard starten"
```

---

## 3.6 Secret Management UI

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `frontend-patterns`**

### Seiten

```
/secrets
→ Ordner-Browser (links) + Secret-Liste (rechts)
→ Suchfeld oben
→ Filter: Typ, Ablaufdatum, Ordner
→ Plan-Limit Badge: "23/50 Secrets"

/secrets/new
→ Formular:
  - Name (Pflicht)
  - Typ: Dropdown (Passwort/API-Key/Zertifikat/Token/Verbindungsstring/Sonstiges)
  - Wert (Passwort-Feld mit Augen-Icon zum Anzeigen)
  - Beschreibung (optional)
  - URL (optional)
  - Ablaufdatum (optional)
  - Ordner (Dropdown)

/secrets/{id}
→ Detail-Ansicht
→ Wert standardmäßig maskiert: ••••••••••••
→ [Anzeigen] Button → zeigt Wert für 30 Sekunden, dann auto-maskiert
→ [Kopieren] Button → kopiert, zeigt Toast "In Zwischenablage kopiert. Wird in 30s gelöscht."
→ Versionshistorie unten
→ Audit-Log: "Zuletzt angesehen von Max Müller am 05.02.2026"

/secrets/audit
→ Tabelle: Wer hat wann welches Secret angesehen/geändert/gelöscht
→ Filter: User, Action, Zeitraum
→ Export als CSV

/secrets/expiring
→ Secrets die in 30/14/7 Tagen ablaufen
→ Sortiert nach Dringlichkeit
```

### Komponenten
- SecretBrowser.svelte (Ordner + Liste)
- SecretForm.svelte (Erstellen/Bearbeiten)
- SecretDetail.svelte (Anzeige mit Maskierung)
- SecretAuditLog.svelte (Zugriffsprotokolle)
- MaskedValue.svelte (Wert mit Anzeigen/Kopieren)
- PlanLimitBadge.svelte (23/50)

---

## 3.7 BSI-Meldung UI

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `frontend-patterns`**

### Seiten

```
/incidents
→ Tabelle aller Vorfälle
→ Status-Badge: Offen/In Bearbeitung/Geschlossen
→ Fristen-Anzeige: "Erstmeldung in 18h fällig" (rot wenn < 4h)
→ [Neuen Vorfall melden] Button

/incidents/new
→ Formular:
  - Was ist passiert? (Incident Type Dropdown)
  - Schweregrad (Critical/High/Medium/Low)
  - Seit wann? (DateTime Picker)
  - Betroffene Systeme (Multi-Select oder Freitext)
  - Beschreibung (Textarea)
  - Sofortmaßnahmen (Textarea)
→ Submit → Erstellt Incident + 3 Draft-Reports

/incidents/{id}
→ Incident-Details
→ Timeline (alle Aktionen chronologisch)
→ 3 Report-Tabs: Erstmeldung | Folgemeldung | Abschlussbericht
→ Jeder Tab: Formular ausfüllen → [Als PDF] → [Einreichen]
→ Nach Einreichung: Formular read-only, Timestamp angezeigt
→ Countdown bis nächste Frist

/incidents/{id}/reports/{rid}/pdf
→ PDF-Vorschau im Browser
→ [Herunterladen] Button
```

---

## 3.8 Dokumente UI

**Agent: frontend-developer (model=sonnet)**
**Skills: `bun-sveltekit`, `frontend-patterns`**

### Seiten

```
/documents
→ Grid/Liste aller verfügbaren Templates
→ Kategorie-Filter: Richtlinien, Pläne, Formulare, Berichte
→ Status pro Dokument: Nicht erstellt / Entwurf / Finalisiert
→ Plan-Lock: Templates die höheren Plan brauchen → Upgrade-CTA

/documents/new?template={slug}
→ Formular basierend auf Template
→ Firmen-Daten vorausgefüllt (aus Tenant-Profil)
→ Platzhalter klar markiert ("Bitte ausfüllen")
→ Live-Vorschau rechts (optional, Phase 2)
→ [Speichern als Entwurf] / [Finalisieren + PDF]

/documents/{id}
→ Dokument-Details
→ Wenn Entwurf: bearbeitbar
→ Wenn Finalisiert: read-only + PDF-Download
→ Versionshistorie: V1, V2, V3...
→ [Neue Version erstellen] (kopiert Inhalt, neue Version)
```

---

## Definition of Done — Stream 3

- [ ] Theme: Eigene Farben, Fonts, Logo-Platzhalter
- [ ] Deutsch: Alle UI-Strings auf Deutsch, korrekte Fachbegriffe
- [ ] Dashboard: Compliance-Score, Alerts, Quick Actions, 10 Maßnahmen-Kacheln
- [ ] Risikoanalyse: 8-Step Wizard komplett durchspielbar
- [ ] Onboarding: Register → Verify → Company → Check → Dashboard
- [ ] Secrets: CRUD, Maskierung, Kopieren, Audit-Log, Plan-Limits
- [ ] BSI-Meldungen: Erstellen, Berichte, Fristen, PDF
- [ ] Dokumente: Templates, Ausfüllen, PDF-Export, Versionierung
- [ ] Responsive: Funktioniert auf Mobile (375px+)
- [ ] Accessibility: Kontraste, Keyboard-Navigation, aria-Labels

## Ausführung

```
Starte Stream 3. PARALLEL zu Stream 2.

1. ZUERST: 3.1 Branding + 3.2 Lokalisierung (PARALLEL)
   → ui-designer + frontend-developer

2. DANN: 3.3 Dashboard + 3.5 Onboarding (PARALLEL)
   → frontend-developer (2 Instanzen)

3. DANN: 3.4 Risikoanalyse + 3.6 Secrets + 3.7 BSI + 3.8 Dokumente (PARALLEL)
   → frontend-developer (4 Instanzen, oder sequentiell wenn Context zu voll)
```
