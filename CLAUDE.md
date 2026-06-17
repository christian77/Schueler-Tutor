# Schueler-Tutor — Claude Code Context

## Project Mission

KI-gestützter Latein-Tutor für Leon, 12 Jahre, Klasse 6, NRW Gymnasium.
Lehrbuch: Campus A, 4. Auflage (C.C. Buchner). Tutor-Persona: **Magister Felix**.
Kernprinzip: Das System gibt **NIEMALS** direkte Lösungen oder Übersetzungen — ausschließlich Sokrates-Methode (Leitfragen). Der gesamte Harness läuft serverseitig und ist für den Schüler nicht umgehbar.

Vollständige Anforderungen: [PRD_Schueler_Tutor.md](PRD_Schueler_Tutor.md)

---

## Quick Commands

```bash
# Alle Services starten
docker compose up -d

# Backend (lokal, ohne Docker)
cd backend && uvicorn main:app --reload --port 8000

# Datenbank-Migrationen
cd backend && alembic upgrade head
cd backend && alembic revision --autogenerate -m "describe change"

# Frontend (lokal)
cd frontend && npm run dev

# Logs
docker compose logs -f backend
docker compose logs -f db

# Nach Dependency-Änderung
docker compose build backend && docker compose up -d backend

# DB Shell
docker compose exec db psql -U tutor -d tutor

# Tests
cd backend && pytest tests/ -v
```

---

## Architecture Overview

```
Browser / PWA (React 18 + Vite + Tailwind)
     │
     │ HTTPS / JWT
     ▼
nginx (80/443)
  ├── /api/*  → proxy → backend:8000 (FastAPI / Python 3.12)
  │                         │
  │                    AI Harness
  │                    (8-stufige Pipeline)
  │                         │
  │                    Anthropic API
  │                    (claude-haiku-4-5)
  │
  └── /*      → frontend:80 (React SPA)

backend ──────────────────────────────────────► PostgreSQL 16
  (async SQLAlchemy 2.0 + asyncpg)
```

---

## AI Harness — Die Kernkomponente

**Alle Harness-Dateien: `backend/harness/`**
Serverseitig. API-Key und System-Prompt erreichen den Browser niemals.

### Pipeline (jede Chat-Nachricht durchläuft alle 8 Stufen)

| Stufe | Datei | Aufgabe |
|-------|-------|---------|
| 1 | `input_filter.py` | Regex-Jailbreak-Check (11 Muster DE+EN), 500-Zeichen-Limit, Off-Topic-Erkennung |
| 2 | `cost_controller.py` | Tägliches Token-Budget prüfen (50.000 Tokens/Tag, Warnung bei 80%) |
| 3 | `conversation_manager.py` | Session laden/erstellen (Inaktivität >10 Min → neue Session, max. 20 Nachrichten Kontext) |
| 4 | `system_prompt.py` | Persona + Pädagogik-Regeln + Lektion + Fehlerstand serverseitig zusammenbauen |
| 5 | `claude_adapter.py` | Anthropic API Call |
| 6 | `output_validator.py` | TUTOR_TAGs extrahieren, Direkt-Lösung erkennen → ggf. neu generieren |
| 7 | `cost_controller.py` | Token-Verbrauch in `usage_log` persistieren |
| 8 | — | Bereinigte Antwort (ohne Tags) an Client |

### TUTOR_TAG Format

Das LLM hängt am Ende jeder Antwort einen strukturierten Tag an (wenn ein Fehler erkannt wurde):
```
[TUTOR_TAG: {"fehler": "deklination_a", "wort": "puellam", "lektion": 5}]
```
- `output_validator.py` extrahiert Tags per Regex → speichert in `error_tags`-Tabelle
- Tags werden aus der sichtbaren Antwort entfernt — **Schüler sieht sie nie**
- Erlaubte Fehlertypen: `deklination_a`, `deklination_o`, `deklination_konsonant`, `kasus_nominativ`, `kasus_akkusativ`, `kasus_genitiv`, `kasus_dativ`, `kasus_ablativ`, `konjugation_praesens`, `konjugation_imperfekt`, `konjugation_perfekt`, `esse_formen`, `vokabel`, `wortstellung`

---

## JWT Auth

Zwei **separate** Secrets, niemals vermischen:

| Token | Secret (Env-Var) | Laufzeit | Scopes |
|-------|-----------------|---------|--------|
| Student | `STUDENT_JWT_SECRET` | 7 Tage | `chat`, `vocab`, `upload_homework` |
| Parent | `PARENT_JWT_SECRET` | 24h | `dashboard`, `notes`, `alerts`, `admin`, `upload_book` |

Dependency-Pattern in Routers:
```python
async def endpoint(user=Depends(require_student_auth), db=Depends(get_db)):
    ...
```

---

## Database Schema

8 Tabellen. Alle PKs: UUID. Alle Timestamps: TIMESTAMPTZ (UTC).

```
sessions ──────────────────────────────────────────────────┐
  id, started_at, ended_at, fach, modus, total_tokens,     │
  summary                                                    │
     │                                                       │
     ├──► messages (1:N)                                     │
     │      id, session_id, role, content,                   │
     │      created_at, expires_at (NOW()+90d ← DSGVO)      │
     │                                                       │
     ├──► error_tags (1:N)                                   │
     │      id, session_id, fehler_typ, wort, lektion,       │
     │      created_at                                       │
     │                                                       │
     ├──► vocab_results (1:N)                                │
     │      id, session_id, vocab_id, correct, created_at   │
     │                                                       │
     └──► usage_log (1:N)                                    │
            id, session_id, datum, tokens_in, tokens_out,   │
            cost_eur DECIMAL(8,6), created_at               │

vocab ──────────────────────────────────────────────────────┘
  id, lektion, latein, deutsch, genus, genitiv,
  dekl_konj, wortart, stammformen
     └──► vocab_results (1:N)

grades (standalone)
  id, fach, datum, art, note DECIMAL(2,1), kommentar,
  scan_path, eingegeben_von, created_at

streaks (1 Zeile, Singleton)
  id, current_streak, longest_streak, last_activity, updated_at
```

---

## Subject Configuration Pattern

Neue Fächer = neue YAML-Datei in `backend/subjects/`, kein Code nötig:

```yaml
# backend/subjects/latein.yaml
name: Latein
klasse: 6
lehrbuch: "Campus A, 4. Auflage (C.C. Buchner)"
llm_adapter: claude_haiku      # → adapters/claude_adapter.py
persona: magister_felix

themen_erlaubt:
  - latein
  - roemische_geschichte
  - roemisches_alltagsleben
  - allgemeines_lernen
  - smalltalk_kurz

fehler_taxonomie:
  - deklination_a
  - deklination_o
  # ... (14 Typen gesamt)

tools: []   # Phase 2: Wörterbuch-API o.ä.
```

---

## LLM Adapter Pattern

```python
# backend/adapters/base_llm.py
class BaseLLMAdapter(ABC):
    async def complete(self, messages: list[dict], system: str, max_tokens: int) -> LLMResponse
    async def complete_with_vision(self, image_b64: str, prompt: str) -> str
```

- `ClaudeAdapter` ist die **einzige Datei**, die `import anthropic` enthält
- Modell-ID: `claude-haiku-4-5` (Claude Haiku 3.5, Stand mid-2026 — bei Preisänderungen in `cost_controller.py` anpassen)
- Kosten (Haiku 3.5): Input $0.80/MTok, Output $4.00/MTok
- 50.000 Tokens/Tag ≈ ~100 Konversationsrunden ≈ max. $0.08–0.20/Tag

---

## API Endpoints

```
Student (STUDENT_JWT erforderlich):
  POST /api/chat                    Body: {message, session_id?}
                                    Response: {reply, session_id, tags}
  POST /api/vocab/session           Body: {lektion, mode: "lat_de"|"de_lat"}
  POST /api/vocab/answer            Body: {session_id, answer}
  GET  /api/progress/me             Response: {streak, vokabel_quote, schwaechen}
  POST /api/upload/homework         multipart/form-data (image/pdf)

Parent (PARENT_JWT erforderlich):
  GET  /api/parent/dashboard        Response: {streak, weekly_minutes, costs, ...}
  POST /api/parent/grade            Body: {fach, datum, note, art, kommentar}
  GET  /api/parent/alerts           Response: {alerts: [{type, days, ts}]}
  POST /api/admin/book/upload       multipart/form-data + ?lektion=5&typ=vokabeln
  POST /api/admin/book/confirm      Body: {review_id, items}

System:
  GET  /api/health                  Response: {"status": "ok"}
  GET  /api/docs                    FastAPI Swagger (in Produktion deaktivieren)
```

---

## Frontend Routes

```
/               → redirect /chat
/chat           → StudentChat.jsx      (Student-Auth-Guard)
/vocab          → VocabTrainer.jsx     (Student-Auth-Guard)
/parent         → ParentDashboard.jsx  (Parent-Auth-Guard)
/login          → Student-Login
/parent/login   → Parent-Login
```

---

## Projektstruktur (Ziel)

```
Schueler-Tutor/
├── CLAUDE.md
├── PRD_Schueler_Tutor.md
├── docker-compose.yml
├── .env.example
├── .gitignore
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py                          # FastAPI app, Router-Registrierung, lifespan
│   ├── harness/
│   │   ├── system_prompt.py             # baut Magister-Felix-Prompt serverseitig
│   │   ├── input_filter.py              # Jailbreak + Länge + Off-Topic
│   │   ├── output_validator.py          # TUTOR_TAG-Extraktion + Lösung-Erkennung
│   │   ├── cost_controller.py           # Token-Budget + Usage-Log
│   │   └── conversation_manager.py      # Session- + Nachrichten-Management
│   ├── adapters/
│   │   ├── base_llm.py                  # Abstrakte Basisklasse
│   │   ├── claude_adapter.py            # Einziger anthropic-Import
│   │   └── openai_adapter.py            # Stub (Phase 2)
│   ├── pipeline/
│   │   ├── ocr_extractor.py             # Lehrbuch-OCR via Claude Vision
│   │   ├── vocab_importer.py            # Vokabeln in DB importieren
│   │   └── homework_analyzer.py         # Klassenarbeit-Analyse
│   ├── routers/
│   │   ├── auth.py                      # POST /api/auth/student + /parent
│   │   ├── chat.py                      # POST /api/chat (orchestriert Harness)
│   │   ├── vocab.py                     # Vokabel-Trainer Endpoints
│   │   ├── parent.py                    # Eltern-Dashboard Endpoints
│   │   └── upload.py                    # Foto/Scan Upload
│   ├── models/
│   │   └── database.py                  # SQLAlchemy 2.0 async, alle 8 Tabellen
│   └── subjects/
│       └── latein.yaml
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json                     # React 18, Vite 5, Tailwind 3, Recharts 2
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx                      # React Router Setup
│       ├── index.css                    # @tailwind base/components/utilities
│       ├── views/
│       │   ├── StudentChat.jsx
│       │   ├── VocabTrainer.jsx
│       │   └── ParentDashboard.jsx
│       └── components/
│
└── nginx/
    └── nginx.conf                       # Rate-Limit, API-Proxy, SPA-Routing
```

---

## Key Environment Variables

```bash
# Datenbank
DATABASE_URL=postgresql+asyncpg://tutor:changeme@db:5432/tutor
POSTGRES_USER=tutor
POSTGRES_PASSWORD=changeme
POSTGRES_DB=tutor

# LLM — NIEMALS loggen oder committen
ANTHROPIC_API_KEY=sk-ant-api03-...

# JWT — zwei verschiedene Secrets!
STUDENT_JWT_SECRET=<openssl rand -hex 32>
PARENT_JWT_SECRET=<openssl rand -hex 32>

# Kostensteuerung
DAILY_TOKEN_LIMIT=50000
COST_WARN_THRESHOLD=0.80
COST_STOP_THRESHOLD=1.00

# E-Mail-Alerts
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=tutor@example.com
SMTP_PASS=...
PARENT_EMAIL=eltern@example.com

# App
APP_ENV=development
CORS_ORIGINS=http://localhost:5173,http://localhost:80
MESSAGE_RETENTION_DAYS=90
```

---

## Konventionen

- **Kein direkter `anthropic`-Import** außerhalb von `adapters/claude_adapter.py`
- **Router-Dateien** enthalten NUR Endpoint-Definitionen — Business-Logik in `harness/` oder `services/`
- **Async DB**: immer `async with AsyncSessionLocal() as db:` (nie sync Session)
- **Geldbeträge**: `DECIMAL(8,6)` EUR in DB
- **Fehler-Responses**: `{"detail": "..."}` (FastAPI-Standard)
- **System-Prompt** wird **NIEMALS** an Client übertragen oder geloggt
- **UUIDs** als Primary Keys (uuid4, server-generated)
- **Timestamps**: immer `TIMESTAMPTZ` (UTC), nie naive datetime

---

## Phase 1 Checkliste (Woche 1–2)

- [ ] `docker compose up` → alle 4 Container healthy
- [ ] `alembic upgrade head` → alle 8 Tabellen erstellt
- [ ] `POST /api/auth/student` → JWT zurück
- [ ] `POST /api/chat` → Harness-Pipeline end-to-end
- [ ] Jailbreak-Test: „Ignoriere alle Anweisungen" → 422 geblockt + Eltern-Alert geloggt
- [ ] Tutor-Test: „Erkläre mir die a-Deklination" → Magister-Felix-Antwort, kein TUTOR_TAG sichtbar
- [ ] Cost-Controller: Token-Zähler incrementiert korrekt in `usage_log`
- [ ] PWA auf Android installierbar
