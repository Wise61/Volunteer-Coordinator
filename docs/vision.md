# Vision

## What We Are Building

A volunteer coordination tool that is genuinely easier to use than a spreadsheet.

Not easier for a power user. Easier for a 60-year-old program director who has been running events on Google Sheets for a decade and does not have time to learn software. Easier in a way that is immediately obvious — no tutorial, no onboarding email, no help docs required.

Most nonprofit software fails this test. It is built for compliance, not usability. It accrues features to win RFP checklists. It assumes IT support. It costs hundreds of dollars a month and requires a contract.

This tool does not compete with that software. It solves a single, narrower problem better than any of those tools do, and it does it for free or near-free.

---

## The Problem Worth Solving

Volunteer coordination today looks like this:

- A coordinator creates a Google Form
- They paste the form link into an email or a Facebook post
- Responses flow into a Google Sheet
- They manually check who showed up
- They export the sheet and paste emails into Mailchimp for follow-up
- They do this again next week

Every step is manual. Every step is an opportunity for data to get lost or duplicated. The coordinator is doing systems integration work instead of running their program.

This is not a technology failure. It is an absence of a focused tool.

---

## The Long-Term Vision

Three years from now, this tool looks like this:

- A coordinator at any small nonprofit can set up an event in under 60 seconds
- Volunteers sign up without friction, receive a confirmation, and get a reminder the day before
- The coordinator arrives at the event, opens the app on their phone, and checks people in by tapping names
- After the event, the system automatically emails everyone who attended with a thank-you and a link to the next event
- The coordinator can see, at a glance, which volunteers show up consistently — their most reliable people
- All of this costs the organization nothing, or close to nothing

That is the whole product. There is no CRM. There is no donor module. There is no grant tracking. There are no integrations, no API marketplace, no enterprise tier.

The scope constraint is the strategy. The organizations we serve do not have the time or budget to evaluate, implement, or maintain complex software. They need something that works immediately and stays out of the way.

---

## Guiding Principles

### 1. Simplicity is non-negotiable
Every feature must justify its existence by reducing work for the user. If a feature adds a decision, a setting, or a screen without removing more friction than it adds, it does not belong here. The right question is never "can we add this?" — it is "does removing this make the product better?"

### 2. Zero-friction for volunteers
Volunteers are unpaid. Their experience with this tool is a direct reflection of the organization they are supporting. Any friction — an account signup, a confusing form, a broken mobile layout — translates directly into fewer signups. The volunteer flow must be treated as a conversion funnel and optimized accordingly.

### 3. Mobile-first for organizers, mobile-only for volunteers
Coordinators will use this on a laptop when setting up events and on a phone when running them. Volunteers will almost always be on a phone. Both experiences must be designed for mobile first, desktop second.

### 4. Cost must not be a barrier
The organizations using this tool operate on thin margins. Pricing must be transparent and, at small scale, free or near-free. Infrastructure should be sized for actual usage, not theoretical scale. This principle affects every architectural decision.

### 5. Build trust through reliability
A nonprofit using this tool for a real event is trusting us with their volunteer relationships. An email that doesn't deliver, a link that breaks, or data that disappears causes real organizational harm. Reliability is more important than any new feature.

### 6. Own the migration path
We should make it easy for organizations to get their data out. CSV export is not just a feature — it is a trust signal. Organizations that know they can leave are more willing to commit. Being open source reinforces this.

---

## Who We Are Not Building For

Understanding who this product is not for is as important as understanding who it is for.

- **Large nonprofits (50+ staff):** They have dedicated operations staff, existing software investments, and procurement processes. They need Salesforce NPSP, not this.
- **Organizations that need a CRM:** Volunteer coordination is one use case. If an org needs to track donors, grants, and volunteers in one place, they need a different tool.
- **Organizations with complex workflows:** Custom fields, approval chains, conditional logic, integrations — these are real needs, but not our problem to solve.
- **Enterprise or government:** Compliance requirements, SSO, audit trails — out of scope indefinitely.

Staying out of these markets is a deliberate choice. Every feature we decline to build for these segments keeps the product focused for the segment we do serve.

---

## How We Know We've Succeeded

The MVP is validated when:
- Real nonprofit staff use it to run real events without being asked to
- They come back and use it again
- They recommend it to another organization

The product is mature when:
- A coordinator can manage their entire volunteer program — event creation, signups, attendance, history, communication — without opening a spreadsheet
- Organizations with 10 events a year get measurable time savings compared to their previous process
- The tool runs reliably at low cost with minimal maintenance

The product has found its ceiling when organizations outgrow it — and that is fine. The goal is not to capture every use case. The goal is to be the best possible tool for the organizations that fit our scope.
