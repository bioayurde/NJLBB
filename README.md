# KI-Büroassistent Dashboard — Code-Dokumentation

Single-File-HTML-Preview (`buero-ki-dashboard.html`) mit Mock-Daten. Kein Backend, keine echte Outlook-Anbindung — dient nur zur Vorschau/Abstimmung des Konzepts.

## Aufbau

- Ein HTML-File, CSS im `<style>`-Block, Logik im `<script>`-Block am Ende.
- Sidebar-Navigation schaltet zwischen `.view`-Sections um (`showView(name, btn)`).
- Farbschema: warmes Design (Anthrazit `--brand`, Orange `--accent`) als Marken-/UI-Farben; `--ok` (grün), `--warn` (gelb/braun) und `--crit` (rot) sind rein semantische Status-Farben für Pills/Kacheln, keine Grüntöne im Branding selbst.

## Views (Sections)

| ID | Inhalt |
|---|---|
| `view-heute` | Tagesübersicht: dringende E-Mails, Termine, Fristen, KI-Empfehlungen |
| `view-emails` | E-Mail-Liste mit Fachbereich-Filter (Buchhaltung/Webdesign/Marketing/Beratung) |
| `view-mandanten` | Mandantenübersicht (inkl. Mandantennummer) |
| `view-kalender` | Terminkalender + von der KI aus E-Mails erkannte Terminvorschläge |
| `view-aufgaben` | Aufgabenliste (Kanban-Spalten) |
| `view-fristen` | Fristenübersicht |
| `view-workflow` | Anhang-Workflow: 4-Schritte-Ablauf A→D (E-Mail mit PDF → Vorschau/Ausfüllen → Checkliste → Antwort-Mail senden) |
| `view-belege` | **Beleg-Workflow** (neu): automatische Anhang-Erfassung, Mandanten-Ordner, Übergabe an Buchhaltungssoftware — siehe unten |
| `view-ki` | KI-Assistent: Chat, Vorlesen, Voice-Eingabe |
| `view-einstellungen` | Einstellungen |

## JavaScript-Funktionen

**Rendering (Mock-Daten → DOM):**
- `renderHeute()`, `renderEmails()`, `renderMandanten()`, `renderKalender()`, `renderAufgaben()`, `renderFristen()`
- `prioPillClass(p)`, `initialsColor(name)`, `initialsOf(name)` — Hilfsfunktionen für Badges/Avatare

**Anhang-Workflow:**
- `wfGoto(step)` — Navigation zwischen den 4 Schritten (A–D)
- `wfSend()` — simuliert Versand der fertigen Antwort-Mail (mit Bestätigungshinweis)

**Beleg-/Ordner-Workflow (neu, `view-belege`):**
- `renderBelege()` — rendert KPI-Kacheln, Beleg-Eingangstabelle und Mandanten-Ordner-Grid aus `belegeEingang`
- `belegStatusInfo(status)`, `belegQuelleIcon(quelle)`, `mandantByNr(nr)` — Hilfsfunktionen für Status-Pills, Quellen-Icon und Mandanten-Lookup über die Mandantennummer
- `belegPruefeNeueMails()` — simuliert das automatische Herunterladen neuer E-Mail-Anhänge (zieht einen Eintrag aus `belegNeuPool` in die Warteschlange)
- `belegManuellZuordnen(id)` — ordnet einen Beleg mit Status „Manuelle Prüfung nötig“ über das Inline-Dropdown einem Mandanten zu
- `belegSendenAnSoftware(nr)` — simuliert die Übergabe aller zugeordneten, noch nicht übergebenen Belege eines Mandanten-Ordners an die Buchhaltungssoftware
- `belegUploadManuell(event)` — manueller Upload (z. B. WhatsApp-Weiterleitung) inkl. Mandantenauswahl und Notiz, direkt zugeordnet
- `belegUploadKamera(event)` — Upload eines mit der Handy-Kamera aufgenommenen Fotos (`<input type="file" accept="image/*" capture="environment">`), landet unzugeordnet in der Prüf-Warteschlange
- `populateBelegMandantSelect()` — befüllt das Mandanten-Dropdown im Upload-Formular beim Start
- `showToast(msg)` — kurze Bestätigungsmeldung unten rechts (genutzt von allen Beleg-Aktionen)

**KI-Assistent / Voice:**
- `askAI(question)` — Haupt-Logik: prüft zuerst auf Termin-Detailfrage (`findTerminDetailFrage`), sonst Lookup in `aiResponses`
- `findTerminDetailFrage(key)` — erkennt Zahlwörter/Ziffern + "termin"+"detail" in der Frage
- `terminDetailAntwort(idx)` — formatiert Detailantwort aus `terminDetails`-Array
- `addMsg(role, html)` — fügt Chat-Nachricht hinzu
- `speak(text)` — Sprachausgabe via Web Speech API (`SpeechSynthesisUtterance`, de-DE)
- `readDashboard()` — liest `dashboardSummary` vor
- `toggleVoice()` — simuliert Mikrofon-Eingabe (zufälliger Beispielsatz, da echter Mikrofonzugriff in der Preview nicht möglich)

**Sonstiges:**
- `showView(name, btn)` — Sidebar-Navigation
- `toggleSidebar(open)` — mobile Sidebar

## Daten-Konstanten (Mock)

- `mandanten` — Mandantenliste inkl. `nr` (Mandantennummer), Branche, offene Mails/Aufgaben, nächste Frist, Status
- `emails`, `fristen`, `aufgaben`, `termine`, `aiRecos` — Daten für die jeweiligen Übersichts-Views
- `belegeEingang` — Beleg-Eingangs-Warteschlange (Quelle E-Mail/WhatsApp/Kamera, Status `auto`/`pruefen`/`manuell`/`gesendet`, verknüpfte Mandantennummer)
- `belegNeuPool` — Beispiel-Anhänge, aus denen `belegPruefeNeueMails()` zufällig einen neuen Eingang simuliert
- `aiResponses` — Frage→Antwort-Mapping (lowercase Key) für den KI-Chat
- `terminDetails` — Array mit Termin-Details (Titel, Zeit, Mandant, Thema, Ort, Vorbereitung)
- `dashboardSummary` — Text für "Dashboard vorlesen"
- `zahlwoerter` — Mapping deutscher Ordnungswörter ("zweiten" etc.) auf Array-Index

## Beleg-/Ordner-Workflow im Detail (`view-belege`)

Als Mock-up umgesetzte erste Version des ursprünglich offenen Bausteins „Automatischer Beleg-/Ordner-Workflow":

1. **Anhänge automatisch herunterladen** — Button „Neue E-Mail-Anhänge prüfen" simuliert das Nachladen neuer Anhänge in die Warteschlange.
2. **Ordner pro Mandantennummer** — Sobald ein Beleg zugeordnet ist, erscheint automatisch eine Ordnerkarte in „Mandanten-Ordner", gruppiert über `mandant.nr`.
3. **Weitergabe an Buchhaltungssoftware** — Button je Ordner übergibt alle offenen Belege (simuliert, Status wechselt zu „übergeben").
4. **Automatische Zuordnung bei eindeutigen Dokumenten** — E-Mail-Anhänge mit klarem Mandantenbezug (z. B. Finanzamt-Schreiben mit Steuernummer/Mandantenname) landen direkt mit Status „Automatisch zugeordnet", ohne Rückfrage.
5. **Manueller Upload mit Notiz** — Formular für Uploads aus anderen Quellen (z. B. WhatsApp) mit Pflichtfeld „Mandant" und Freitext-Notiz zur Zuordnung.
6. **Upload per Handy-Kamera** — eigenes Formular mit `capture="environment"`, landet zur weiteren Zuordnung in der Prüf-Warteschlange.

Wichtig: Die „automatische Erkennung" ist in dieser Preview **vordefiniert in den Mock-Daten** (kein echtes Auslesen von Dateiinhalten/OCR) — sie zeigt das Konzept und die Bedienung, nicht die produktive Logik.

## Offen / nächster Baustein (noch nicht umgesetzt)

- Echtes automatisches Herunterladen von E-Mail-Anhängen (Microsoft Graph / Outlook-Anbindung), statt simuliertem Pool
- Echte inhaltliche Dokumentenerkennung (OCR/Textanalyse) statt Heuristik auf Mock-Daten, um Mandant/Steuernummer wirklich aus dem Dokument zu lesen
- Echte Anbindung an die eigene Buchhaltungssoftware (API/Dateiexport) zum tatsächlichen Auslesen der Belege
- Persistente Speicherung der Ordnerstruktur (aktuell nur im Browser-Speicher der Sitzung)
- Echter Kamera-/Dateizugriff auf dem Endgerät für den Produktivbetrieb (aktuell Standard-`<input type="file">`)
- Echte Outlook-Anbindung für die übrigen Views (E-Mails, Kalender, Termine)
