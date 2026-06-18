# DÜRERs Café & Bistro – Website

**Live:** [duererscafehassfurt.github.io/cafe](https://duererscafehassfurt.github.io/cafe/)

Die Website des schülergeführten Cafés der Albrecht-Dürer-Mittelschule Haßfurt. Schülerinnen und Schüler kochen jeden Donnerstag frische, vegetarische Mittagsmenüs – Reservierungen werden direkt über diese Seite entgegengenommen.

---

## Seiten

| Datei | Inhalt |
|---|---|
| `index.html` | Startseite – Hero-Bereich, Menü-Vorschau, Über-uns-Abschnitt |
| `menu-reservierung.html` | Wochenmenü & Online-Reservierungsformular |
| `catering.html` | Catering- und Veranstaltungsseite |
| `anfrage.html` | Kontaktformular für Catering-/Event-Anfragen |
| `bestaetigung.html` | Bestätigungs-Landingpage (wird per E-Mail-Link aufgerufen) |
| `stornierung.html` | Stornierungsseite (wird per E-Mail-Link aufgerufen) |
| `impressum.html` | Impressum |
| `datenschutz.html` | Datenschutzerklärung |

### Gemeinsame Komponenten (`_includes/`)
- `header.html` – Sticky-Navigation mit Logo, Navigationslinks und Social-Icons
- `footer.html` – Kontaktdaten, Navigation, rechtliche Links und Schulhinweis

---

## So funktioniert es

Die Website ist eine statische Jekyll-Seite, die auf **GitHub Pages** gehostet wird. Alle dynamischen Daten (Menü, Einstellungen, Reservierungen) werden über **Google Sheets** verwaltet – ohne Datenbank oder eigenen Server.

```
Browser  →  Google Sheets (lesen, via GViz JSON API)
         →  Google Apps Script (schreiben: Buchungen, Stornierungen, E-Mails)
```

### Struktur der Google Tabelle

| Tabellenblatt | Inhalt |
|---|---|
| `Einstellungen` | Seitenweite Einstellungen: Telefon, E-Mail, Fax, Sitzplätze, Logo-URL, Social-Links |
| `Reservations` | Alle Reservierungsdatensätze |
| `EventRequests` | Eingegangene Catering-/Event-Anfragen |
| `Bilder Catering` | Bild-URLs für die Galerie auf der Catering-Seite |

Die Einstellungen werden beim Seitenaufruf geladen und dynamisch in Header, Footer und Kontaktbuttons eingefügt.

### Ablauf einer Reservierung

```
1. Gast wählt Datum, Zeitfenster und Menügerichte → Formular absenden
2. Apps Script prüft verfügbare Portionen (max. 25 pro Tag)
3. Reservierung wird als „Ausstehend" gespeichert
4. Gast erhält E-Mail mit Bestätigungslink (15 Minuten gültig)
5. Gast klickt Link → Status wird auf „Bestätigt" gesetzt
6. Nicht bestätigte Reservierungen verfallen automatisch nach 15 Minuten
7. Vergangene Reservierungen werden täglich per Trigger gelöscht
```

Eine kostenfreie Stornierung ist bis **2 Tage vor dem Reservierungsdatum um 11:00 Uhr** möglich. Der Stornierungslink ist in der Bestätigungs-E-Mail enthalten.

---

## Google Apps Script (`Code.gs`)

Als Web App bereitgestellt – alle Schreibzugriffe laufen über `doPost()`:

| Aktion | Beschreibung |
|---|---|
| `getMenu` | Gibt die Menü-CSV zurück (5-Minuten-Cache via CacheService) |
| `check` | Gibt die bereits gebuchten Portionen für ein bestimmtes Datum zurück |
| `book` | Legt eine neue Reservierung an und versendet die Bestätigungs-E-Mail |
| `confirm` | Bestätigt eine ausstehende Reservierung und sendet die finale Bestätigung |
| `cancel` | Löscht eine Reservierung (innerhalb der Frist) und sendet eine Stornierungsbestätigung |
| `eventRequest` | Speichert eine Catering-Anfrage und benachrichtigt das Café-Team per E-Mail |

`doGet()` ermöglicht die Stornierung direkt per URL – als Fallback für E-Mail-Clients ohne JavaScript.

**Zeitgesteuerte Trigger** (in Apps Script einrichten):
- `deleteExpiredReservations` – läuft alle paar Minuten, entfernt unbestätigte Buchungen und benachrichtigt die Gäste
- `deletePastReservations` – läuft täglich und bereinigt vergangene Einträge

---

## Technologie

- **Jekyll** (Static-Site-Generator, GitHub Pages)
- **Vanilla HTML/CSS/JS** – keine Frameworks
- **Google Fonts** – Playfair Display, DM Sans, DM Mono
- **Google Sheets** (GViz JSON API) – schreibgeschützte Datenquelle
- **Google Apps Script** – serverloser Backend für Buchungen und E-Mail-Versand
- **GmailApp** – Transaktions-E-Mails (Bestätigung, Stornierung, Ablauf)

---

## Lokale Entwicklung

```bash
# Jekyll installieren (Ruby erforderlich)
gem install bundler jekyll

# Lokalen Server starten
bundle exec jekyll serve

# Im Browser öffnen: http://localhost:4000
```

Der Ordner `_includes/` wird von Jekylls `{% include %}`-Tag genutzt, um Header und Footer in jede Seite einzubinden.

---

## Hinweise zum Projekt

- Alle Menüs sind **vegetarisch** und werden aus frischen, regionalen Zutaten zubereitet
- Die Kapazität ist auf **25 Portionen pro Donnerstag** begrenzt
- Bezahlung erfolgt **ausschließlich in bar** bei Abholung
- Das Café ist ein Schulprojekt – Anfragen außerhalb der Öffnungszeiten bitte telefonisch oder per E-Mail
- Catering ist für Gruppen bis zu **60 Personen** möglich; die Bistro-Kapazität wird über das Tabellenblatt `Einstellungen` verwaltet

---

*Mit ❤ gebaut von Schülerinnen und Schülern der Albrecht-Dürer-Mittelschule Haßfurt*
