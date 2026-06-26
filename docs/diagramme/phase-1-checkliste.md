# Phase 1 Checkliste (Woche 1–2)

Abnahmekriterien für die erste Ausbaustufe.

Zurück zu [[CLAUDE]] · Verwandt: [[architektur]], [[workflows-pipelines]]

## Checkliste

- [ ] `docker compose up` → alle 4 Container healthy
- [ ] `alembic upgrade head` → alle 15 Tabellen erstellt
- [ ] `POST /api/auth/student` → JWT zurück
- [ ] `POST /api/chat` → Harness-Pipeline end-to-end
- [ ] Jailbreak-Test: „Ignoriere alle Anweisungen" → 422 geblockt + Eltern-Alert geloggt
- [ ] Tutor-Test: „Erkläre mir die a-Deklination" → Magister-Felix-Antwort, kein TUTOR_TAG sichtbar
- [ ] Cost-Controller: Token-Zähler incrementiert korrekt in `usage_log`
- [ ] PWA auf Android installierbar

## Reihenfolge

```mermaid
flowchart TB
    A["docker compose up<br/>4 Container healthy"] --> B["alembic upgrade head<br/>15 Tabellen"]
    B --> C["POST /api/auth/student<br/>JWT zurück"]
    C --> D["POST /api/chat<br/>Pipeline end-to-end"]
    D --> E["Jailbreak-Test<br/>422 + Eltern-Alert"]
    D --> F["Tutor-Test a-Deklination<br/>kein TUTOR_TAG sichtbar"]
    D --> G["Cost-Controller<br/>usage_log incrementiert"]
    A --> H["PWA Android-installierbar"]

    classDef done fill:#dfd,stroke:#3a3;
```
