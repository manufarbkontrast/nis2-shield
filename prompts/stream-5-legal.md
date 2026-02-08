# Stream 5: Rechtliches & DSGVO

## Kontext

Lies `/CLAUDE.md` für Projektkontext.
Dieser Stream erstellt alle rechtlich notwendigen Dokumente für den SaaS-Betrieb.
Kann ab Tag 1 PARALLEL zu allen anderen Streams laufen.

---

## 5.1 Rechtliche Dokumente

**Agent: legal-advisor (model=sonnet)**
**Skills: `gdpr-data-handling`, `compliance-check`**

### Dokument 1: Datenschutzerklärung

Erstelle als Markdown-Datei: `marketing/content/legal/datenschutz.md`

Inhalt abdecken:
- Verantwortlicher (Platzhalter für Name/Adresse)
- Welche Daten werden erhoben (Account-Daten, Nutzungsdaten, Zahlungsdaten)
- Zweck der Verarbeitung
- Rechtsgrundlage (Art. 6 DSGVO)
- Speicherdauer
- Empfänger (Auftragsverarbeiter):
  - Hetzner Online GmbH (Hosting, Serverstandort: Nürnberg, Deutschland)
  - Stripe Inc. (Zahlungsabwicklung, AVV vorhanden)
  - Vercel Inc. (Marketing-Website Hosting)
  - Zoho Corporation (E-Mail)
- Drittlandtransfer (Stripe USA → EU-US Data Privacy Framework)
- Rechte der Betroffenen (Auskunft, Berichtigung, Löschung, Einschränkung, Portabilität, Widerspruch)
- Cookies (nur technisch notwendige)
- Kontakt Datenschutz
- Änderungshistorie

### Dokument 2: AGB / Nutzungsbedingungen

Erstelle: `marketing/content/legal/agb.md`

Inhalt:
- §1 Geltungsbereich
- §2 Vertragsgegenstand (NIS2-Compliance SaaS Plattform)
- §3 Registrierung und Account
- §4 Leistungsbeschreibung (nach Tiers)
- §5 Testphase (14 Tage, kostenlos, keine Kreditkarte)
- §6 Preise und Zahlung
  - Einrichtungsgebühr (einmalig)
  - Monatliche Gebühr
  - Alle Preise netto zzgl. USt (wenn Kleinunternehmerregelung nicht greift)
  - Zahlung via Stripe
  - Preisänderungen mit 30 Tagen Vorlauf
- §7 Vertragslaufzeit und Kündigung
  - Monatlich kündbar (zum Ende des Abrechnungszeitraums)
  - Kündigung per E-Mail oder über Billing Portal
  - Nach Kündigung: Daten 30 Tage verfügbar zum Export, dann Löschung
- §8 Verfügbarkeit
  - Ziel: 99% Verfügbarkeit (monatlich)
  - Geplante Wartung: mit 48h Vorlauf angekündigt
  - Kein Anspruch auf ununterbrochene Verfügbarkeit
- §9 Haftung
  - Haftungsbegrenzung auf Vertragswert der letzten 12 Monate
  - Ausschluss für indirekte Schäden, entgangenen Gewinn
  - KEINE Haftung dafür, dass Kunde durch unser Tool NIS2-compliant IST
  - Tool ist Hilfsmittel, keine Rechtsberatung
  - Kunde ist selbst für seine Compliance verantwortlich
- §10 Datenschutz (Verweis auf Datenschutzerklärung + AVV)
- §11 Geheimhaltung
- §12 Änderungen der AGB
- §13 Schlussbestimmungen
  - Deutsches Recht
  - Gerichtsstand: [dein Sitz]
  - Salvatorische Klausel

### Dokument 3: Auftragsverarbeitungsvertrag (AVV)

Erstelle: `marketing/content/legal/avv.md`

Basierend auf Art. 28 DSGVO:
- Gegenstand und Dauer der Verarbeitung
- Art und Zweck der Verarbeitung
- Art der personenbezogenen Daten
- Kategorien betroffener Personen
- Pflichten des Auftragsverarbeiters (uns):
  - Verarbeitung nur auf dokumentierte Weisung
  - Vertraulichkeitsverpflichtung der Mitarbeiter
  - Technische und organisatorische Maßnahmen (Verweis auf TOMs)
  - Unterauftragsverarbeiter nur mit Genehmigung
  - Unterstützung bei Betroffenenrechten
  - Löschung nach Vertragsende
  - Kontrollrechte des Auftraggebers
- Anlage 1: TOMs
- Anlage 2: Liste der Unterauftragsverarbeiter (Hetzner, Stripe, Vercel, Zoho)

### Dokument 4: Technisch-Organisatorische Maßnahmen (TOMs)

Erstelle: `marketing/content/legal/toms.md`

- Zutrittskontrolle (Hetzner Datacenter: ISO 27001, SOC 2)
- Zugangskontrolle (SSH-Key only, MFA für Admin)
- Zugriffskontrolle (RBAC, Tenant-Isolation, Principle of Least Privilege)
- Weitergabekontrolle (TLS 1.3, HSTS)
- Eingabekontrolle (Audit-Logging)
- Auftragskontrolle (nur dokumentierte Weisungen)
- Verfügbarkeitskontrolle (Backups, Monitoring, Redundanz)
- Trennungskontrolle (Multi-Tenant mit Row-Level Isolation)
- Verschlüsselung (AES-256-GCM at-rest, TLS 1.3 in-transit)
- Pseudonymisierung (wo möglich)

### Dokument 5: Impressum

Erstelle: `marketing/content/legal/impressum.md`

Pflichtangaben nach §5 TMG + §18 MStV:
- Name und Anschrift (Platzhalter)
- Kontaktdaten (E-Mail, Telefon)
- Gewerbeanmeldung
- USt-ID (wenn vorhanden, sonst "in Beantragung")
- Verantwortlich für Inhalte nach §18 MStV
- Streitbeilegung (Hinweis auf OS-Plattform der EU)

### Dokument 6: Cookie-Banner Konfiguration

Erstelle: `marketing/content/legal/cookies.md`

- Nur technisch notwendige Cookies (Session, Auth, CSRF)
- KEINE Tracking-Cookies (kein Google Analytics)
- Einfacher Banner: "Wir verwenden nur technisch notwendige Cookies." [OK]
- Kein Opt-In nötig da nur technisch notwendig

---

## 5.2 Haftungsschutz

**Agent: legal-advisor (model=sonnet)**

Erstelle: `docs/legal-notes.md`

Dokumentiere für den Gründer:
1. Warum §9 AGB (Haftungsbegrenzung) kritisch wichtig ist
2. Empfehlung: IT-Haftpflichtversicherung (welche Anbieter, welche Summe)
3. Empfehlung: Cyber-Versicherung (für eigenen Betrieb)
4. Risiko: Kunde wird trotz Tool nicht NIS2-compliant → unsere Haftung?
5. Disclaimer-Texte die im Tool angezeigt werden sollten:
   - "NIS2 Shield unterstützt Sie bei der NIS2-Compliance. Es ersetzt keine Rechtsberatung."
   - "Die Nutzung garantiert keine vollständige Konformität."
6. Dokumentiere: Ab wann brauchen wir einen Datenschutzbeauftragten? (>20 MA)

---

## Definition of Done — Stream 5

- [ ] Datenschutzerklärung (deutsch, vollständig, DSGVO-konform)
- [ ] AGB mit Haftungsbegrenzung und Disclaimer
- [ ] AVV-Template mit TOMs-Anlage
- [ ] TOMs dokumentiert
- [ ] Impressum-Template (mit Platzhaltern)
- [ ] Cookie-Banner Konfiguration
- [ ] Legal Notes für den Gründer
- [ ] Alle Dokumente als Markdown in `marketing/content/legal/`

## Ausführung

```
Starte Stream 5 PARALLEL zu allen anderen Streams (keine Abhängigkeiten).
Ein Agent reicht: legal-advisor (model=sonnet)
Alle Dokumente in einem Durchgang erstellen.
```
