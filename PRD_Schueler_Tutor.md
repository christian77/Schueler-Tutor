# PRD: KI-gestützter Schüler-Tutor mit AI Harness

**Version:** 1.0  
**Datum:** Juni 2026  
**Status:** Entwurf  
**Zielgruppe:** Privat, Familie

---

## 1. Executive Summary

Der KI-gestützte Schüler-Tutor ist ein selbst gehostetes, personalisiertes Lernsystem für einen 12-jährigen Gymnasialschüler (Klasse 6, NRW). Das System kombiniert einen großen Sprachmodell-basierten Tutoren mit einem robusten AI Harness, der sicherstellt, dass die KI ausschließlich als pädagogischer Assistent agiert — nicht als Hausaufgaben-Löser.

Der Tutor startet mit dem Fach Latein (Lehrbuch: Campus A, 4. Auflage, C.C. Buchner) und ist von Grund auf erweiterbar für weitere Schulfächer. Ein zentrales Alleinstellungsmerkmal ist die Digitalisierung des echten Schulbuchs: Vokabellisten, Grammatikregeln und Lesetexte werden per Claude Vision OCR extrahiert, strukturiert in einer Datenbank abgelegt und dem Tutor zur Laufzeit präzise zur Verfügung gestellt.

Eltern erhalten ein separates Dashboard mit wöchentlichen Reports, Lernfortschritt, Schwächen-Analyse und proaktiven Benachrichtigungen. Der gesamte Stack läuft in Docker — zunächst auf einem lokalen Proxmox-Container, später auf einem Hetzner VPS in Deutschland (DSGVO-konform).

**MVP-Ziel:** Ein voll funktionsfähiger Latein-Tutor mit AI Harness, Eltern-Dashboard, Lehrbuch-Integration und Kostensteuerung, deploybar auf einem einzelnen VPS innerhalb von 4 Wochen.

---

## 2. Mission

**Mission:** Jedem Schüler einen geduldigen, personalisierten Tutor zur Seite stellen, der Schwächen systematisch erkennt und abbaut, Stärken weiter festigt, und dabei nie die Hausaufgaben abnimmt — sondern das echte Verstehen fördert.

**Kernprinzipien:**

1. **Pädagogik vor Effizienz** — Der Tutor führt durch Fragen zur Erkenntnis, löst nicht für den Schüler.
2. **Harness ist unumgehbar** — Kein API-Key im Browser, alle Sicherheitsebenen laufen serverseitig.
3. **Konfiguration statt Code** — Neue Fächer, LLMs und Tools werden per YAML-Konfiguration ergänzt, nicht durch Code-Änderungen.
4. **Eltern behalten den Überblick** — Transparenz über Lernverhalten, Fortschritt und Kosten zu jeder Zeit.
5. **Datensparsamkeit** — Nur was pädagogisch notwendig ist wird gespeichert; nach 90 Tagen werden Volltext-Chats zu Statistiken aggregiert.

---

## 3. Zielnutzer

### Primärer Nutzer: Schüler

- **Name/Persona:** Leon, 12 Jahre, Klasse 6, Gymnasium NRW
- **Geräte:** Laptop (primär), Android-Smartphone (sekundär)
- **Technisches Level:** Digital native, gewohnt an Chat-Interfaces (WhatsApp, YouTube)
- **Bedürfnisse:** Schnelle, verständliche Erklärungen; Motivation durch Erfolgserlebnisse; kein Gefühl von Überforderung
- **Schmerzpunkte:** Latein-Vokabeln vergessen, Deklinationstabellen verwirren, fehlende Lernroutine

### Sekundärer Nutzer: Elternteil

- **Persona:** Technisch versierter Elternteil mit Linux/Docker-Kenntnissen
- **Bedürfnisse:** Überblick über Lernfortschritt, Kontrolle über Kosten und Inhalte, proaktive Benachrichtigungen bei Auffälligkeiten
- **Schmerzpunkte:** Keine Transparenz bei kommerzielle Nachhilfe-Apps; Sorge vor unkontrollierter KI-Nutzung

### Tertiärer Nutzer: System-Administrator (= Elternteil)

- **Aufgaben:** VPS-Betrieb, Updates einspielen, neue Fächer konfigurieren, Lehrbuch-Digitalisierung durchführen

---

## 4. MVP Scope

### Core Functionality

✅ Latein-Tutor-Chat mit Persona "Magister Felix"  
✅ AI Harness (Input-Filter, Output-Validator, System-Prompt-Lock, Cost-Controller)  
✅ Drei Modi: Tutor (freies Gespräch), Vokabel-Trainer, Grammatik-Übungen  
✅ Session-basiertes Progress-Tracking (passive Fehler-Tagging während Chat)  
✅ Aktive Session-Zusammenfassung am Ende jeder Lernsitzung  
✅ Streak-Anzeige und Fortschrittsbalken pro Thema (leichte Gamification)  
✅ Schüler-Login (Username + Passwort, JWT)  
✅ Eltern-Dashboard mit Lernstatistiken  
✅ Eltern-Login (separates JWT-Secret, separate Routen)  
✅ Wöchentlicher E-Mail-Report an Eltern  
✅ Proaktive Alerts (Lernausfall >3 Tage, Kostenschwelle, Sicherheitsauffälligkeit)  
✅ Noten- und Klassenarbeit-Eingabe (Eltern + Schüler)  
✅ Foto/Scan-Upload von Klassenarbeiten mit automatischer Fehlertyp-Extraktion  
✅ Lehrbuch-Digitalisierungs-Pipeline (Scan → OCR → JSON → DB)  
✅ Vokabeldatenbank aus Campus A (Lektionen 1–16)  
✅ Tägliches Token-Limit mit Warn- und Stop-Schwelle  
✅ Datenhaltung: 90 Tage Volltext, danach Aggregation  
✅ Täglicher PostgreSQL-Backup-Dump (lokal)  
✅ Wöchentlicher Hetzner-Snapshot  
✅ Docker-basiertes Deployment (Proxmox lokal + Hetzner VPS)  
✅ PWA (Progressive Web App) für Browser + Android  

### Out of Scope (Phase 2+)

❌ Weitere Schulfächer (Mathematik, Englisch, Geschichte)  
❌ Wolfram Alpha / Python-Interpreter Tool für Mathematik  
❌ Vollständige Gamification (Punkte, Abzeichen, Level)  
❌ Allgemeiner Off-Topic-Agent  
❌ Unterschiedliche LLMs pro Fach (Konfiguration vorbereiten, aber nicht aktivieren)  
❌ Blue-Green-Deployment / Zero-Downtime-Updates  
❌ Multi-User-Betrieb (mehrere Schüler, andere Familien)  
❌ Native Mobile App (iOS/Android)  
❌ Sprachein- und -ausgabe  
❌ Automatische Lehrplan-Synchronisation mit Schule  

---

## 5. User Stories

### Schüler

**US-01: Vokabeln üben**  
Als Schüler möchte ich Vokabeln aus einer bestimmten Lektion abgefragt bekommen, damit ich mich gezielt auf einen Vokabeltest vorbereiten kann.  
*Beispiel: "Frag mich Vokabeln aus Lektion 5" → Tutor fragt Latein→Deutsch und Deutsch→Latein im Wechsel, gibt Hinweise bei Fehlern, zeigt am Ende Trefferquote.*

**US-02: Grammatik verstehen**  
Als Schüler möchte ich eine Grammatikregel erklärt bekommen wenn ich sie nicht verstehe, damit ich sie in Übungen anwenden kann.  
*Beispiel: "Ich verstehe die a-Deklination nicht" → Tutor erklärt mit Beispielen aus Campus A, stellt Kontrollfragen, gibt gezielte Übungsaufgaben.*

**US-03: Textstelle bearbeiten**  
Als Schüler möchte ich bei einem lateinischen Text Hilfe bekommen ohne dass der Tutor ihn direkt übersetzt, damit ich selbst das Übersetzen lerne.  
*Beispiel: Schüler tippt einen Satz aus dem Schulbuch → Tutor stellt Leitfragen: "Welcher Kasus ist 'puellam'? Was ist die Grundform?"*

**US-04: Klassenarbeit eingeben**  
Als Schüler möchte ich meine Klassenarbeit fotografieren und hochladen können, damit der Tutor meine Fehler kennt und wir gezielt daran arbeiten.

**US-05: Fortschritt sehen**  
Als Schüler möchte ich sehen wie viele Tage ich in Folge gelernt habe und wie gut ich in verschiedenen Themen bin, damit ich motiviert bleibe.

### Elternteil

**US-06: Wöchentlicher Überblick**  
Als Elternteil möchte ich jeden Sonntag eine E-Mail mit einer Zusammenfassung der Lernwoche erhalten, damit ich den Fortschritt meines Sohnes im Blick behalte.  
*Enthält: Lernminuten, Vokabel-Trefferquote, häufigste Fehlertypen, Streak, Vergleich zur Vorwoche.*

**US-07: Noten eintragen**  
Als Elternteil möchte ich Schulnoten und Klassenarbeiten ins Dashboard eintragen können, damit der Tutor diesen Kontext bei der Trainingsplanung berücksichtigt.

**US-08: Alert bei Lernausfall**  
Als Elternteil möchte ich eine Benachrichtigung erhalten wenn mein Sohn mehr als 3 Tage nicht gelernt hat, damit ich ihn rechtzeitig ansprechen kann.

**US-09: Kostenübersicht**  
Als Elternteil möchte ich sehen wie viele API-Tokens verbraucht wurden und was das kostet, damit ich die monatlichen Ausgaben kontrollieren kann.

**US-10: Lehrbuch-Digitalisierung**  
Als Administrator möchte ich Seiten des Schulbuchs fotografieren und als strukturierte Vokabel- und Grammatikdaten importieren können, damit der Tutor präzise auf das tatsächliche Lehrmaterial zugreift.

---

## 6. Core-Architektur & Patterns

### Überblick

```
┌─────────────────────────────────────────────┐
│           Frontend (PWA / React)            │
│   Schüler-Chat │ Vokabel-Trainer │ Eltern   │
└──────────────────────┬──────────────────────┘
                       │ HTTPS / JWT
┌──────────────────────▼──────────────────────┐
│              Auth-Schicht                   │
│   Rate-Limiting · JWT-Validierung           │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│           Backend (FastAPI / Python)        │
│  ┌─────────────────────────────────────┐   │
│  │           AI Harness                │   │
│  │  System-Prompt-Lock                 │   │
│  │  Input-Filter → LLM → Output-Val.  │   │
│  │  Progress-Tracker                   │   │
│  │  Cost-Controller                    │   │
│  │  Conversation-Manager               │   │
│  └──────────────┬──────────────────────┘   │
│                 │                           │
│  Usage-Logger   │   Lehrbuch-Pipeline       │
└─────────────────┼───────────────────────────┘
                  │
        ┌─────────▼──────────┐
        │   LLM-API          │
        │  (Adapter-Pattern) │
        │  Claude Haiku 3.5  │
        └────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│           PostgreSQL                        │
│  Sessions │ Progress │ Vokabeln │ Logs      │
└─────────────────────────────────────────────┘
```

### Adapter-Pattern für LLM-Austausch

```python
# base_llm.py
class BaseLLMAdapter:
    async def complete(self, messages: list, system: str) -> str:
        raise NotImplementedError

# claude_adapter.py
class ClaudeAdapter(BaseLLMAdapter):
    async def complete(self, messages, system):
        # Anthropic API call
        ...

# openai_adapter.py
class OpenAIAdapter(BaseLLMAdapter):
    async def complete(self, messages, system):
        # OpenAI API call
        ...
```

### Fach-Konfiguration per YAML

```yaml
# subjects/latein.yaml
name: Latein
klasse: 6
lehrbuch: "Campus A, 4. Auflage"
llm_adapter: claude_haiku
persona: magister_felix
themen_erlaubt:
  - latein
  - roemische_geschichte
  - allgemeines_lernen
  - smalltalk_kurz
fehler_taxonomie:
  - deklination_a
  - deklination_o
  - konjugation_praesens
  - konjugation_imperfekt
  - konjugation_perfekt
  - vokabel
  - kasus_verwechslung
tools: []  # keine externen Tools für Latein

# subjects/mathematik.yaml (Phase 2)
name: Mathematik
llm_adapter: gpt4o_mini
tools:
  - calculator
  - wolfram_alpha
```

### Projektstruktur

```
tutor/
├── docker-compose.yml
├── .env.example
├── backend/
│   ├── Dockerfile
│   ├── main.py
│   ├── harness/
│   │   ├── system_prompt.py
│   │   ├── input_filter.py
│   │   ├── output_validator.py
│   │   ├── cost_controller.py
│   │   └── conversation_manager.py
│   ├── adapters/
│   │   ├── base_llm.py
│   │   ├── claude_adapter.py
│   │   └── openai_adapter.py
│   ├── pipeline/
│   │   ├── ocr_extractor.py       # Lehrbuch-Digitalisierung
│   │   ├── vocab_importer.py
│   │   └── homework_analyzer.py   # Klassenarbeit-Analyse
│   ├── routers/
│   │   ├── chat.py
│   │   ├── vocab.py
│   │   ├── parent.py
│   │   └── upload.py
│   ├── models/
│   │   └── database.py
│   └── subjects/
│       └── latein.yaml
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   │   ├── views/
│   │   │   ├── StudentChat.jsx
│   │   │   ├── VocabTrainer.jsx
│   │   │   └── ParentDashboard.jsx
│   │   └── components/
└── nginx/
    └── nginx.conf
```

---

## 7. Komponenten & Features

### 7.1 AI Harness

Der Harness ist die Kernkomponente. Er sitzt zwischen Frontend und LLM-API und ist serverseitig — für den Schüler unsichtbar und nicht umgehbar.

**System-Prompt-Lock**

Der System-Prompt wird serverseitig zusammengesetzt und niemals an den Client übertragen. Er enthält:
- Persona-Definition (Magister Felix, Alter der Zielgruppe, Tonalität)
- Pädagogische Regeln (keine direkten Lösungen, Sokrates-Methode)
- Aktuelle Lektion und Lernstand (aus DB geladen)
- Erlaubte Themen
- Fehler-Tagging-Instruktionen (JSON-Tags am Ende jeder Antwort)

Nutzereingaben werden ausschließlich als `user`-Nachrichten übergeben — nie als `system`.

**Input-Filter**

```python
class InputFilter:
    BLOCKED_PATTERNS = [
        r"ignoriere.*anweisung",
        r"du bist jetzt",
        r"vergiss.*regeln",
        r"als.*ki ohne einschränkungen",
    ]
    
    def check(self, text: str, subject_config: dict) -> FilterResult:
        # 1. Jailbreak-Pattern-Check (Regex)
        # 2. Themen-Relevanz-Check (LLM-basiert, leichtgewichtig)
        # 3. Längen-Limit (max 500 Zeichen)
        # Returns: FilterResult(allowed: bool, reason: str)
```

**Output-Validator**

Prüft jede LLM-Antwort vor der Auslieferung:
- Enthält die Antwort eine direkte Hausaufgaben-Lösung? → Neu-Generierung mit strengerem Prompt
- Sind Fehler-Tags vorhanden? → In DB speichern, aus Antwort entfernen
- Länge > 800 Zeichen? → Kürzen mit Fortsetzungs-Option
- Auffällige Inhalte? → Loggen, Eltern-Alert auslösen

**Cost-Controller**

```python
DAILY_TOKEN_LIMIT = 50_000      # ~100 Konversationsrunden
WARNING_THRESHOLD = 0.80        # Alert an Eltern bei 80%
HARD_STOP_THRESHOLD = 1.00      # Freundliche Stop-Nachricht an Schüler

# Freundliche Stop-Nachricht:
# "Für heute haben wir genug Latein gepaukt! 
#  Magister Felix macht jetzt Pause. Bis morgen! 🏛️"
```

**Progress-Tracker**

Das LLM taggt intern jeden Fehler in strukturierter Form:

```
[TUTOR_TAG: {"fehler": "deklination_a", "wort": "puellam", "lektion": 5}]
```

Der Output-Validator extrahiert diese Tags, entfernt sie aus der sichtbaren Antwort und speichert sie in der Datenbank. Am Ende jeder Session (Inaktivität > 10 Minuten) generiert der Harness automatisch eine Session-Zusammenfassung für den Schüler.

### 7.2 Lehrbuch-Digitalisierungs-Pipeline

**Workflow:**

```
Phase 1: Aufnahme
  Smartphone-Foto oder Scanner-PDF der Buchseite
  → Upload über Admin-Endpoint (nur Eltern-Token)

Phase 2: OCR mit Claude Vision
  → claude-haiku-3-5 mit Vision
  → Strukturierter Prompt: "Extrahiere alle Vokabeln aus diesem Bild 
     als JSON mit Feldern: latein, genus, genitiv, deutsch, wortart"
  → Ergebnis: strukturiertes JSON

Phase 3: Validierung & Import
  → Automatische Plausibilitätsprüfung (Latein-Zeichensatz, Pflichtfelder)
  → Manuelle Review-UI im Eltern-Dashboard (Korrekturen möglich)
  → Import in PostgreSQL

Phase 4: Nutzung
  → Harness lädt zur Laufzeit nur aktuelle Lektion
  → Vokabel-Trainer greift direkt auf DB zu
```

**Implementierung OCR-Extraktion:**

```python
async def extract_vocab_from_image(image_base64: str, lektion: int) -> list[dict]:
    prompt = """
    Extrahiere alle Vokabeln aus diesem Lehrbuch-Bild.
    Antworte NUR mit einem JSON-Array, kein weiterer Text.
    Format:
    [
      {
        "latein": "puella",
        "genus": "f",
        "genitiv": "puellae", 
        "dekl_konj": "a-Dekl.",
        "deutsch": "das Mädchen",
        "wortart": "Substantiv"
      }
    ]
    Bei Verben: Felder "stammformen" statt "genus"/"genitiv".
    """
    response = await claude_vision.complete(
        image=image_base64,
        prompt=prompt
    )
    return json.loads(response)
```

**Priorisierung der Digitalisierung:**

| Priorität | Inhalt | Aufwand | Nutzen |
|-----------|--------|---------|--------|
| 1 | Vokabellisten Lektionen 1–10 | ~2h | Sehr hoch |
| 2 | Grammatik-Übersichten | ~3h | Hoch |
| 3 | Lesetexte (Latein-Original) | ~4h | Hoch |
| 4 | Inhaltsverzeichnis / Lehrplan | ~30min | Mittel |
| 5 | Übungsaufgaben-Typen | ~2h | Mittel |

**Urheberrecht:** Die Digitalisierung ist für den privaten Eigengebrauch eines einzelnen Schülers nach §53 UrhG (Privatkopie) zulässig. Das Material darf nicht öffentlich zugänglich gemacht oder mit anderen geteilt werden.

### 7.3 Klassenarbeit-Analyse

Schüler oder Eltern fotografieren eine korrigierte Klassenarbeit. Das System:
1. Führt OCR mit Claude Vision durch
2. Extrahiert Fehlertypen automatisch: "Roter Strich bei 'puellam' → deklination_a"
3. Trägt Note und Fehler-Kategorien in die Datenbank ein
4. Passt den Trainingsplan automatisch an (mehr Übungen zu erkannten Schwächen)

### 7.4 Eltern-Dashboard

**Übersichts-Widgets:**
- Lern-Streak (aktuelle Tage in Folge)
- Wochenstunden (letzte 4 Wochen als Balkendiagramm)
- Vokabel-Trefferquote pro Lektion
- Grammatik-Schwächen-Radar (Top 5 Fehlertypen)
- API-Kosten (aktueller Monat, Tagesschnitt, Prognose)
- Letzte Klassenarbeit (Note, Datum, Trend)

**Noten-Eingabe:**
```
Fach: Latein
Datum: [Datepicker]
Art: [Klassenarbeit / Vokabeltest / Mündlich]
Note: [1-6]
Kommentar: [optional]
Scan: [Foto/PDF upload]
```

**Alert-Konfiguration (Eltern):**
- Lernausfall: [X] Tage ohne Session → E-Mail
- Kosten-Warnung: Token-Budget [X]% erreicht → E-Mail
- Sicherheit: Jailbreak-Versuch erkannt → Sofort-E-Mail
- Wöchentlicher Report: [Wochentag] [Uhrzeit]

---

## 8. Technology Stack

### Backend

| Komponente | Technologie | Version | Begründung |
|------------|-------------|---------|------------|
| Framework | FastAPI | 0.115+ | Async, schnell, automatische API-Docs |
| Sprache | Python | 3.12 | LLM-Ökosystem, OCR-Libraries |
| Datenbank | PostgreSQL | 16 | Robust, JSON-Support, bewährt |
| ORM | SQLAlchemy | 2.0 | Async-Support |
| Auth | python-jose | 3.3 | JWT-Handling |
| E-Mail | fastapi-mail | 1.4 | SMTP, HTML-Templates |
| Scheduling | APScheduler | 3.10 | Cronjobs (Weekly Report, Backup) |
| File-Upload | python-multipart | – | Foto/Scan-Uploads |

### Frontend

| Komponente | Technologie | Version | Begründung |
|------------|-------------|---------|------------|
| Framework | React | 18 | Bewährt, PWA-Support |
| Build | Vite | 5 | Schnell, modernes Tooling |
| Styling | Tailwind CSS | 3 | Utility-first, kein Design-System nötig |
| Charts | Recharts | 2 | Einfach, React-nativ |
| PWA | vite-plugin-pwa | – | Offline-fähig, installierbar |
| HTTP | Axios | – | API-Calls |

### Infrastruktur

| Komponente | Technologie | Begründung |
|------------|-------------|------------|
| Containerisierung | Docker + Docker Compose | Portabel Proxmox → Hetzner |
| Reverse Proxy | Nginx | Static Files + API-Proxy |
| TLS | Let's Encrypt / Certbot | Kostenloses HTTPS |
| Hosting Produktion | Hetzner CX22 (DE) | DSGVO, günstig, zuverlässig |
| Hosting Entwicklung | Proxmox LXC | Lokales Testen |
| Backups DB | pg_dump + rclone | Täglich lokal |
| Backups Server | Hetzner Snapshot | Wöchentlich |

### LLM

| Zweck | Modell | Kosten (ca.) |
|-------|--------|--------------|
| Tutor-Chat (Standard) | Claude Haiku 3.5 | ~0,08€/1000 Nachrichten |
| Lehrbuch-OCR (einmalig) | Claude Haiku 3.5 Vision | ~0,02€/Seite |
| Klassenarbeit-Analyse | Claude Haiku 3.5 Vision | ~0,05€/Arbeit |
| Kostenschätzung/Monat | – | 2–5€ bei normaler Nutzung |

---

## 9. Security & Configuration

### Authentication

```
Schüler-Token:
  - JWT, Secret: STUDENT_JWT_SECRET (env)
  - Expiry: 7 Tage (auto-refresh)
  - Scope: chat, vocab, upload_homework

Eltern-Token:
  - JWT, Secret: PARENT_JWT_SECRET (anderes Secret!)
  - Expiry: 24h
  - Scope: dashboard, notes, alerts, admin, upload_book
```

### Environment Variables

```bash
# .env (niemals in Git!)
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/tutor
ANTHROPIC_API_KEY=sk-ant-...
STUDENT_JWT_SECRET=<random-256-bit>
PARENT_JWT_SECRET=<random-256-bit-different>
SMTP_HOST=smtp.example.com
SMTP_USER=tutor@example.com
SMTP_PASS=...
PARENT_EMAIL=eltern@example.com
DAILY_TOKEN_LIMIT=50000
BACKUP_DEST=/backups/  # oder rclone-remote
```

### AI Harness Security

- API-Key lebt ausschließlich serverseitig — niemals im Browser oder Frontend-Code
- System-Prompt wird niemals an Client übertragen
- Alle Harness-Ebenen sind serverseitig und können vom Schüler nicht umgangen werden
- Rate-Limiting: max. 30 Requests/Minute pro Session
- Maximale Input-Länge: 500 Zeichen pro Nachricht
- Jailbreak-Versuche werden geloggt und lösen Eltern-Alert aus

### Datenschutz (DSGVO)

- Server in Deutschland (Hetzner FSN/NBG)
- Chat-Volltext: 90 Tage Aufbewahrung, danach Aggregation zu Statistiken
- Fotos von Klassenarbeiten: nach Analyse-Extraktion optional löschbar
- Kein Tracking, keine Werbung, keine Drittanbieter außer LLM-API
- Lehrbuch-Scans: nur privat, nicht zugänglich von außen

---

## 10. API-Spezifikation

### Schüler-Endpunkte

```
POST /api/chat
Auth: Student-JWT
Body: { "message": "string", "session_id": "uuid" }
Response: { "reply": "string", "session_id": "uuid", "tags": [] }

POST /api/vocab/session
Auth: Student-JWT  
Body: { "lektion": 5, "mode": "lat_de" | "de_lat" }
Response: { "question": "string", "session_id": "uuid" }

POST /api/vocab/answer
Auth: Student-JWT
Body: { "session_id": "uuid", "answer": "string" }
Response: { "correct": bool, "feedback": "string", "next": "string" }

GET /api/progress/me
Auth: Student-JWT
Response: { "streak": 7, "vokabel_quote": 0.82, "schwaechen": [...] }

POST /api/upload/homework
Auth: Student-JWT | Parent-JWT
Body: multipart/form-data (image/pdf)
Response: { "upload_id": "uuid", "status": "processing" }
```

### Eltern-Endpunkte

```
GET /api/parent/dashboard
Auth: Parent-JWT
Response: { "streak": 7, "weekly_minutes": 145, "costs": {...}, ... }

POST /api/parent/grade
Auth: Parent-JWT
Body: { "fach": "latein", "datum": "2026-06-15", "note": 3, "art": "Klassenarbeit" }

GET /api/parent/alerts
Auth: Parent-JWT
Response: { "alerts": [{ "type": "learning_gap", "days": 4, "ts": "..." }] }

POST /api/admin/book/upload
Auth: Parent-JWT (admin scope)
Body: multipart/form-data (image)
Query: lektion=5&typ=vokabeln|grammatik|text
Response: { "extracted": [...], "review_id": "uuid" }

POST /api/admin/book/confirm
Auth: Parent-JWT
Body: { "review_id": "uuid", "items": [...] }  # nach manuellem Review
```

---

## 11. Datenbankschema

```sql
-- Kernentitäten

CREATE TABLE sessions (
  id UUID PRIMARY KEY,
  started_at TIMESTAMPTZ,
  ended_at TIMESTAMPTZ,
  fach VARCHAR(50),
  modus VARCHAR(20),  -- tutor|vokabeln|grammatik
  total_tokens INTEGER,
  summary TEXT  -- Ende-der-Session-Zusammenfassung
);

CREATE TABLE messages (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id),
  role VARCHAR(10),  -- user|assistant
  content TEXT,
  created_at TIMESTAMPTZ,
  -- Wird nach 90 Tagen gelöscht (nur Aggregate bleiben)
  expires_at TIMESTAMPTZ DEFAULT NOW() + INTERVAL '90 days'
);

CREATE TABLE error_tags (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id),
  fehler_typ VARCHAR(50),  -- deklination_a, vokabel, etc.
  wort VARCHAR(100),
  lektion INTEGER,
  created_at TIMESTAMPTZ
);

CREATE TABLE grades (
  id UUID PRIMARY KEY,
  fach VARCHAR(50),
  datum DATE,
  art VARCHAR(30),  -- Klassenarbeit|Vokabeltest|Muendlich
  note DECIMAL(2,1),
  kommentar TEXT,
  scan_path VARCHAR(255),
  eingegeben_von VARCHAR(10),  -- schueler|eltern
  created_at TIMESTAMPTZ
);

CREATE TABLE vocab (
  id UUID PRIMARY KEY,
  lektion INTEGER,
  latein VARCHAR(100),
  deutsch VARCHAR(200),
  genus VARCHAR(5),
  genitiv VARCHAR(100),
  dekl_konj VARCHAR(50),
  wortart VARCHAR(30),
  stammformen VARCHAR(200)
);

CREATE TABLE vocab_results (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id),
  vocab_id UUID REFERENCES vocab(id),
  correct BOOLEAN,
  created_at TIMESTAMPTZ
);

CREATE TABLE usage_log (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES sessions(id),
  datum DATE,
  tokens_in INTEGER,
  tokens_out INTEGER,
  cost_eur DECIMAL(8,6),
  created_at TIMESTAMPTZ
);

CREATE TABLE streaks (
  id UUID PRIMARY KEY,
  current_streak INTEGER DEFAULT 0,
  longest_streak INTEGER DEFAULT 0,
  last_activity DATE,
  updated_at TIMESTAMPTZ
);
```

---

## 12. Erfolgskriterien

### MVP-Definition

Das MVP gilt als erfolgreich wenn:

✅ Schüler kann einen vollständigen Latein-Lernabend (30–60 Min) ohne Unterbrechung durchführen  
✅ AI Harness blockiert mindestens 95% aller getesteten Jailbreak-Versuche  
✅ Tutor löst keine Hausaufgaben direkt (manuelle Stichproben-Tests bestanden)  
✅ Vokabel-Trainer fragt korrekte Vokabeln aus Campus A ab (nach Digitalisierung Lektionen 1–5)  
✅ Eltern-Dashboard zeigt korrekte Statistiken der letzten 7 Tage  
✅ Wöchentlicher Report wird automatisch per E-Mail versendet  
✅ Täglicher Backup-Dump funktioniert und ist wiederherstellbar (Test-Restore)  
✅ PWA ist auf Android-Smartphone installierbar und funktionsfähig  
✅ Monatliche LLM-Kosten unter 10€ bei normaler Nutzung  

### Qualitätsindikatoren (nach 4 Wochen Betrieb)

- Schüler nutzt den Tutor mindestens 4x/Woche
- Vokabel-Trefferquote steigt über 4 Wochen messbar
- Keine ungeplanten Downtimes > 30 Minuten
- Kein Datenverlust

---

## 13. Implementierungsphasen

### Phase 1: Fundament (Woche 1–2)

**Ziel:** Lauffähiger Tutor-Chat mit Harness auf Proxmox

Deliverables:
✅ Docker Compose Setup (Backend + Frontend + PostgreSQL + Nginx)  
✅ FastAPI-Grundstruktur mit Router-Trennung  
✅ PostgreSQL-Schema und Migrationen (Alembic)  
✅ JWT-Auth für Schüler und Eltern  
✅ Claude Haiku Adapter (Basis-Integration)  
✅ AI Harness: System-Prompt-Lock + Input-Filter + Output-Validator  
✅ Cost-Controller mit Tages-Limit  
✅ Einfacher React-Chat (kein Design, nur Funktion)  
✅ Latein.yaml Fach-Konfiguration  

**Validierung:** Manuelle Jailbreak-Tests, Kostenmonitoring aktiv, Chat läuft stabil

---

### Phase 2: Lehrbuch & Tracking (Woche 2–3)

**Ziel:** Buchinhalt integriert, Fortschritt wird getrackt

Deliverables:
✅ OCR-Pipeline: Upload → Claude Vision → JSON-Extraktion  
✅ Admin-Review-UI für extrahierte Vokabeln  
✅ Vokabeldatenbank befüllt (Lektionen 1–10)  
✅ Vokabel-Trainer-Modus (Latein↔Deutsch, Trefferquote)  
✅ Fehler-Tagging im Output-Validator  
✅ Progress-Tracker (passive Fehler-Speicherung)  
✅ Session-Zusammenfassung am Ende  
✅ Streak-Anzeige + Fortschrittsbalken im Frontend  
✅ Klassenarbeit-Upload + Fehlertyp-Extraktion  

**Validierung:** 5 Vokabellektionen vollständig digitalisiert, Fehler-Tags werden korrekt gespeichert

---

### Phase 3: Eltern & Benachrichtigungen (Woche 3–4)

**Ziel:** Eltern-Dashboard vollständig, Alerts funktionieren

Deliverables:
✅ Eltern-Dashboard mit allen Widgets  
✅ Noten-Eingabe (Eltern + Schüler)  
✅ Wöchentlicher E-Mail-Report (APScheduler + Template)  
✅ Alert-System (Lernausfall, Kostenschwelle, Sicherheit)  
✅ Usage-Log + Kosten-Übersicht  
✅ Datenhaltungs-Cronjob (90-Tage-Bereinigung)  

**Validierung:** Report wird korrekt generiert und versendet, Alerts feuern bei Test-Szenarien

---

### Phase 4: Produktion & Qualität (Woche 4)

**Ziel:** Deployment auf Hetzner, stabile Produktion

Deliverables:
✅ Hetzner CX22 Setup (Docker, Nginx, Let's Encrypt)  
✅ Backup-System: pg_dump täglich + Hetzner-Snapshot wöchentlich  
✅ PWA-Konfiguration (installierbar auf Android)  
✅ Frontend-Polish (responsives Design, Mobile-optimiert)  
✅ Monitoring: einfaches Health-Check-Endpoint  
✅ Dokumentation: README, Deployment-Guide, Fach-Konfiguration  

**Validierung:** Test-Restore aus Backup erfolgreich, PWA auf Android installiert, 48h Betrieb ohne Probleme

---

## 14. Zukünftige Erweiterungen

### Phase 2+ Features

**Weitere Schulfächer:**
- Englisch (Klasse 6 NRW, Green Line 2)
- Mathematik + Wolfram Alpha Tool + Python-Calculator
- Geschichte, Erdkunde

**Erweiterter AI Harness:**
- Tool-Use pro Fach (Konfiguration bereits vorbereitet)
- LLM-Wechsel pro Fach (z.B. GPT-4o mini für Mathe)
- Automatische Schwierigkeitsgrad-Anpassung

**Erweiterte Gamification:**
- Punkte-System, Abzeichen, Level
- Wöchentliche Lernziele mit Belohnungen

**Allgemeiner Agent:**
- Off-Topic-Modus für allgemeine Fragen
- Klar getrennt vom Tutor-Modus

**Multi-User:**
- Mehrere Schüler-Accounts
- Geschwister-Profile
- Optional: andere Familien (dann DSGVO-Review nötig)

---

## 15. Risiken & Mitigationen

| Risiko | Wahrscheinlichkeit | Impact | Mitigation |
|--------|-------------------|--------|------------|
| Schüler umgeht Harness durch geschickte Prompts | Mittel | Hoch | Kontinuierliche Jailbreak-Tests; Output-Validator als zweite Linie; Eltern-Alert bei Auffälligkeiten |
| LLM-API-Kosten steigen unerwartet | Niedrig | Mittel | Tägliches Hard-Limit; Kosten-Alert bei 80%; Einfacher LLM-Wechsel per Adapter |
| Lehrbuch-OCR extrahiert falsche Vokabeln | Mittel | Mittel | Manuelle Review-UI nach jeder Extraktion; Plausibilitäts-Check (Zeichensatz, Pflichtfelder) |
| VPS-Ausfall verliert Lerndaten | Niedrig | Hoch | Täglicher DB-Dump lokal; wöchentlicher Hetzner-Snapshot; Test-Restore dokumentiert |
| Schüler verliert Motivation | Mittel | Hoch | Streak-System; ermutigende Persona; Eltern-Alert bei Lernausfall; Session-Zusammenfassungen mit positivem Feedback |

---

## 16. Appendix

### Abhängigkeiten & Links

- Anthropic API: https://docs.anthropic.com
- FastAPI: https://fastapi.tiangolo.com
- Hetzner Cloud: https://www.hetzner.com/cloud
- Let's Encrypt: https://letsencrypt.org
- Campus A Verlag: https://www.ccbuchner.de

### Kostenübersicht Betrieb (monatlich, geschätzt)

| Posten | Kosten |
|--------|--------|
| Hetzner CX22 VPS | ~4,50 € |
| Hetzner Snapshots (wöchentlich) | ~0,50 € |
| Claude Haiku 3.5 (normale Nutzung) | ~2–5 € |
| Domain + E-Mail (optional) | ~1–2 € |
| **Gesamt** | **~8–12 €/Monat** |

### Latein-Fehler-Taxonomie (Campus A, Klassen 5–6)

```
deklination_a          → a-Deklination (puella, -ae)
deklination_o          → o-Deklination (servus, -i / puer, -i)
deklination_konsonant  → Konsonantische Deklination (Phase 2)
kasus_nominativ        → Verwechslung Nominativ
kasus_akkusativ        → Verwechslung Akkusativ
kasus_genitiv          → Verwechslung Genitiv
kasus_dativ            → Verwechslung Dativ
kasus_ablativ          → Verwechslung Ablativ
konjugation_praesens   → Präsens o-Konjugation
konjugation_imperfekt  → Imperfekt
konjugation_perfekt    → Perfekt
esse_formen            → Formen von esse (sein)
vokabel                → Vokabel-Fehler (Lektion angegeben)
wortstellung           → Lateinische Wortstellung
```
