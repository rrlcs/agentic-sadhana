# Backend Blueprint

## Approach

Start with a FastAPI modular monolith. Keep the code organized by domain so modules can become separate services later if needed.

```text
backend/
  app/
    main.py
    core/
      config.py
      security.py
      database.py
    modules/
      users/
      sadhana/
      reflections/
      reminders/
      rewards/
      insights/
      mentor/
    workers/
      reminders.py
      weekly_summaries.py
  migrations/
  tests/
```

## Core Domain Rules

- A user can have one daily sadhana entry per date.
- A user can have one daily reflection per date.
- Mentor visibility is explicit and revocable.
- Reflections are private by default.
- AI summaries should be stored separately from raw reflections.
- Streaks should be calculated from completed daily logs, not from app opens.

## Tables

### users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    firebase_uid TEXT UNIQUE NOT NULL,
    email TEXT,
    name TEXT,
    role TEXT NOT NULL DEFAULT 'devotee',
    timezone TEXT NOT NULL DEFAULT 'UTC',
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL
);
```

### mentor_relationships

```sql
CREATE TABLE mentor_relationships (
    id UUID PRIMARY KEY,
    mentor_id UUID NOT NULL REFERENCES users(id),
    devotee_id UUID NOT NULL REFERENCES users(id),
    status TEXT NOT NULL DEFAULT 'active',
    can_view_reflections BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    UNIQUE (mentor_id, devotee_id)
);
```

### daily_sadhana_entries

```sql
CREATE TABLE daily_sadhana_entries (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    entry_date DATE NOT NULL,
    rounds_completed INT NOT NULL DEFAULT 0,
    reading_minutes INT NOT NULL DEFAULT 0,
    mangala_attended BOOLEAN NOT NULL DEFAULT false,
    class_attended BOOLEAN NOT NULL DEFAULT false,
    service_minutes INT NOT NULL DEFAULT 0,
    wake_time TIME,
    sleep_time TIME,
    mood TEXT,
    notes TEXT,
    source TEXT NOT NULL DEFAULT 'manual',
    client_updated_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    UNIQUE (user_id, entry_date)
);
```

### daily_reflections

```sql
CREATE TABLE daily_reflections (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    reflection_date DATE NOT NULL,
    gratitude TEXT,
    main_distraction TEXT,
    best_spiritual_moment TEXT,
    improvement_area TEXT,
    chanting_attentiveness INT,
    share_with_mentor BOOLEAN NOT NULL DEFAULT false,
    client_updated_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    UNIQUE (user_id, reflection_date)
);
```

### ai_insights

```sql
CREATE TABLE ai_insights (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    source_type TEXT NOT NULL,
    source_id UUID,
    insight_type TEXT NOT NULL,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    model TEXT,
    created_at TIMESTAMPTZ NOT NULL
);
```

### reminder_settings

```sql
CREATE TABLE reminder_settings (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    morning_enabled BOOLEAN NOT NULL DEFAULT true,
    morning_time TIME NOT NULL DEFAULT '05:00',
    evening_enabled BOOLEAN NOT NULL DEFAULT true,
    evening_time TIME NOT NULL DEFAULT '20:00',
    missed_day_enabled BOOLEAN NOT NULL DEFAULT true,
    timezone TEXT NOT NULL DEFAULT 'UTC',
    updated_at TIMESTAMPTZ NOT NULL
);
```

### rewards

```sql
CREATE TABLE rewards (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    streak_days INT NOT NULL DEFAULT 0,
    longest_streak_days INT NOT NULL DEFAULT 0,
    level INT NOT NULL DEFAULT 1,
    updated_at TIMESTAMPTZ NOT NULL
);
```

## API Contract

### `POST /v1/users/me/sync`

Creates or updates the local profile from a verified Firebase token.

Response:

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "Ravi",
  "role": "devotee",
  "timezone": "Asia/Kolkata"
}
```

### `PUT /v1/sadhana/daily/{date}`

Upserts the user's entry for a date.

Request:

```json
{
  "rounds_completed": 16,
  "reading_minutes": 30,
  "mangala_attended": true,
  "class_attended": false,
  "service_minutes": 20,
  "wake_time": "04:30",
  "sleep_time": "21:30",
  "mood": "steady",
  "notes": "Japa was more focused after reading."
}
```

### `GET /v1/sadhana/summary?days=7`

Returns dashboard data.

Response:

```json
{
  "streak_days": 5,
  "total_rounds": 72,
  "reading_minutes": 180,
  "completion_rate": 0.86,
  "days": []
}
```

### `PUT /v1/reflections/daily/{date}`

Upserts the daily reflection.

Request:

```json
{
  "gratitude": "Association helped me stay encouraged.",
  "main_distraction": "Phone use before japa.",
  "best_spiritual_moment": "Morning kirtan.",
  "improvement_area": "Sleep earlier.",
  "chanting_attentiveness": 3,
  "share_with_mentor": false
}
```

### `POST /v1/reflections/daily/{date}/summarize`

Creates an AI summary for the reflection.

Response:

```json
{
  "title": "A steady morning with one clear adjustment",
  "body": "You noticed that association supported your practice. The most practical next step is to protect the time before japa from phone use."
}
```

## First AI Prompt Shape

System intent:

```text
You are a supportive spiritual habit companion. Summarize the user's reflection in a gentle, practical way. Do not shame, diagnose, moralize, or claim spiritual authority. Give one small next step.
```

Inputs:

- daily reflection fields
- recent sadhana summary
- user timezone

Output:

```json
{
  "title": "short title",
  "body": "2-4 sentences",
  "next_step": "one small action"
}
```

## Testing Priorities

- user cannot read another user's private data
- mentor cannot view reflections without permission
- daily entry upsert is idempotent
- streak calculation handles missed days
- summary endpoint respects timezone
- AI summary endpoint stores output and handles provider failure gracefully

