# Scope (MVP)

## Purpose

Define the exact scope of the initial MVP for the Volunteer Coordination tool.

This document exists to prevent scope creep and ensure the project remains focused on solving a single, high-impact problem well.

---

## Problem Statement

Small nonprofits struggle to coordinate volunteer events due to fragmented tools (spreadsheets, forms, email). This leads to:

- Lost or inconsistent data
- Time-consuming manual coordination
- Poor visibility into attendance
- Inefficient follow-up

---

## Target User

### Primary User: Organizer
- Staff member or volunteer responsible for coordinating events
- Non-technical or moderately technical
- Needs a fast, reliable way to manage volunteers

### Secondary User: Volunteer
- Individual signing up for an event
- Should experience zero friction

---

## Core User Flows

### Organizer Flow

1. Create an event
2. Receive a private dashboard link (shown on screen + emailed to organizer)
3. Share the public signup link
4. View list of signed-up volunteers and waitlisted volunteers
5. Promote volunteers from the waitlist if slots open
6. Mark attendance during or after the event
7. Export or copy volunteer contact list for follow-up

---

### Volunteer Flow

1. Open signup link
2. Enter name and email (phone optional)
3. Submit form
4. See confirmation page
5. Receive confirmation email with event details

If the event is full, the volunteer is offered a waitlist signup instead.

No account creation. No login required.

---

## MVP Features

### 1. Event Creation
- Title
- Date/time
- Location (free text)
- Optional notes
- Optional max number of volunteers (cap)
- On creation: system generates two tokens
  - A `public_token` embedded in the shareable signup URL
  - An `organizer_token` embedded in the private dashboard URL
- Both URLs displayed to the organizer on the confirmation screen
- Dashboard URL emailed to the organizer's provided address

---

### 2. Public Signup Page
- Unique URL per event (`/signup/<public_token>`)
- Fields:
  - Name (required)
  - Email (required)
  - Phone (optional)
- If event is under cap (or has no cap): registers as confirmed volunteer
- If event has hit cap: offers waitlist signup with clear messaging
- Simple confirmation page on submission
- Confirmation email sent to volunteer with event title, date, time, and location

---

### 3. Organizer Dashboard
- Accessed via private URL (`/dashboard/<organizer_token>`)
- No login required — token is the credential
- Displays:
  - Event details (title, date/time, location, notes)
  - Confirmed volunteer list (name, email, phone if provided, attendance status)
  - Waitlist (name, email, position in queue)
- Actions available:
  - Mark individual volunteers as "attended"
  - Promote a waitlisted volunteer to confirmed (if slots available or cap is overridden)
  - Copy the public signup link

---

### 4. Attendance Tracking
- Organizer can mark each volunteer as "attended" from the dashboard
- Must be usable on mobile during events
- Bulk mark-all-attended option for speed

---

### 5. Waitlist Management
- When a volunteer cap is set and reached, new signups go to a waitlist
- Waitlist volunteers receive a confirmation email noting they are waitlisted
- Organizer can promote volunteers from the waitlist to confirmed
- Promoted volunteers do not receive an automatic email in MVP (Phase 3 adds this)

---

### 6. Follow-Up Support
- Copy all volunteer emails to clipboard (one-click)
- CSV export of full volunteer list (name, email, phone, attended status)

---

### 7. Transactional Email
- Organizer receives dashboard link email immediately after creating an event
- Volunteer receives confirmation email immediately after signing up
- Waitlisted volunteer receives a waitlist confirmation email
- Email provider: Resend (free tier: 3,000 emails/month)
- No marketing email, no campaigns, no opt-out management in MVP

---

## Explicit Non-Goals (MVP)

The following are **intentionally excluded**:

### Traditional Account System
- No username/password authentication for organizers
- No volunteer accounts
- Access control is handled via secret organizer tokens (see architecture.md)
- A proper auth system is Phase 2

### Organizer-to-Volunteer Communication
- No ability to send custom emails to volunteers from within the app
- No pre-event reminders or post-event follow-up emails (Phase 3)
- Organizers use the exported contact list for their own outreach

### CRM Functionality
- No long-term tracking of volunteers across events
- No contact profiles beyond a single event
- Cross-event volunteer history is Phase 4

### Donations / Payments
- No donation tracking
- No payment integration

### Customization
- No custom fields
- No configurable workflows
- No theming or branding

### Integrations
- No integrations with external tools (Google Sheets, Mailchimp, etc.)

### Recurring Events
- No event series or templates (Phase 5)

---

## Constraints

- Must be usable with zero onboarding or training
- Event creation should take < 60 seconds
- Volunteer signup should take < 15 seconds
- Must be mobile-friendly for both organizer and volunteer
- Infrastructure cost target: < $25/month at MVP scale (managed hosting)
- Email delivery must be reliable — organizer dashboard link is the only recovery mechanism

---

## Success Criteria (MVP)

The MVP is successful if:

- At least 2–5 nonprofits actively use the tool for real events
- Organizers can run events without reverting to spreadsheets
- Feedback indicates the tool is simpler than current workflows
- Users can complete the full event flow without guidance
- No organizer reports losing access to their dashboard due to a lost link

---

## Future Considerations (Out of Scope for Now)

These are addressed in the roadmap (docs/roadmap.md):

- Organizer accounts and multi-event dashboard (Phase 2)
- Email reminders and post-event follow-up (Phase 3)
- Volunteer history across events (Phase 4)
- Recurring events and templates (Phase 5)
- Analytics and reporting (Phase 6)

---

## Guiding Principle

> Build the simplest possible system that successfully supports one complete volunteer event lifecycle.

Any feature that does not directly support this goal should be excluded from the MVP.
