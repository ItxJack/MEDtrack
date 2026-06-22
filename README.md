# MedTrack
**Full-Stack Clinic Management System** · ACA Summer Project, IIT Kanpur · Team of 3

---

## Overview

A clinic platform serving 4 roles — patient, reception, doctor, admin — with online booking, walk-in registration, a live doctor queue, visit notes, and an admin dashboard; built on a decoupled Flask JSON API with real-time WebSocket updates consumed by a React SPA.

---

## Tech Stack

| Layer | Tech |
|---|---|
| Backend | Python · Flask · Flask-SocketIO · SQLite (raw SQL) |
| Frontend | React 18 · React Router v6 · Axios · Socket.io client |
| Auth | JWT (PyJWT) · Werkzeug PBKDF2 hashing |
| Real-time | WebSockets via Flask-SocketIO |

---

## Architecture

**Frontend — React 18 SPA**
- Role-based protected routing — each role lands on its own dashboard; unauthorized routes redirect automatically.
- Socket.io client subscribes to `appointment_update` events and re-fetches instantly — no polling.
- Shared Axios instance injects `Authorization: Bearer <JWT>` on every request automatically.

**Backend — Flask + SQLite**
- 5 Flask blueprints: `auth, patient, doctor, reception, admin` — each owns its own routes.
- Single `db.py` owns all SQLite access; all queries parameterized (zero SQL injection surface).
- `patients` and `users` kept as separate tables — walk-in patients need no app account.

**Connection — REST + WebSockets**
- Flask serves pure JSON on `/api/*`; React consumes it via Axios.
- Flask-SocketIO pushes `appointment_update` on every state change; React pages re-fetch on the event.
- JWT validated on every request via `@login_required` / `@role_required` decorators.

---

## Key Engineering Decisions

**DB-layer double-booking prevention**

Booking races are closed at the database layer with a partial unique index:
```sql
UNIQUE(doctor_id, appt_date, appt_time) WHERE status != 'cancelled'
```
If two requests race, SQLite rejects the second with an `IntegrityError` — the API returns 409. Cancelled slots automatically become rebookable with no application-level locking needed.

**Real-time via WebSocket push**

Every booking creation, status change, or cancellation emits an `appointment_update` event via Flask-SocketIO. All live pages (patient tracker, doctor queue, reception board) subscribe to this single event and re-fetch — zero polling logic anywhere in the codebase.

---

## Project Structure
