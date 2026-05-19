# Sadhana Companion

AI-powered spiritual habit and sadhana companion for daily logging, reflection, gentle reminders, mentor accountability, and long-term consistency.

## MVP Focus

The first version should prove the daily core loop:

```text
Log sadhana -> Reflect -> Receive insight -> Return tomorrow
```

## Initial Stack

- Mobile: Flutter, Riverpod, Firebase Auth, Firebase Messaging
- Backend: FastAPI, PostgreSQL, Redis
- AI: direct LLM integration first, LangGraph later
- Storage: offline-first mobile cache with background sync

## Start Here

- [MVP Plan](./docs/mvp-plan.md)
- [Backend Blueprint](./docs/backend-blueprint.md)
- [Mobile MVP Flow](./docs/mobile-mvp-flow.md)

## First Build Target

Create one end-to-end flow:

1. User signs in.
2. User logs today's sadhana.
3. User writes a reflection.
4. App shows today's progress and streak.
5. AI generates a short supportive reflection summary.

