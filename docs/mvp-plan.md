# Sadhana Companion MVP Plan

## MVP North Star

Build the smallest lovable version of Sadhana Companion that helps a devotee log daily sadhana, reflect honestly, receive gentle AI-assisted guidance, and maintain consistency over time.

The MVP should prove one core loop:

```text
User logs daily sadhana -> User reflects -> System summarizes progress -> System nudges at the right time -> User returns tomorrow
```

## MVP User Roles

Start with two roles only:

- `devotee`: logs sadhana, journals, receives reminders and insights.
- `mentor`: views opt-in devotee summaries and basic consistency trends.

Defer `sangha_admin` and `super_admin` until the mentor workflow is validated.

## MVP Feature Scope

### 1. Authentication

Use Firebase Authentication for the first version.

MVP support:

- Google Sign-In
- Email login
- Firebase user ID mapped to backend profile

Defer:

- Phone OTP
- Apple Sign-In
- complex admin role management

### 2. Daily Sadhana Logging

The first release should support one daily entry per user per date.

Fields:

- rounds completed
- reading minutes
- mangala arati attended
- class attended
- service minutes
- wake time
- sleep time
- mood
- notes

Input modes:

- manual entry
- quick actions

Defer voice input and AI chat logging until the core model is stable.

### 3. Reflection Journal

Daily fields:

- gratitude
- main distraction
- best spiritual moment
- improvement area
- chanting attentiveness score, 1-5

AI in MVP:

- summarize the reflection
- generate one gentle next-step suggestion
- create a weekly reflection summary

### 4. Dashboard

Devotee dashboard:

- today completion status
- rounds progress
- current streak
- 7-day consistency view
- latest AI insight

Mentor dashboard:

- devotee list
- 7-day consistency
- missed-day alerts
- opt-in weekly summaries

### 5. Reminders

Start with rule-based reminders before adaptive AI reminders.

MVP reminder types:

- morning reminder
- evening logging reminder
- missed-day reminder

Implementation:

- Firebase Cloud Messaging
- backend scheduled job
- per-user reminder settings

AI adaptation can be added after enough behavior data exists.

### 6. Rewards

Keep rewards spiritually aligned and calm.

MVP:

- streak days
- weekly consistency badge
- simple level based on consistency, not endless XP grinding

Defer:

- spiritual garden
- temple restoration theme
- advanced badge system

## Recommended MVP Architecture

Use a modular monolith backend first, with clear service boundaries. This keeps development fast while preserving the option to split services later.

```text
Flutter App
  |
FastAPI Backend
  |-- auth module
  |-- users module
  |-- sadhana module
  |-- reflections module
  |-- reminders module
  |-- insights module
  |-- mentor module
  |
PostgreSQL
Redis
Firebase Auth + FCM
OpenAI or other LLM provider
```

Do not start with Kubernetes, microservices, Pinecone, or multiple AI agents. Introduce those when user volume and product learning justify them.

## Initial Tech Stack

### Mobile

- Flutter
- Riverpod
- GoRouter
- Drift or Hive for local persistence
- Firebase Auth
- Firebase Messaging

### Backend

- FastAPI
- PostgreSQL
- SQLAlchemy 2.x
- Alembic
- Pydantic
- Redis for background jobs and reminder queue
- Celery, RQ, or Arq for jobs

### AI

- Start with direct LLM service wrapper
- Add LangGraph only after the first three AI workflows are clear

Initial AI workflows:

- daily reflection summary
- weekly summary
- reminder message generation

## First Database Model

Core tables:

- `users`
- `mentor_relationships`
- `daily_sadhana_entries`
- `daily_reflections`
- `reminder_settings`
- `rewards`
- `ai_insights`

Important constraints:

- one sadhana entry per user per date
- one reflection per user per date
- reflection sharing is opt-in
- mentor access must be permissioned per devotee

## MVP API Surface

### Auth and User

- `POST /v1/users/me/sync`
- `GET /v1/users/me`
- `PATCH /v1/users/me`

### Sadhana

- `PUT /v1/sadhana/daily/{date}`
- `GET /v1/sadhana/daily/{date}`
- `GET /v1/sadhana/summary?range=7d`

### Reflections

- `PUT /v1/reflections/daily/{date}`
- `GET /v1/reflections/daily/{date}`
- `POST /v1/reflections/daily/{date}/summarize`

### Reminders

- `GET /v1/reminders/settings`
- `PATCH /v1/reminders/settings`
- `POST /v1/devices/register`

### Mentor

- `GET /v1/mentor/devotees`
- `GET /v1/mentor/devotees/{user_id}/summary`
- `GET /v1/mentor/devotees/{user_id}/weekly-report`

## Offline-First MVP

Mobile should allow logging even when offline.

Local behavior:

- store daily logs locally
- store reflections locally
- mark records as `pending_sync`
- show local dashboard from cached data

Sync behavior:

- background sync when online
- server accepts client-generated IDs
- conflict resolution uses `updated_at`

For MVP, latest update wins is acceptable. Add conflict UI later if users edit the same entry across multiple devices.

## AI Safety and Tone

The AI companion should not act as a guru, therapist, or moral authority.

It should:

- encourage consistency
- ask reflective questions
- summarize user-provided data
- suggest small practical next steps
- recommend contacting a mentor when distress or burnout appears

It should avoid:

- shame
- comparison
- spiritual superiority language
- diagnosing mental health conditions
- claiming divine authority

## Build Order

### Milestone 1: Repo Foundation

- create backend FastAPI app
- create Flutter app
- configure formatting and linting
- add Docker Compose for PostgreSQL and Redis
- add local environment templates

### Milestone 2: Backend Core

- user profile sync from Firebase token
- daily sadhana CRUD
- reflection CRUD
- streak calculation
- 7-day summary endpoint

### Milestone 3: Mobile Core Loop

- auth screen
- today dashboard
- daily log screen
- reflection screen
- local storage and sync queue

### Milestone 4: AI Reflection

- reflection summary endpoint
- weekly summary endpoint
- store AI insights
- display latest insight on dashboard

### Milestone 5: Reminders

- reminder settings screen
- FCM device registration
- scheduled reminder worker
- missed-day detection

### Milestone 6: Mentor Preview

- mentor relationship model
- devotee summary endpoint
- mentor dashboard screen
- reflection sharing permission

## Suggested 6-Week MVP Schedule

### Week 1

- finalize UX flows
- scaffold repo
- backend app setup
- database migrations
- Flutter navigation and theme

### Week 2

- Firebase auth integration
- user profile sync
- daily sadhana backend and mobile forms

### Week 3

- offline local storage
- sync queue
- streak and 7-day dashboard

### Week 4

- reflection journal
- AI summary service
- dashboard insight card

### Week 5

- reminder settings
- Firebase Cloud Messaging
- backend scheduled reminders

### Week 6

- mentor summary
- privacy controls
- polish, QA, beta release

## What To Defer

Defer these until the core loop has real users:

- microservices
- Kubernetes
- Pinecone or vector memory
- multi-agent orchestration
- voice assistant
- WhatsApp and Telegram
- spiritual garden
- advanced analytics
- predictive burnout scoring
- payment gateway

## Immediate Next Step

Start by creating:

1. FastAPI backend scaffold.
2. Flutter mobile scaffold.
3. Docker Compose with PostgreSQL and Redis.
4. Database migrations for users, daily sadhana, reflections, reminders, rewards, and insights.
5. One end-to-end flow: sign in, log today, reflect, view dashboard.

