# Mobile MVP Flow

## Principle

The first app experience should make daily practice easier to record in under one minute. Reflection and insights should feel supportive, not like another productivity system.

## Primary Navigation

Use four tabs:

- Today
- Journal
- Progress
- Profile

Mentor features can be hidden behind role-based routing after login.

## Screens

### 1. Sign In

Purpose:

- authenticate with Firebase
- sync backend profile

MVP options:

- Google Sign-In
- email login

### 2. Today

Purpose:

- show the current day at a glance
- allow fast logging

Content:

- rounds progress
- reading minutes
- mangala arati toggle
- class attended toggle
- service minutes
- mood selector
- streak count
- latest insight

Primary actions:

- edit today log
- open reflection

### 3. Daily Log

Purpose:

- capture the user's sadhana data

Inputs:

- rounds completed
- reading minutes
- mangala arati attended
- class attended
- service minutes
- wake time
- sleep time
- mood
- notes

UX:

- use steppers for numeric fields
- use toggles for attendance
- save automatically when possible
- show offline sync status quietly

### 4. Journal

Purpose:

- capture daily reflection

Inputs:

- gratitude
- main distraction
- best spiritual moment
- improvement area
- chanting attentiveness
- share with mentor toggle

Primary action:

- generate reflection summary

### 5. Progress

Purpose:

- show consistency without encouraging unhealthy comparison

Content:

- 7-day completion strip
- current streak
- longest streak
- rounds trend
- reading trend
- reflection completion

### 6. Profile

Purpose:

- settings and account management

Content:

- name
- timezone
- reminder settings
- privacy settings
- mentor sharing permissions
- sign out

### 7. Mentor Dashboard

Purpose:

- help mentors support devotees without exposing private details by default

Content:

- devotee list
- 7-day consistency
- missed-day alerts
- shared weekly summaries

## Offline Behavior

When offline:

- allow logging
- allow journaling
- calculate local streak from cached entries
- show pending sync indicator

When online:

- sync pending records
- refresh dashboard
- generate AI summaries

## First End-To-End Acceptance Test

```text
Given a new devotee signs in
When they log 8 rounds and 20 reading minutes for today
And they complete a reflection
Then the Today screen shows today's progress
And the Progress screen shows the day as completed
And the backend has one sadhana entry and one reflection for the date
```

## Design Direction

Visual tone:

- calm
- clear
- warm
- low-distraction
- devotional without visual clutter

Avoid:

- competitive leaderboards
- excessive badges
- streak pressure language
- dense admin dashboards in the mobile devotee flow

