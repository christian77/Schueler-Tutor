# Schueler-Tutor — Claude Code Context

## Project Mission

KI-gestützter Latein-Tutor für Leon, 12 Jahre, Klasse 6, NRW Gymnasium.
Lehrbuch: Campus A, 4. Auflage (C.C. Buchner). Tutor-Persona: **Magister Felix**.
Kernprinzip (v1.5, **Wissen direkt / Anwendung gestuft-sokratisch, Reveal als letztes Mittel**): Das System **vermittelt Wissen direkt** (Grammatikregeln, Vokabelbedeutungen, Konzepte klar und effizient erklärt). Die **Anwendung** (Hausaufgaben, Übersetzungen, Übungen) wird **sokratisch** geführt — und erst **nach nachgewiesener Anstrengung** (Default 3 echte Versuche, eltern-tunbar **pro Fach**; expliziter „Ich geb auf" nach ≥1 echtem Versuch verkürzt) über eine **4-Sprossen-Hint-Leiter** (Leitfrage → konkreter Hinweis+Paradigma → Teil-Gerüst → **erklärter Reveal + Folgeübung**) bis zur Lösung aufgelöst, damit ein ungeduldiger Schüler nicht abbricht. Der Output-Validator (LLM-als-Judge, **stateful**) unterscheidet **vier Kontexte** — **Wissen** (immer erklären), **Übung** (gestuft, Reveal als letztes Mittel), **Test aktiv** (keine Hilfe, kein Reveal), **Test-Auflösung** (volle Lösung + Erklärung nach Abgabe) — und schließt **fail-closed** für unautorisierte/verfrühte/Jailbreak-Reveals (sicherer sokratischer Fallback statt geflaggter Output). Im Zweifel gilt der strengere Kontext. „Echter Versuch", Aufgaben-Abgrenzung und Eskalations-Zustand laufen über den Analyzer-Call + ephemeres `sessions.current_task_state`. Der gesamte Harness läuft serverseitig und ist für den Schüler nicht umgehbar.

Vollständige Anforderungen: [PRD_Schueler_Tutor.md](PRD_Schueler_Tutor.md)

---

## Diagramme & Referenz

> Detail-Dokumentation liegt ausgelagert unter `docs/diagramme/` (Mermaid, Obsidian-lesbar). **Nur bei Bedarf lesen** — nicht standardmäßig in jeden Kontext laden.

| Thema | Datei | Inhalt |
|-------|-------|--------|
| Software-Architektur | [docs/diagramme/architektur.md](docs/diagramme/architektur.md) | Container-Topologie, Request-Fluss, Tech-Stack |
| App-Struktur & APIs | [docs/diagramme/app-struktur-apis.md](docs/diagramme/app-struktur-apis.md) | Frontend-Routen, alle API-Endpoints, JWT-Auth-Modell |
| Datenbankmodell | [docs/diagramme/datenbank-modell.md](docs/diagramme/datenbank-modell.md) | ER-Diagramm aller 15 Tabellen, Constraints, DSGVO |
| Workflows & Pipelines | [docs/diagramme/workflows-pipelines.md](docs/diagramme/workflows-pipelines.md) | 8-stufige Harness-Pipeline, TUTOR_TAG, Fehlertaxonomie, Adapter-/Subject-Pattern |
| Projektstruktur | [docs/diagramme/projektstruktur.md](docs/diagramme/projektstruktur.md) | Ziel-Verzeichnisbaum |
| Phase 1 Checkliste | [docs/diagramme/phase-1-checkliste.md](docs/diagramme/phase-1-checkliste.md) | Abnahmekriterien Woche 1–2 |

**Kurzüberblick Architektur:** Browser/PWA (React) → nginx → `/api/*` zu backend:8000 (FastAPI, AI Harness, LLM provider-agnostisch — Modellwahl pro Pipeline-Stufe, Start OpenAI `gpt5.4-mini`) + `/*` zu frontend:80; backend → PostgreSQL 16 (async SQLAlchemy + asyncpg). Der **AI Harness** (`backend/harness/`) ist die Kernkomponente: 8-stufige serverseitige Pipeline pro Chat-Nachricht — Details in [workflows-pipelines.md](docs/diagramme/workflows-pipelines.md).

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

## Konventionen

- **Sprache der Bezeichner**: Alle technischen Objekte sind **englisch** benannt — Code, Variablen, Funktionen, Skripte, Dateien/Ordner, API-Pfade & -Felder, DB-Tabellen/Spalten, Enum-/Key-Werte, Config-/YAML-Keys, Env-Vars. **Prosa/Inhalte** (PRD-Texte, Persona-Antworten, UI-Labels) dürfen deutsch oder zweisprachig sein. **Key/Label-Trennung**: fachliche Enum-Werte (Fehlertaxonomie, Modi, …) werden als **englische Keys** gespeichert/übertragen und erst in der **UI lokalisiert** angezeigt (Default-Anzeige: Deutsch).
- **Kein direkter Provider-SDK-Import** (`openai`, `anthropic`, …) außerhalb des jeweiligen Adapters (`adapters/openai_adapter.py`, `adapters/claude_adapter.py`). Modellwahl pro Pipeline-Stufe via Subject-YAML `models:`; strukturierte JSON-Ausgabe ist adapter-garantierter Vertrag
- **Router-Dateien** enthalten NUR Endpoint-Definitionen — Business-Logik in `harness/` oder `services/`
- **Async DB**: immer `async with AsyncSessionLocal() as db:` (nie sync Session)
- **Geldbeträge**: `DECIMAL(8,6)` EUR in DB
- **Fehler-Responses**: `{"detail": "..."}` (FastAPI-Standard)
- **System-Prompt** wird **NIEMALS** an Client übertragen oder geloggt
- **UUIDs** als Primary Keys (uuid4, server-generated)
- **Timestamps**: immer `TIMESTAMPTZ` (UTC), nie naive datetime
- **Mandanten-Scoping**: jede lernbezogene Query auf die `student_id` aus dem JWT-Claim filtern (nie ungefiltert über alle Schüler). `UNIQUE`-Constraints stets mit `student_id` denken.
- **Fach-/Sprach-Generik**: `vocab` ist Common-Core + `attribute` JSONB — fachspezifische Felder NICHT als Spalten ergänzen, sondern in `attribute`. Neues Fach = YAML + Paradigmen-Seed, kein Schema-Umbau.
- **Datenhaltung**: Chat-Volltext 90 Tage; dauerhafte Lernerkenntnisse nur aggregiert in `skill_mastery`/`student_profile` (kein Volltext).

**JWT Auth:** Zwei **separate** Secrets, niemals vermischen — Student (`STUDENT_JWT_SECRET`, 7 Tage) und Parent (`PARENT_JWT_SECRET`, 24h). Scopes & Dependency-Pattern: siehe [app-struktur-apis.md](docs/diagramme/app-struktur-apis.md).

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
