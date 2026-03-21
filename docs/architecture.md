# Architecture

This document describes the technical architecture of the Volunteer Coordinator application. It covers the tech stack, infrastructure, database schema, API surface, and security model.

This is a living document. It reflects the MVP design and notes where later phases require changes.

---

## Tech Stack

### Frontend
- **React 18 + TypeScript** — component model, type safety
- **Vite** — build tooling, fast dev server
- **React Router v6** — client-side routing
- **TanStack Query** — server state management, caching, optimistic updates
- **Deployed on Vercel** — free tier, global CDN, zero-config deploys from GitHub

### Backend
- **Python 3.12 + FastAPI** — async HTTP framework, automatic OpenAPI docs
- **SQLAlchemy 2.x (async)** — ORM with async support
- **Alembic** — database migrations
- **Pydantic v2** — request/response validation, settings management
- **Resend** — transactional email delivery (3,000 emails/month free)
- **Deployed on Railway** — managed container hosting, PostgreSQL add-on optional

### Database
- **PostgreSQL 15** via **Supabase** — managed Postgres, free tier (500MB, 2 projects)
- Supabase is used as a Postgres host only — no Supabase Auth, no Supabase Realtime in use

### Infrastructure Costs (MVP estimate)

| Service | Tier | Estimated Cost |
|---------|------|---------------|
| Vercel (frontend) | Hobby (free) | $0/month |
| Railway (backend) | Starter | ~$5–10/month |
| Supabase (database) | Free | $0/month |
| Resend (email) | Free (3k/month) | $0/month |
| **Total** | | **~$5–10/month** |

At meaningful scale (100+ events/month, thousands of volunteers), migration to AWS is straightforward:
- Frontend: S3 + CloudFront
- Backend: ECS Fargate or Lambda
- Database: RDS PostgreSQL
- Email: SES

---

## Repository Structure

```
volunteer_tracker/
├── backend/
│   ├── app/
│   │   ├── api/          # Route handlers (organized by resource)
│   │   ├── core/         # Config, security utilities, email
│   │   ├── db/           # Database session, base model
│   │   ├── models/       # SQLAlchemy ORM models
│   │   ├── schemas/      # Pydantic request/response schemas
│   │   └── main.py       # FastAPI app factory
│   ├── migrations/       # Alembic migration files
│   └── tests/
├── frontend/
│   ├── src/
│   │   ├── components/   # Shared UI components
│   │   ├── pages/        # Route-level page components
│   │   ├── api/          # API client functions
│   │   └── main.tsx
│   └── index.html
└── docs/
```

---

## Database Schema

### MVP Schema (Phase 0 / Phase 1)

#### `events`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Internal identifier |
| `public_token` | VARCHAR(64) UNIQUE NOT NULL | Embedded in signup URL; safe to expose |
| `organizer_token_hash` | VARCHAR(128) NOT NULL | bcrypt hash of the organizer token; never returned to client |
| `title` | VARCHAR(255) NOT NULL | Event title |
| `event_date` | DATE NOT NULL | Date of the event |
| `start_time` | TIME NOT NULL | Start time |
| `end_time` | TIME | Optional end time |
| `location` | TEXT NOT NULL | Free text location |
| `notes` | TEXT | Optional public notes shown on signup page |
| `max_volunteers` | INTEGER | NULL means no cap |
| `organizer_email` | VARCHAR(255) NOT NULL | For sending dashboard link and resend-link flow |
| `created_at` | TIMESTAMPTZ NOT NULL | Auto-set on insert |
| `updated_at` | TIMESTAMPTZ NOT NULL | Auto-set on update |

**Indexes:**
- `public_token` — unique index (lookup by public token)
- `organizer_email` — index (resend-link lookup)

---

#### `volunteers`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Internal identifier |
| `event_id` | UUID (FK → events.id) NOT NULL | The event this signup belongs to |
| `name` | VARCHAR(255) NOT NULL | Volunteer's name |
| `email` | VARCHAR(255) NOT NULL | Volunteer's email |
| `phone` | VARCHAR(50) | Optional |
| `status` | ENUM('signed_up', 'attended') NOT NULL | Default: 'signed_up' |
| `signed_up_at` | TIMESTAMPTZ NOT NULL | Auto-set on insert |
| `attended_at` | TIMESTAMPTZ | Set when status is marked 'attended' |

**Indexes:**
- `(event_id, email)` — unique index (prevents duplicate signups per event)
- `event_id` — index (dashboard lookup)

---

#### `waitlist`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Internal identifier |
| `event_id` | UUID (FK → events.id) NOT NULL | The event this waitlist entry belongs to |
| `name` | VARCHAR(255) NOT NULL | Volunteer's name |
| `email` | VARCHAR(255) NOT NULL | Volunteer's email |
| `phone` | VARCHAR(50) | Optional |
| `position` | INTEGER NOT NULL | Order in the waitlist (1-indexed) |
| `joined_at` | TIMESTAMPTZ NOT NULL | Auto-set on insert |

**Indexes:**
- `(event_id, email)` — unique index (prevents duplicate waitlist entries per event)
- `(event_id, position)` — index (ordered waitlist display)

---

### Phase 2 Additions

#### `organizers`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Internal identifier |
| `email` | VARCHAR(255) UNIQUE NOT NULL | Primary identifier for the account |
| `created_at` | TIMESTAMPTZ NOT NULL | Auto-set on insert |
| `last_login_at` | TIMESTAMPTZ | Updated on each verified login |

#### `auth_tokens`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Internal identifier |
| `organizer_id` | UUID (FK → organizers.id) NOT NULL | The organizer this token belongs to |
| `token_hash` | VARCHAR(128) NOT NULL | Hash of the magic link token |
| `expires_at` | TIMESTAMPTZ NOT NULL | Short-lived (15 minutes) |
| `used_at` | TIMESTAMPTZ | Set when token is consumed; single-use |

#### `events` additions (Phase 2 migration)

| Column | Type | Notes |
|--------|------|-------|
| `organizer_id` | UUID (FK → organizers.id) | NULL for events created before Phase 2 |

---

## Token Security Model

### Overview

Two tokens are generated per event. They have different purposes and different security properties.

```
public_token      — safe to share publicly
organizer_token   — must be kept private by the organizer
```

### `public_token`
- Generated as a URL-safe random 32-byte string (encoded as hex or base64url)
- Stored in plaintext in the `events.public_token` column
- Embedded in the public signup URL: `/signup/<public_token>`
- Allows: reading event details, submitting a signup
- Does not allow: viewing the volunteer list, marking attendance, exporting data

### `organizer_token`
- Generated as a URL-safe random 32-byte string
- **Never stored in plaintext.** Hashed with bcrypt before storing in `events.organizer_token_hash`
- Returned to the client exactly once: in the response to `POST /events` and in the email to the organizer
- Embedded in the private dashboard URL: `/dashboard/<organizer_token>`
- On each dashboard request, the token is extracted from the URL, hashed, and compared to the stored hash
- Allows: full event management (view volunteers, mark attendance, export, manage waitlist)

### Token Generation (Python)

```python
import secrets
import bcrypt

def generate_token() -> str:
    return secrets.token_urlsafe(32)

def hash_token(token: str) -> str:
    return bcrypt.hashpw(token.encode(), bcrypt.gensalt()).decode()

def verify_token(token: str, hashed: str) -> bool:
    return bcrypt.checkpw(token.encode(), hashed.encode())
```

### Why not JWT?

JWTs for this use case introduce unnecessary complexity: they can be validated without a database lookup, but that means they can't be revoked. For organizer dashboard links, we want the ability to invalidate a compromised link in the future (Phase 2+). A database-backed token hash provides that without the statefulness trade-offs.

---

## API Surface

All endpoints are prefixed with `/api/v1`.

### Public Endpoints (no auth required)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/events` | Create a new event |
| `GET` | `/events/<public_token>` | Get event details for the signup page |
| `POST` | `/events/<public_token>/signup` | Submit a volunteer signup |
| `POST` | `/resend-link` | Resend organizer dashboard link by email |

### Organizer Endpoints (organizer token in path)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/dashboard/<organizer_token>` | Get event + all volunteers + waitlist |
| `PATCH` | `/dashboard/<organizer_token>/volunteers/<id>/attended` | Toggle attendance status |
| `POST` | `/dashboard/<organizer_token>/waitlist/<id>/promote` | Promote waitlisted volunteer |
| `GET` | `/dashboard/<organizer_token>/export` | Download CSV of all volunteers |

### Phase 2 Endpoints (auth required)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/auth/login` | Send magic link to email |
| `GET` | `/auth/verify` | Verify magic link token, issue session |
| `GET` | `/account/events` | List all events for authenticated organizer |
| `PATCH` | `/events/<id>` | Edit event (authenticated) |
| `DELETE` | `/events/<id>` | Soft-delete event (authenticated) |
| `POST` | `/account/claim-event` | Link a legacy event to the current account |

---

## Infrastructure Diagram

```
                        ┌─────────────────┐
                        │   Volunteer     │
                        │   (browser)     │
                        └────────┬────────┘
                                 │ HTTPS
                        ┌────────▼────────┐
                        │   Vercel CDN    │
                        │  React + Vite   │
                        │  (static build) │
                        └────────┬────────┘
                                 │ HTTPS API calls
                        ┌────────▼────────┐
                        │    Railway      │
                        │    FastAPI      │
                        │    (container)  │
                        └────┬───────┬────┘
                             │       │
              ┌──────────────▼──┐  ┌─▼──────────────┐
              │   Supabase      │  │   Resend        │
              │   PostgreSQL    │  │   (email API)   │
              └─────────────────┘  └────────────────┘
```

---

## Environment Variables

### Backend

```
DATABASE_URL          PostgreSQL connection string
SECRET_KEY            Used for any signed tokens (Phase 2+)
RESEND_API_KEY        Resend transactional email API key
FRONTEND_URL          Base URL for constructing frontend links in emails
ENVIRONMENT           "development" | "staging" | "production"
```

### Frontend

```
VITE_API_BASE_URL     Base URL of the FastAPI backend
```

---

## Key Design Decisions

### Why managed hosting over AWS for MVP?
AWS provides more control but introduces operational complexity that is not justified at MVP scale. A Railway + Supabase setup requires near-zero ops overhead, costs ~$5–10/month, and can be migrated to AWS without code changes (only environment variables and infrastructure config change). The application is written to be infrastructure-agnostic.

### Why PostgreSQL over a simpler option (SQLite, DynamoDB)?
PostgreSQL provides ACID guarantees, a well-understood query model, and full support for the relational data we need (events → volunteers → waitlist). SQLite does not support multi-process writes needed in production. DynamoDB introduces access pattern complexity for a relational domain. PostgreSQL via Supabase is free at our scale.

### Why bcrypt for token hashing instead of SHA-256?
Organizer tokens are essentially credentials. If the database were ever compromised, bcrypt hashing means tokens could not be recovered in bulk. SHA-256 without a salt would be vulnerable to rainbow table attacks on short random tokens. bcrypt is the appropriate choice.

### Why Resend over SES or SendGrid?
Resend has a generous free tier (3,000 emails/month), a clean developer API, and excellent deliverability. SES requires domain verification and IAM setup — unnecessary complexity for MVP. SendGrid's free tier is more restrictive and their UX is heavier. Resend is the right starting point; SES is the right migration target at scale.

### Why token-in-URL over a session cookie for organizer access?
In MVP, organizer "authentication" is entirely token-based (the dashboard URL is the credential). This design:
- Requires no session management infrastructure
- Works across devices without login
- Allows co-organizers to share access by sharing the URL
- Is explicitly temporary — Phase 2 replaces it with real auth

The trade-off is that a leaked URL grants dashboard access. This risk is accepted in MVP and mitigated in Phase 2.
