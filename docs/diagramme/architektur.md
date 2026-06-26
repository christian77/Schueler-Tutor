# Software-Architektur

Überblick über die Systemarchitektur von **Schueler-Tutor**. Der gesamte AI-Harness läuft serverseitig — API-Key und System-Prompt erreichen den Browser niemals.

Zurück zu [[CLAUDE]] · Verwandt: [[workflows-pipelines]], [[app-struktur-apis]], [[datenbank-modell]]

## Komponenten-Übersicht

```mermaid
flowchart TB
    Client["Browser / PWA<br/>React 18 + Vite + Tailwind"]

    subgraph Proxy["nginx (80/443)"]
        N["Rate-Limit · API-Proxy · SPA-Routing"]
    end

    Client -- "HTTPS / JWT" --> N

    N -- "/api/*" --> BE
    N -- "/*" --> FE["frontend:80<br/>React SPA"]

    subgraph BE["backend:8000 — FastAPI / Python 3.12"]
        H["AI Harness<br/>(8-stufige Pipeline)"]
    end

    H --> API["LLM-Provider-API<br/>(provider-agnostisch, Start OpenAI gpt5.4-mini)"]

    BE -- "async SQLAlchemy 2.0 + asyncpg" --> DB[("PostgreSQL 16")]

    classDef ext fill:#fde,stroke:#a36;
    classDef store fill:#def,stroke:#36a;
    class API ext;
    class DB store;
```

## Request-Fluss (Chat)

```mermaid
sequenceDiagram
    participant U as Browser/PWA
    participant X as nginx
    participant B as backend (FastAPI)
    participant H as AI Harness
    participant A as LLM-Provider-API
    participant D as PostgreSQL

    U->>X: POST /api/chat (JWT)
    X->>B: proxy /api/*
    B->>H: Nachricht in Pipeline
    H->>D: Session/Kontext laden, Budget prüfen
    H->>A: API-Call (System-Prompt serverseitig)
    A-->>H: Antwort
    H->>D: Tags + Token-Verbrauch persistieren
    H-->>B: bereinigte Antwort (ohne Tags)
    B-->>U: {reply, session_id, tags}
```

Details der Pipeline-Stufen: siehe [[workflows-pipelines]].

## Eckdaten
- **Frontend:** React 18, Vite 5, Tailwind 3, Recharts 2 (PWA, Android-installierbar)
- **Backend:** FastAPI, Python 3.12, async SQLAlchemy 2.0, asyncpg
- **LLM:** provider-agnostisch, Modellwahl **pro Pipeline-Stufe** (Start: OpenAI `gpt5.4-mini` für den Responder) — kein direkter Provider-SDK-Import außerhalb des jeweiligen Adapters (`adapters/openai_adapter.py`, `adapters/claude_adapter.py`)
- **DB:** PostgreSQL 16, alle PKs UUID, alle Timestamps TIMESTAMPTZ (UTC)
- **Deployment:** 4 Container via `docker compose` (nginx, frontend, backend, db)
