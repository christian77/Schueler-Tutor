# Datenbankmodell

15 Kern-Tabellen (+ `chat_attachments` in Phase 2.1). Alle PKs: UUID. Alle Timestamps: TIMESTAMPTZ (UTC).

Zurück zu [[CLAUDE]] · Verwandt: [[architektur]], [[app-struktur-apis]]

> **v1.3-Naht:** Betrieb mit GENAU EINEM Schüler, aber alle lernbezogenen Tabellen tragen bereits `student_id`, und `UNIQUE`-Constraints sind auf `(student_id, …)` ausgelegt (Multi-User später ohne Tabellen-Umbau). Event-Logs (`messages`, `error_tags`, `vocab_results`, `usage_log`) erben `student_id` transitiv über `session_id`.

## ER-Diagramm

```mermaid
erDiagram
    students ||--o{ student_subjects : hat
    students ||--|| student_profile : hat
    students ||--o{ sessions : fuehrt
    students ||--o{ grades : hat
    students ||--|| streaks : hat
    students ||--o{ vocab_srs : lernt
    students ||--o{ skill_mastery : hat
    students ||--o{ upcoming_tests : plant

    sessions ||--o{ messages : enthaelt
    sessions ||--o{ error_tags : protokolliert
    sessions ||--o{ vocab_results : protokolliert
    sessions ||--o{ usage_log : protokolliert

    vocab ||--o{ vocab_results : referenziert
    vocab ||--o{ vocab_srs : referenziert

    students {
        uuid id PK
        string username UK
        string password_hash
        string display_name
        int grade_level
        timestamptz created_at
    }
    student_subjects {
        uuid id PK
        uuid student_id FK
        string subject
        int current_lesson
        bool active
    }
    student_profile {
        uuid id PK
        uuid student_id UK
        jsonb strengths
        jsonb weaknesses
        jsonb strategies
        text notes
        timestamptz updated_at
    }
    sessions {
        uuid id PK
        uuid student_id FK
        timestamptz started_at
        timestamptz ended_at
        string subject
        string mode
        string test_state
        jsonb current_task_state
        int total_tokens
        text summary
    }
    messages {
        uuid id PK
        uuid session_id FK
        string role
        text content
        timestamptz created_at
        timestamptz expires_at
    }
    error_tags {
        uuid id PK
        uuid session_id FK
        string error_type
        string word
        int lesson
        timestamptz created_at
    }
    vocab_results {
        uuid id PK
        uuid session_id FK
        uuid vocab_id FK
        bool correct
        int quality
        string direction
        timestamptz created_at
    }
    usage_log {
        uuid id PK
        uuid session_id FK
        date date
        int tokens_in
        int tokens_out
        decimal cost_eur
        timestamptz created_at
    }
    vocab {
        uuid id PK
        string subject
        int lesson
        string source_language
        string target_language
        string term
        string meaning
        string part_of_speech
        jsonb attribute
    }
    vocab_srs {
        uuid id PK
        uuid student_id FK
        uuid vocab_id FK
        string direction
        float ease_factor
        int interval_days
        int repetitions
        int lapses
        date due_date
        timestamptz last_reviewed
    }
    grades {
        uuid id PK
        uuid student_id FK
        string subject
        date date
        string type
        decimal grade
        text comment
        string scan_path
        string entered_by
        timestamptz created_at
    }
    streaks {
        uuid id PK
        uuid student_id UK
        int current_streak
        int longest_streak
        date last_activity
        timestamptz updated_at
    }
    grammar_paradigms {
        uuid id PK
        string subject
        string type
        string name
        int lesson
        jsonb forms
    }
    skill_mastery {
        uuid id PK
        uuid student_id FK
        string subject
        string skill_type
        string skill_key
        int attempts
        int correct
        float mastery
        timestamptz last_seen
    }
    upcoming_tests {
        uuid id PK
        uuid student_id FK
        string subject
        date date
        text topics
        text lessons
        timestamptz created_at
    }
```

## Tabellen-Notizen

- **`vocab`** ist fach-/sprachgenerisch: Common-Core-Spalten + `attribute` JSONB.
  - Latein: `{gender, genitive, declension_conjugation, principal_parts}`
  - Englisch: `{irregular_plural, ...}`
  - Fachspezifische Felder **nicht** als Spalten ergänzen, sondern in `attribute`.
- **`vocab_srs`** — SM-2 je `(student, vocab, direction)`, `UNIQUE (student_id, vocab_id, direction)`.
- **`student_profile`** — dauerhaftes Langzeit-Gedächtnis (kein Chat-Volltext).
- **`skill_mastery`** — rollendes Lernstandsmodell, speist Personalisierung + Dashboard. `UNIQUE (student_id, subject, skill_type, skill_key)`.
- **`grammar_paradigms`** — standalone Referenz-Tabellen je Fach, verhindert LLM-Halluzination.
- **`upcoming_tests`** — speist Klassenarbeits-Vorbereitung.
- **UNIQUE-Constraints:** `student_subjects (student_id, subject)`; `student_profile (student_id)`; `streaks (student_id)`.

### Datenhaltung (DSGVO)
- **`messages`**: `expires_at = NOW() + 90d` — Chat-Volltext 90 Tage.
- Dauerhafte Lernerkenntnisse nur aggregiert in `skill_mastery` / `student_profile` (kein Volltext).
- **Geldbeträge**: `DECIMAL(8,6)` EUR (`usage_log.cost_eur`).
