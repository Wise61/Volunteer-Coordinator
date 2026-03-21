# User Flows

This document describes the complete interaction flows for both user types: organizers and volunteers. Each flow includes the happy path and the relevant edge cases that must be handled.

---

## Flow 1: Organizer — Create and Run an Event

### Step 1: Create Event

The organizer navigates to the app's homepage and clicks "Create Event."

They fill out the event form:
- **Title** (required) — e.g. "Spring Food Drive"
- **Date** (required)
- **Start time** (required)
- **End time** (optional)
- **Location** (required, free text) — e.g. "123 Main St, Community Center Room B"
- **Notes** (optional) — shown on the public signup page
- **Max volunteers** (optional) — if set, enforces a cap with waitlist
- **Organizer email** (required) — used to send the dashboard link; not displayed publicly

They click "Create Event."

**System actions:**
- Generate a UUID-based `public_token` (used in the signup URL)
- Generate a separate UUID-based `organizer_token` (used in the dashboard URL)
- Store both tokens (organizer token stored as a hash) against the event record
- Send an email to the organizer with their private dashboard link
- Redirect organizer to the event confirmation screen

---

### Step 2: Event Confirmation Screen

After creation, the organizer sees a confirmation screen with:

- Event title and details summary
- **Public signup link** (copyable): `https://app.example.com/signup/<public_token>`
- **Your private dashboard link** (copyable): `https://app.example.com/dashboard/<organizer_token>`
- A note: "This dashboard link has been sent to [organizer email]. Save it — you'll need it to manage this event."

The organizer copies the signup link and shares it (email, social media, text message, etc.).

---

### Step 3: View Volunteer Dashboard

The organizer opens their private dashboard link in a browser (from email or saved bookmark).

They see:
- Event details header (title, date, time, location)
- Count of confirmed volunteers (e.g. "12 of 20 spots filled")
- **Confirmed volunteers table:** name, email, phone (if provided), attendance status (checkbox)
- **Waitlist section** (if cap is set and reached): name, email, position in queue, "Promote" button
- Button to copy public signup link
- Button to export CSV

The dashboard updates in real time (or on refresh) as new volunteers sign up.

---

### Step 4: Manage Waitlist (if applicable)

If a volunteer cancels or the organizer wants to expand capacity:

1. Organizer clicks "Promote" next to a waitlisted volunteer
2. The volunteer is moved from the waitlist to the confirmed list
3. Their status updates immediately on the dashboard
4. No automatic notification is sent to the volunteer in MVP (Phase 3 adds this)

If the organizer wants to add more volunteers beyond the original cap, they can promote freely — the cap is a soft default, not a hard server-side lock for the organizer.

---

### Step 5: Mark Attendance

During or after the event, the organizer opens the dashboard on their phone.

For each volunteer who shows up:
- They tap the checkbox (or toggle) next to the volunteer's name
- Status changes from "Signed up" to "Attended"

Bulk option: "Mark all as attended" for cases where everyone showed up.

---

### Step 6: Export for Follow-Up

After the event, the organizer clicks "Export CSV."

The downloaded file includes:
- Name
- Email
- Phone (if provided)
- Status (Signed up / Attended / Waitlisted)

Alternatively, "Copy all emails" copies a comma-separated list of confirmed volunteer emails to the clipboard for direct use in an email client.

---

## Edge Cases: Organizer Flow

### Lost dashboard link
The organizer cannot find the email and has no saved link.

- On the homepage, there is a "Resend my dashboard link" option
- Organizer enters their email address
- If that email matches one or more events, they receive a new email listing all their dashboard links
- The organizer token itself does not change — only a new email is sent

### Organizer uses a different device
No login required. The dashboard link works on any device, any browser. The organizer can share the dashboard link with a co-organizer or volunteer lead.

### Organizer wants to edit the event
Not supported in MVP. The organizer must note corrections manually and communicate them through their own channels. Event editing is a Phase 2 feature.

### Organizer wants to delete the event
Not supported in MVP. Scope is kept minimal.

---

## Flow 2: Volunteer — Sign Up for an Event

### Step 1: Open Signup Link

The volunteer clicks or taps the signup link shared by the organizer.

They are taken to the public signup page, which shows:
- Event title
- Date, time, and location
- Notes (if provided by organizer)
- Number of spots remaining (if a cap is set) — e.g. "8 spots left"
- Signup form

---

### Step 2: Fill Out Form

The volunteer fills in:
- **Name** (required)
- **Email** (required)
- **Phone** (optional)

They click "Sign me up."

---

### Step 3: Confirmation

**Happy path (event has open spots):**

The volunteer is shown a confirmation page:
- "You're signed up for [Event Title]"
- Date, time, and location repeated for easy reference
- "A confirmation has been sent to [email]"

A confirmation email is sent immediately with the same details.

**Waitlist path (event is full):**

The form changes to a waitlist form before the volunteer submits (the page clearly states "This event is currently full. Join the waitlist to be notified if a spot opens.").

On submission, the volunteer sees a waitlist confirmation page:
- "You've been added to the waitlist for [Event Title]"
- "We'll reach out if a spot becomes available."

A waitlist confirmation email is sent.

---

## Edge Cases: Volunteer Flow

### Volunteer submits with the same email twice
- If the volunteer submits the same email for the same event, the system recognizes the duplicate
- They are shown a message: "You're already signed up for this event."
- No duplicate record is created
- No second email is sent

### Event link is invalid or malformed
- The volunteer is shown a simple error page: "This signup link is not valid. Please check the link and try again."

### Event is in the past
- The volunteer is shown: "This event has already passed. Signups are closed."
- The organizer can optionally disable signups manually (Phase 2 feature)

### Volunteer wants to cancel
- Not supported in MVP. Volunteers must contact the organizer directly.
- This is an intentional scope constraint — cancellation logic adds significant complexity (waitlist promotion, notifications, etc.) and is deferred to Phase 3.

### Email confirmation not received
- No self-service recovery in MVP
- Volunteer should contact the organizer directly
- The organizer can verify signup status from the dashboard

### Form submitted with invalid email format
- Client-side and server-side validation rejects malformed email addresses
- Volunteer sees an inline error: "Please enter a valid email address."

---

## Flow 3: Organizer — Resend Dashboard Link

1. Organizer goes to the app homepage
2. Clicks "Find my events" or "Resend my link"
3. Enters their organizer email
4. System looks up all events associated with that email
5. Sends a single email listing each event with its dashboard link
6. Organizer sees: "If that email is associated with any events, you'll receive a link shortly."

This flow uses the same organizer email captured at event creation. It does not require a password or account.

---

## Summary: What Happens With Each Token

| Token | Embedded in | Who has it | What it allows |
|-------|------------|------------|----------------|
| `public_token` | Signup URL | Anyone | View event details, submit signup form |
| `organizer_token` | Dashboard URL | Organizer only | View all volunteers, mark attendance, export, manage waitlist |

The `organizer_token` is never exposed in the frontend except at the moment of creation and in the email sent to the organizer. It is stored as a hash server-side and validated on each dashboard request.
