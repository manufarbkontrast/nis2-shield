# Stream 4: Landing Page & SEO

## Kontext

Lies `/CLAUDE.md` für Projektkontext.
Die Marketing-Website ist ein SEPARATES Projekt (Next.js), nicht Teil von CISO Assistant.
Kann ab Tag 1 PARALLEL zu allen anderen Streams gebaut werden.

---

## 4.1 Landing Page (Next.js)

**Agent: frontend-developer (model=sonnet)**
**Skills: `nextjs-app-router-patterns`, `tailwind-v4-shadcn`, `responsive-design`, `seo-optimizer`**

### Tech Stack
- Next.js 15 (App Router)
- TypeScript
- Tailwind CSS
- Hosting: Vercel Free Tier (0 EUR)

### Erstelle Projekt: `marketing/`

### Seiten

#### Homepage (`/`)

```
[Nav: Logo | Produkt | Pricing | Blog | NIS2-Info | Login]

HERO:
"NIS2-Compliance für den Mittelstand.
 Einfach. Bezahlbar. Sicher."
[14 Tage kostenlos testen]  [Demo ansehen]

TRUST BAR:
🇩🇪 Hosted in Germany | 🔒 DSGVO-konform | 🏛️ Open Source basiert

PROBLEM:
"Seit Dezember 2025 müssen ~30.000 deutsche Unternehmen NIS2 einhalten.
 Bei Verstoß: bis zu 10 Mio. EUR Bußgeld. Geschäftsführer haften persönlich."

LÖSUNG:
"NIS2 Shield führt Sie Schritt für Schritt zur Compliance."

FEATURES (6 Kacheln):
1. Compliance Dashboard — Sehen Sie sofort wo Sie stehen
2. Geführte Risikoanalyse — In 30 Minuten zur Bewertung
3. BSI-Meldeformulare — 24h/72h/30d Workflows automatisiert
4. Dokumenten-Generator — 10 Richtlinien-Vorlagen auf Deutsch
5. Secret Management — Passwörter & API-Keys verschlüsselt verwalten
6. Schulungs-Tracker — GF-Schulungspflicht im Blick

HOW IT WORKS (3 Steps):
1. Registrieren & Betroffenheitscheck (2 Minuten)
2. Risikoanalyse durchführen (30 Minuten)
3. Maßnahmenplan umsetzen & dokumentieren (laufend)

PRICING PREVIEW:
[Starter 99 EUR/Mo] [Professional 199 EUR/Mo] [Enterprise 399 EUR/Mo]
→ Link zu /pricing

TESTIMONIALS (Platzhalter):
"NIS2 Shield hat uns Wochen an Beraterkosten gespart."
— IT-Leiter, Maschinenbau, 120 MA

FAQ (5-8 Fragen):
- Was ist NIS2?
- Bin ich betroffen?
- Was passiert wenn ich nichts tue?
- Warum nicht einfach einen Berater beauftragen?
- Ist NIS2 Shield DSGVO-konform?
- Wo werden meine Daten gespeichert?
- Kann ich kündigen?
- Gibt es eine kostenlose Testphase?

FOOTER:
Links | Impressum | Datenschutz | AGB | Kontakt
© 2026 NIS2 Shield
```

#### Pricing (`/pricing`)

```
HEADER:
"Transparent. Fair. Ohne versteckte Kosten."

3 TIER CARDS:

┌─────────────┐  ┌─────────────────┐  ┌──────────────┐
│   Starter   │  │  Professional   │  │  Enterprise  │
│  50-100 MA  │  │   100-250 MA    │  │  250-500 MA  │
│             │  │   ⭐ Beliebt    │  │              │
│  499 EUR    │  │   999 EUR       │  │  1.999 EUR   │
│  Einrichtung│  │   Einrichtung   │  │  Einrichtung │
│             │  │                 │  │              │
│  99 EUR/Mo  │  │   199 EUR/Mo    │  │  399 EUR/Mo  │
│             │  │                 │  │              │
│ ✅ Dashboard │  │ ✅ Alles Starter│  │ ✅ Alles Pro │
│ ✅ Risiko   │  │ ✅ Lieferanten  │  │ ✅ Custom    │
│ ✅ BSI-Meld.│  │ ✅ Multi-Frame. │  │ ✅ API       │
│ ✅ 10 Docs  │  │ ✅ 500 Secrets  │  │ ✅ SSO       │
│ ✅ 50 Secret│  │ ✅ 15 User      │  │ ✅ Unlimited │
│ ✅ 5 User   │  │                 │  │ ✅ Priority  │
│             │  │                 │  │              │
│ [Testen]    │  │ [Testen]        │  │ [Kontakt]    │
└─────────────┘  └─────────────────┘  └──────────────┘

FEATURE COMPARISON TABLE (vollständig)

FAQ:
- Kann ich den Plan wechseln?
- Was passiert nach der Testphase?
- Gibt es Rabatt bei jährlicher Zahlung?
- Welche Zahlungsarten akzeptieren Sie?
```

#### NIS2 Info (`/nis2`)

```
"Was ist NIS2?"
→ Erklärung in einfacher Sprache

"Bin ich betroffen?"
→ Interaktiver Quick-Check (3 Fragen → Ergebnis)

"Was muss ich tun?"
→ 10 Pflichtmaßnahmen erklärt

"Welche Fristen gelten?"
→ Timeline: Registrierung 06.03.2026, etc.

"Was passiert bei Verstoß?"
→ Bußgeld-Tabelle, persönliche Haftung GF

CTA: "Prüfen Sie Ihren Compliance-Status → Kostenlos testen"
```

#### Blog (`/blog`)

```
→ Blog-Index mit Kategorie-Filter
→ Kategorien: NIS2, IT-Sicherheit, Compliance, Updates
→ Einzelne Posts als MDX
→ Autor, Datum, Lesezeit
→ CTA am Ende jedes Posts
→ Related Posts
```

#### Impressum (`/impressum`), Datenschutz (`/datenschutz`), AGB (`/agb`)
→ Inhalte aus Stream 5

#### Kontakt (`/kontakt`)
→ Einfaches Formular: Name, E-Mail, Nachricht
→ Oder: info@nis2shield.de

### SEO Anforderungen
- Meta-Title + Description für jede Seite
- Open Graph Tags (og:title, og:description, og:image)
- Structured Data: Organization, Product, FAQ, BreadcrumbList
- sitemap.xml (automatisch via next-sitemap)
- robots.txt
- Canonical URLs
- Core Web Vitals: LCP < 2.5s, FID < 100ms, CLS < 0.1

---

## 4.2 SEO-Content (5 Artikel)

**Agent: seo-content-writer (model=sonnet)**
**Skills: `seo-keyword-cluster-builder`, `seo-optimizer`, `seo-content-planner`**

### Artikel 1: "NIS2 Checkliste 2026: Was Ihr Unternehmen jetzt tun muss"
- Target Keywords: "NIS2 Checkliste", "NIS2 Anforderungen 2026"
- Länge: 2.000 Wörter
- Struktur: H2 pro Maßnahme, Checkbox-Liste, FAQ am Ende
- CTA: "Automatisieren Sie Ihre NIS2-Checkliste mit NIS2 Shield"

### Artikel 2: "NIS2 Bußgelder: Was droht bei Nichteinhaltung?"
- Target Keywords: "NIS2 Bußgeld", "NIS2 Strafe", "NIS2 Sanktionen"
- Länge: 1.500 Wörter
- Tabelle mit Bußgeldern, Beispielrechnungen, GF-Haftung

### Artikel 3: "NIS2 vs. ISO 27001: Unterschiede und Gemeinsamkeiten"
- Target Keywords: "NIS2 ISO 27001 Unterschied", "NIS2 ISO 27001 Vergleich"
- Länge: 1.800 Wörter
- Vergleichstabelle, Mapping, "Brauche ich beides?"

### Artikel 4: "BSI-Registrierung NIS2: Schritt-für-Schritt Anleitung"
- Target Keywords: "BSI Registrierung NIS2", "NIS2 BSI melden"
- Länge: 1.500 Wörter
- Screenshots (Platzhalter), Fristen, häufige Fehler

### Artikel 5: "NIS2 für den Mittelstand: Der praktische Leitfaden"
- Target Keywords: "NIS2 Mittelstand", "NIS2 KMU"
- Länge: 2.500 Wörter
- Pillar Page, verlinkt auf alle anderen Artikel
- Praxisbeispiele, Kosten-Nutzen-Rechnung

### Alle Artikel:
- Sprache: Deutsch, professionell, verständlich
- CTA am Ende + im Text
- Interne Links zwischen den Artikeln
- H2/H3 für Featured Snippets optimiert
- FAQ-Section (für Google FAQ Rich Results)
- Als MDX in `marketing/content/blog/`

---

## 4.3 SEO Setup

**Agent: seo-specialist (model=sonnet)**
**Skills: `seo-setup`, `seo-optimizer`**

1. Google Search Console einrichten (Anleitung schreiben)
2. Sitemap.xml automatisch generieren
3. robots.txt konfigurieren
4. Schema.org Structured Data:
   - Organization
   - SoftwareApplication (für jedes Tier)
   - FAQPage (für FAQ-Sections)
   - BreadcrumbList
   - BlogPosting (für Artikel)
5. Meta-Tags Template für alle Seiten
6. hreflang Tags (de-DE primär, en als Fallback)

---

## Definition of Done — Stream 4

- [ ] Landing Page deployed auf Vercel
- [ ] Alle Seiten: Home, Pricing, NIS2-Info, Blog, Impressum, Datenschutz, AGB, Kontakt
- [ ] 5 SEO-Artikel veröffentlicht
- [ ] Structured Data auf allen Seiten
- [ ] sitemap.xml + robots.txt
- [ ] Core Web Vitals im grünen Bereich
- [ ] Mobile-responsive (375px+)
- [ ] "Kostenlos testen" Button verlinkt zur App-Registration
- [ ] Google Search Console Anleitung geschrieben

## Ausführung

```
Starte Stream 4 PARALLEL zu Streams 1-3 (keine Abhängigkeiten).

1. ZUERST: 4.1 Landing Page (frontend-developer)
2. PARALLEL: 4.2 SEO-Artikel (seo-content-writer)
3. DANACH: 4.3 SEO Setup (seo-specialist)
```
