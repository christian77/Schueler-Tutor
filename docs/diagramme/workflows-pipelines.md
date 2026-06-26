# Workflows & Pipelines

Die **Kernkomponente** des Systems: der serverseitige AI-Harness. Alle Harness-Dateien liegen unter `backend/harness/`. API-Key und System-Prompt erreichen den Browser niemals.

Zurück zu [[CLAUDE]] · Verwandt: [[architektur]], [[datenbank-modell]]

> **v1.5:** Der Harness folgt dem **Vier-Kontext-Modell** (Wissen / Übung / Test aktiv / Test-Auflösung) mit **gestufter Eskalation und Reveal als letztes Mittel**. Der Output-Validator ist ein **stateful LLM-als-Judge** (separater Call), und die Modellwahl ist **provider-agnostisch pro Pipeline-Stufe** (Start: OpenAI `gpt5.4-mini`).

## AI-Harness-Pipeline

Jede Chat-Nachricht durchläuft die Pipeline. Stufen 4b/5/6/6b sind **separate, spezialisierte LLM-Calls** (kein autonomer Agent — deterministischer Workflow).

```mermaid
flowchart TB
    IN["Chat-Nachricht (Schüler)"]
    S1["1 · input_filter.py<br/>Regex-Jailbreak (DE+EN)<br/>500-Zeichen-Limit · Off-Topic"]
    S2["2 · cost_controller.py<br/>Tages-Token-Budget (50.000/Tag)<br/>Warnung bei 80 %"]
    S3["3 · conversation_manager.py<br/>Session + current_task_state laden<br/>max. 20 Nachrichten Kontext"]
    S4["4 · system_prompt.py<br/>Persona + Pädagogik + Lektion<br/>+ skill_mastery + Eskalations-Zustand"]
    C["4b · Classifier-Call<br/>{context: Wissen|Übung, confidence}<br/>+ mode/test_state · im Zweifel strenger"]
    S5["5 · Responder-Call (adapter)<br/>Magister-Felix-Prosa (NUR Prosa)"]
    J["6 · Judge-Call (output_validator)<br/>stateful · 4 Kontexte<br/>{reveals_solution, reason}"]
    AN["6b · Analyzer-Call<br/>error_tags · task_fingerprint · genuine_attempt"]
    S7["7 · cost_controller.py<br/>Token-Verbrauch (alle Calls) → usage_log"]
    S8["8 · Bereinigte Antwort an Client"]

    IN --> S1
    S1 -->|blockiert| BLK["422 + Eltern-Alert"]
    S1 -->|ok| S2
    S2 -->|Budget überschritten| STOP["Stop / Hinweis"]
    S2 -->|ok| S3 --> S4 --> C --> S5 --> J
    J -->|unautorisierter / verfrühter Reveal| REGEN["neu generieren (strenger)<br/>nach M Versuchen ODER Judge unavailable<br/>→ fail-closed: sokratischer Fallback"]
    REGEN --> S5
    J -->|ok / legitime Eskalation| AN --> S7 --> S8
```

## Kernprinzip: Vier Kontexte (v1.5)

Der Judge (Stufe 6) ist **stateful** und kennt den Eskalations-Zustand der aktuellen Aufgabe (aus `sessions.current_task_state`). **Im Zweifel gilt der strengere Kontext** (Übung vor Wissen, Test-aktiv vor Auflösung).

```mermaid
flowchart TD
    Q["Schüler-Anfrage"] --> K{Kontext?}
    K -->|Wissen| W["Regeln, Vokabeln, Konzepte<br/>→ DIREKT & klar erklären"]
    K -->|Übung| U{≥ N echte Versuche<br/>pro Aufgabe?}
    K -->|Test aktiv| T["keine Hilfe, kein Reveal<br/>(reine Messung)"]
    K -->|Test-Auflösung| AU["volle Lösung + Erklärung<br/>(nachfragbar = Wissen)"]
    U -->|nein| SK["sokratisch: Leitfrage →<br/>konkreter Hinweis+Paradigma →<br/>Teil-Gerüst<br/>(Reveal geblockt, fail-closed)"]
    U -->|ja / 'Ich geb auf' nach ≥1 Versuch| REV["erklärter Reveal<br/>(Lösungsweg + warum)<br/>+ Folgeübung → skill_mastery"]
    W --> OK["Antwort freigeben"]
    SK --> OK
    REV --> OK
    AU --> OK
    T --> OK
```

**Trigger / Anti-Gaming:** Ein „echter Versuch" ist ein tatsächlicher Lösungsansatz — bloßes „sag die Lösung" / „ich geb auf" ohne Ansatz zählt **nicht** (beurteilt der Analyzer-Call). Schwelle `N` = Default 3, eltern-tunbar **pro Fach** (`reveal.patience_threshold`); expliziter „Ich geb auf"-Pfad nach ≥1 echtem Versuch verkürzt. Versuchszählung **pro Aufgabe** über den Aufgaben-Fingerprint; Aufgabenwechsel → Reset.

## TUTOR_TAG-Format (via Analyzer-Call)

Pädagogik-Signale liefert ein **separater Analyzer-Call als strukturierter Output** — **nicht** der Responder im Fließtext (keine Tag-Leckage in der Schülerantwort, keine Regex-Fragilität, provider-agnostisch). Der Responder gibt **nur Prosa** zurück.

```json
{ "error": "declension_a", "word": "puellam", "lesson": 5,
  "task_fingerprint": "cornelia-puellam-vocat", "same_task_as_previous": true,
  "genuine_attempt": true }
```

- `error_tags` → gespeichert in `error_tags`-Tabelle + Aggregat `skill_mastery`.
- `task_fingerprint` / `same_task_as_previous` / `genuine_attempt` → pflegen `sessions.current_task_state` (Versuchszähler, Eskalations-Zustand).
- Der Schüler sieht nichts davon.

### Fehlertaxonomie (14 Typen)

```mermaid
flowchart LR
    subgraph Deklination
        d1[declension_a]
        d2[declension_o]
        d3[declension_consonant]
    end
    subgraph Kasus
        k1[case_nominative]
        k2[case_accusative]
        k3[case_genitive]
        k4[case_dative]
        k5[case_ablative]
    end
    subgraph Konjugation
        j1[conjugation_present]
        j2[conjugation_imperfect]
        j3[conjugation_perfect]
        j4[esse_forms]
    end
    subgraph Sonstige
        s1[vocab]
        s2[word_order]
    end
```

## LLM-Adapter-Pattern (provider-agnostisch)

**Regel:** Kein direkter Provider-SDK-Import (`openai`, `anthropic`, …) außerhalb des **jeweiligen Adapters**. `BaseLLMAdapter` definiert ein schmales Interface; strukturierte JSON-Ausgabe ist ein **adapter-garantierter Vertrag** (nativ wo unterstützt, sonst Prompt + Parse-Validate-Retry im Adapter) — die Pipeline-Stufen sehen den Unterschied nie.

```mermaid
classDiagram
    class BaseLLMAdapter {
        <<abstract>>
        +complete(messages, system, schema?) LLMResponse
        +complete_with_vision(image_b64, prompt) str
        +supports_structured_output bool
        +supports_prompt_cache bool
    }
    class OpenAIAdapter {
        +einziger openai import
        +start_modell gpt5.4-mini
    }
    class ClaudeAdapter {
        +einziger anthropic import
    }
    BaseLLMAdapter <|-- OpenAIAdapter
    BaseLLMAdapter <|-- ClaudeAdapter
```

- **Modellwahl pro Pipeline-Stufe** (Subject-YAML `models:`), nicht global. Start: OpenAI `gpt5.4-mini` für den **Responder**; günstigere Modelle für Classifier/Judge/Analyzer.
- **Fähigkeiten von `gpt5.4-mini`** (Structured Output, Limits, Caching) sind **zur Build-Zeit im Adapter zu verifizieren** — die Capability-Flags steuern nur den günstigeren Pfad, nie die Korrektheit.
- **Preise je aktivem Anbieter** in `cost_controller.py` pflegen; 50.000 Tokens/Tag ≈ ~100 Konversationsrunden (jetzt verteilt über Responder + Classifier + Judge + Analyzer pro Nachricht).
- **Adapter-Fallback-Kette** (Provider A nicht erreichbar → Provider B), da Multi-Provider gefordert ist.

## Subject-Configuration-Pattern

Neues Fach = neue YAML-Datei in `backend/subjects/`, **kein Code nötig**. Reveal-Politik und Modellwahl sind Teil der Fach-Config.

```mermaid
flowchart LR
    Y["backend/subjects/latin.yaml<br/>name · grade_level · textbook · persona<br/>models (pro Stufe) · reveal<br/>allowed_topics · error_taxonomy · tools"]
    Y --> H["Harness liest YAML"]
    H --> SP["system_prompt.py"]
    H --> AD["adapters/* (models pro Stufe)"]
    H --> RV["Judge/Responder (reveal-Politik)"]
```

Beispiel:
```yaml
# backend/subjects/latin.yaml
name: Latein
grade_level: 6
textbook: "Campus A, 4. Auflage (C.C. Buchner)"
persona: magister_felix

# Modellwahl pro Pipeline-Stufe (provider-agnostisch; Adapter kapselt den Anbieter)
models:
  responder:  { provider: openai, model: "gpt5.4-mini" }       # qualitäts-/sicherheitskritisch (Start)
  classifier: { provider: openai, model: "<günstiges Modell>" } # Wissen vs. Übung
  judge:      { provider: openai, model: "<günstiges Modell>" } # Output-Validator (≠ Eval-Grader-Modell)
  analyzer:   { provider: openai, model: "<günstiges Modell>" } # Tag-/Fehler-Extraktion
fallback_provider: null   # optionaler zweiter Anbieter für die Adapter-Fallback-Kette

# Reveal-Politik (v1.5) — Fach-Default, von Eltern pro Fach überschreibbar
reveal:
  enabled: true            # Reveal als letztes Mittel im Übungsmodus erlauben
  patience_threshold: 3    # echte Versuche bis Eskalation freigeschaltet (2–4)

allowed_topics:
  - latin
  - roman_history
  - roman_daily_life
  - general_learning
  - smalltalk_brief

error_taxonomy:
  - declension_a
  - declension_o
  # ... (14 Typen gesamt)

tools: []   # Phase 2: Wörterbuch-API o.ä.
```

### Erlaubte Themen (Latein)
`latin` · `roman_history` · `roman_daily_life` · `general_learning` · `smalltalk_brief`
