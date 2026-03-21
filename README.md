# Volunteer Coordinator

A simple, fast, no-friction tool for managing volunteer events.

## Overview

Built for small nonprofits (5–25 staff) who coordinate volunteers using a mix of spreadsheets, forms, and email threads. This tool replaces that workflow with a single, streamlined system:

1. Create an event
2. Share a signup link
3. Volunteers sign up — no account required
4. Track attendance on the day
5. Export contacts for follow-up

No accounts required for volunteers. Organizer access is handled via a private dashboard link sent by email — no passwords, no login pages.

## Goals

- Reduce time spent coordinating volunteers
- Eliminate messy spreadsheets and manual tracking
- Provide a clean, intuitive workflow that works immediately
- Be usable by non-technical users with zero onboarding

## Non-Goals

This project is intentionally limited in scope. It is **not**:

- A full CRM
- A donor management system
- A highly customizable platform
- An all-in-one nonprofit solution

Simplicity and usability are prioritized over flexibility. See [docs/vision.md](docs/vision.md) for the full product direction.

## Core Features (MVP)

- **Event creation** — title, date/time, location, notes, optional volunteer cap
- **Public signup page** — no login required; supports cap enforcement and waitlist
- **Organizer dashboard** — private link per event; view volunteers, manage waitlist
- **Attendance tracking** — mobile-friendly check-in during the event
- **Transactional email** — organizer receives their dashboard link; volunteers receive confirmation
- **Export** — CSV download and one-click email copy for follow-up

## How Access Works

Organizers do not create an account to use the MVP. When an event is created:

- A **public signup link** is generated and shared with volunteers
- A **private dashboard link** is generated and emailed to the organizer

The dashboard link is the organizer's credential — anyone with that link can manage the event. A proper authentication system is planned for Phase 2.

## Target Users

- Small nonprofits (5–25 staff)
- Volunteer coordinators and event organizers
- Teams currently relying on spreadsheets, Google Forms, or manual processes

## Design Principles

- **Fast:** Events created in under 60 seconds. Volunteer signup in under 15 seconds.
- **Simple:** Minimal fields, minimal decisions, no training required.
- **Accessible:** Mobile-first for both organizers and volunteers.
- **Focused:** Solve one problem well. Expand scope only after validation.

## Tech Stack

| Layer | Technology | Hosting |
|-------|-----------|---------|
| Frontend | React 18 + TypeScript (Vite) | Vercel |
| Backend | Python 3.12 + FastAPI | Railway |
| Database | PostgreSQL 15 | Supabase |
| Email | Resend | — |

Infrastructure is designed to run at MVP scale for ~$5–10/month. The architecture is infrastructure-agnostic and documented for AWS migration when needed. See [docs/architecture.md](docs/architecture.md) for full details.

## Documentation

| Document | Description |
|----------|-------------|
| [docs/vision.md](docs/vision.md) | Long-term product direction and guiding principles |
| [docs/scope.md](docs/scope.md) | MVP scope, feature definitions, explicit non-goals |
| [docs/roadmap.md](docs/roadmap.md) | Phased roadmap (Phase 0–6), dependency-ordered with checkboxes |
| [docs/user_flows.md](docs/user_flows.md) | Organizer and volunteer flows with edge cases |
| [docs/architecture.md](docs/architecture.md) | Tech stack, DB schema, API surface, security model |

## Project Status

Early development — Phase 0 (Foundation) not yet started.

See [docs/roadmap.md](docs/roadmap.md) for the full plan and progress tracking.

## Contributing

Contributions, feedback, and ideas are welcome — especially from those with experience in nonprofit operations or volunteer coordination.

## License

This project is licensed under the AGPL-3.0 License.

The goal is to ensure that improvements remain open and accessible to the nonprofit community.
