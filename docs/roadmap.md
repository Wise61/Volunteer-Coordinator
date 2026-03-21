# Roadmap

This roadmap is organized by dependency order, not by timeline. Each phase produces a working, shippable increment. Phases should be completed sequentially — later phases build directly on the foundations established by earlier ones.

---

## Phase 0: Foundation

**Goal:** A working skeleton with no user-facing features. Everything needed to build on top of.

This phase produces no visible product. It produces the infrastructure, tooling, and conventions that every subsequent phase depends on.

### Infrastructure & Deployment
- [ ] Initialize monorepo structure (frontend, backend, docs)
- [ ] Set up Python/FastAPI project with dependency management (uv or Poetry)
- [ ] Set up React + TypeScript project (Vite)
- [ ] Configure Railway project for backend deployment
- [ ] Configure Vercel project for frontend deployment
- [ ] Set up Supabase project (PostgreSQL database)
- [ ] Configure environment variable management (local + deployed)
- [ ] Establish dev/staging/production environment separation

### Database
- [ ] Write initial migration: `events` table
- [ ] Write initial migration: `volunteers` table
- [ ] Write initial migration: `waitlist` table
- [ ] Set up Alembic (or equivalent) for migration management
- [ ] Validate schema against architecture.md

### Backend
- [ ] FastAPI project skeleton with health check endpoint
- [ ] Database connection and session management
- [ ] CORS configuration for frontend domain
- [ ] Error handling and logging middleware
- [ ] Request/response validation with Pydantic models

### Frontend
- [ ] React project skeleton with routing (React Router)
- [ ] Base layout component and design token setup
- [ ] API client utility (fetch wrapper with error handling)
- [ ] Mobile-responsive base styles

### CI/CD
- [ ] GitHub Actions: lint + type check on PR
- [ ] GitHub Actions: run tests on PR
- [ ] GitHub Actions: deploy to staging on merge to main
- [ ] GitHub Actions: deploy to production on tagged release

### Email
- [ ] Resend account and API key configured
- [ ] Email sending utility in backend
- [ ] Transactional email templates: organizer dashboard link, volunteer confirmation, waitlist confirmation

---

## Phase 1: MVP

**Goal:** A complete, working event lifecycle from creation to post-event export.

Depends on: **Phase 0**

This phase produces the first version that real nonprofits can use for real events.

### Event Creation
- [ ] `POST /events` — create event, generate tokens, return both URLs
- [ ] Event creation form (frontend) — title, date/time, location, notes, cap, organizer email
- [ ] Form validation (required fields, date in future, valid email)
- [ ] Event confirmation screen — display both links, copy-to-clipboard functionality
- [ ] Send organizer dashboard link email on event creation

### Public Signup Page
- [ ] `GET /events/<public_token>` — return event details for signup page
- [ ] `POST /events/<public_token>/signup` — register volunteer (enforces cap, routes to waitlist if full)
- [ ] Signup page (frontend) — event details, signup form
- [ ] Cap enforcement: show remaining spots, transition to waitlist form when full
- [ ] Confirmation page (frontend) — happy path and waitlist path
- [ ] Duplicate email detection — return friendly message, no duplicate record
- [ ] Past event detection — show "signups closed" message
- [ ] Invalid token handling — show clear error page
- [ ] Send volunteer confirmation email on signup
- [ ] Send waitlist confirmation email when routed to waitlist

### Organizer Dashboard
- [ ] `GET /dashboard/<organizer_token>` — return event + all volunteers + waitlist
- [ ] Dashboard page (frontend) — event header, volunteer table, waitlist section
- [ ] Invalid/expired token handling — show clear error page
- [ ] Copy public signup link button
- [ ] Mobile-responsive layout for on-site use

### Attendance Tracking
- [ ] `PATCH /dashboard/<organizer_token>/volunteers/<id>/attended` — toggle attendance status
- [ ] Attendance checkbox/toggle on dashboard (frontend)
- [ ] "Mark all attended" bulk action
- [ ] Optimistic UI update with rollback on error

### Waitlist Management
- [ ] `POST /dashboard/<organizer_token>/waitlist/<id>/promote` — move volunteer to confirmed list
- [ ] Promote button on waitlist section (frontend)
- [ ] Dashboard reflects updated counts and tables immediately

### Follow-Up & Export
- [ ] `GET /dashboard/<organizer_token>/export` — return CSV of all volunteers + status
- [ ] CSV download button (frontend)
- [ ] "Copy all emails" — copy comma-separated confirmed volunteer emails to clipboard

### Resend Dashboard Link
- [ ] `POST /resend-link` — accepts organizer email, sends email with all matching dashboard links
- [ ] "Find my events" form on homepage (frontend)

---

## Phase 2: Organizer Auth + Multi-Event Dashboard

**Goal:** Give organizers persistent identities so they can manage multiple events in one place.

Depends on: **Phase 1**

This phase is the unlock for Phases 3, 4, and 5. Without organizer accounts, volunteer history and cross-event communication have no ownership model.

### Magic Link Authentication
- [ ] `POST /auth/login` — accept email, send magic link (time-limited JWT)
- [ ] `GET /auth/verify` — validate token, issue session cookie
- [ ] Session management (server-side or signed JWT)
- [ ] Login page (frontend) — "Send me a login link" form
- [ ] Auth guard for protected routes

### Organizer Account Model
- [ ] Add `organizers` table (migration)
- [ ] Account is created automatically on first magic link login
- [ ] Events created post-auth are associated with organizer account
- [ ] Update `POST /events` to attach event to authenticated organizer if logged in

### Retroactive Event Claiming
- [ ] `POST /account/claim-event` — organizer can link a legacy event (by organizer token) to their account
- [ ] Claim event UI on the dashboard or account page
- [ ] Events created before auth existed can be adopted into an account

### Multi-Event Dashboard
- [ ] `GET /account/events` — return all events owned by the authenticated organizer
- [ ] Multi-event dashboard page (frontend) — list of all events with status and quick stats
- [ ] Ability to create new events from the dashboard (authenticated flow)
- [ ] Event status indicators: upcoming, in progress, past

### Event Management
- [ ] `PATCH /events/<id>` — edit event details (authenticated organizer only)
- [ ] `DELETE /events/<id>` — soft-delete event (authenticated organizer only)
- [ ] Edit event form (frontend)
- [ ] Delete confirmation dialog

---

## Phase 3: Volunteer Communication

**Goal:** Automate the volunteer touchpoints that organizers currently do manually.

Depends on: **Phase 2** (needs organizer accounts for reliable send attribution and reply-to configuration)

### Pre-Event Reminders
- [ ] Background job scheduler (e.g. APScheduler or Celery with Redis)
- [ ] Job: send reminder email to all confirmed volunteers 24 hours before event
- [ ] Reminder email template — event details, location, what to expect
- [ ] Organizer can enable/disable reminders per event (dashboard setting)

### Post-Event Follow-Up
- [ ] Job: send post-event summary email to organizer after event end time
  - Attendance count, volunteer list, link to dashboard
- [ ] Organizer-to-volunteers follow-up: compose and send a custom thank-you email from the dashboard
- [ ] Volunteer reply-to is set to organizer's email (not the system's)

### Waitlist Promotion Notification
- [ ] When organizer promotes a waitlisted volunteer, send them an email confirming their spot
- [ ] Email includes event details and a reminder that their spot is now confirmed

### Volunteer Cancellation
- [ ] `DELETE /events/<public_token>/signup/<volunteer_id>` — cancel signup (via unique link in confirmation email)
- [ ] Cancellation link included in volunteer confirmation email
- [ ] On cancellation: slot opens up, top waitlisted volunteer is promoted automatically and notified

---

## Phase 4: Volunteer History

**Goal:** Build lightweight volunteer profiles so organizers can understand who their most engaged volunteers are.

Depends on: **Phase 2** (needs organizer accounts to scope history meaningfully)

### Cross-Event Volunteer Tracking
- [ ] Match volunteers across events by email address
- [ ] `volunteers` table gains a normalized `volunteer_profiles` concept (same email = same person)
- [ ] Volunteer participation count displayed in organizer dashboard

### Volunteer Profile View
- [ ] Organizer can click a volunteer's name to see their history
- [ ] Shows: events attended, events signed up for, first event, most recent event
- [ ] Simple read-only view — no editing of volunteer data by organizer in MVP of this phase

### Repeat Volunteer Indicators
- [ ] Flag on dashboard: highlight volunteers who have attended 3+ events
- [ ] "Returning volunteer" label visible to organizer in volunteer table

### Volunteer Profile Self-Service (optional, Phase 4b)
- [ ] Volunteer can view their own history via email-verified link
- [ ] Shows events they've attended (own org only)

---

## Phase 5: Recurring Events and Templates

**Goal:** Reduce organizer overhead for events that happen on a regular schedule.

Depends on: **Phase 2** (needs accounts to own event series)

### Event Duplication
- [ ] "Duplicate this event" button on event dashboard
- [ ] Copies all settings to a new event with a new date (date left blank or set to +7 days)
- [ ] New event gets its own tokens; volunteer list is empty

### Event Templates
- [ ] Organizer can save an event as a template (named, stored on their account)
- [ ] "Create from template" option on the new event form
- [ ] Template stores: title pattern, location, notes, cap, reminder settings

### Recurring Event Series
- [ ] Organizer can set an event to recur (weekly, biweekly, monthly)
- [ ] System generates upcoming instances in the series (up to N in advance)
- [ ] Series dashboard: view all instances, which are upcoming/past, aggregate stats

---

## Phase 6: Analytics and Reporting

**Goal:** Give organizers the data to understand their volunteer program and make better decisions.

Depends on: **Phase 4** (meaningful analytics require cross-event volunteer history)

### Per-Event Summary
- [ ] Event detail page includes: total signups, attended count, no-show count, waitlist count
- [ ] Attendance rate displayed as a percentage
- [ ] Exportable as PDF or enhanced CSV

### Cross-Event Trends
- [ ] Organizer dashboard shows aggregate stats across all events
  - Total volunteers coordinated, average attendance rate, most active volunteers
- [ ] Trend chart: volunteer signups and attendance over time
- [ ] Top volunteers list: by total events attended

### Volunteer Retention Metrics
- [ ] Returning volunteer rate: what % of attendees have attended before?
- [ ] First-time vs. returning breakdown per event

### Org-Level Report Export
- [ ] Export full volunteer activity report for a date range
- [ ] Useful for grant reporting, board updates, annual reviews

---

## Dependency Map

```
Phase 0: Foundation
    └── Phase 1: MVP
            └── Phase 2: Organizer Auth
                    ├── Phase 3: Volunteer Communication
                    ├── Phase 4: Volunteer History
                    │       └── Phase 6: Analytics
                    └── Phase 5: Recurring Events
```

Each phase must be complete before work begins on any phase that depends on it. Phases 3, 4, and 5 can be worked in parallel once Phase 2 is complete.
