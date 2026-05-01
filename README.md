# Community Session Invite System

A fully automated online session planning and calendar invite system 
for remote teams and professional communities. Built with Google 
Sheets, Google Forms, Google Calendar, and Zapier.

---

## The Problem

During my time volunteering with an organization, I noticed session attendance 
declining steadily over time. Several factors could explain this, but 
one stood out clearly. Attendees were not receiving calendar invites 
for sessions. Only an event link was shared publicly on WhatsApp and 
LinkedIn. For busy professionals who may have missed those 
announcements, there was no second layer of reminder. No calendar 
block. No notification on the day.

I personally avoided missing sessions by manually copying the link and 
creating my own Google Calendar event each time. Most people do not do 
that. If it is not on your calendar, it does not happen.

Beyond attendance, the org had a growing database of leads — 
professionals who had shown interest in their sessions — with no 
structured way to convert that interest into confirmed attendance. A 
registration system with automatic calendar invites would solve both 
problems at once.

---

## What It Does

Session planning is managed on a Google Sheet — simple, flexible, and 
easy to update as details change. Once a session is marked as 
**Approved**, the automation triggers:

- A Google Calendar event is created automatically
- The event link is written back to the planning sheet

The session can then be announced publicly — this time with a 
registration form link alongside it. When someone registers, they 
automatically receive a Google Calendar invite to the session.

If a session is cancelled, the calendar event is deleted and 
registered attendees receive a cancellation notification. 

The registration sheet also serves as a lead database. The organization can 
track attendance patterns, grow their audience data, and use 
registration records for future session marketing.

---

## Data Integrity and Duplicate Prevention

Data cleaning is handled directly on Google Sheets using formulas 
before the automation runs. This keeps the Zap logic simple, 
reduces unnecessary automation runs, and makes the system easier 
to troubleshoot.

The sheet checks for:
- **Duplicate registrations** — prevents the same person registering 
  multiple times for the same session
- **Invite already sent** — prevents the calendar invite from being 
  sent more than once to the same attendee

Handling this at the sheet level rather than inside the Zap keeps 
the automation clean and predictable.

---

## Architecture

The workflow is split into two separate Zaps to avoid loops and 
reduce complexity.

```
Zap 1: Event Creation and Deletion
        │
        ▼
Google Sheets — Updated Row (Planning Sheet)
  Dedupe column: Done?
  Timezone: Africa/Lagos
        │
        ▼
Branching — Split by Criteria
   ┌──────────────┬──────────────┐
   ▼              ▼
Path A            Path B
(Approved)        (Cancelled)
   │              │
   ▼              ▼
Google Calendar   Google Calendar
Create Event      Search Event
   │              │
   ▼              ▼
Google Sheets     Google Calendar
Update row        Delete Event
(Event ID,        (with notifications)
Hangout link,        │
HTML link)           ▼
                  Google Sheets
                  Search rows by Session ID
                     │
                     ▼
                  Loop Values
                     │
                     ▼
                  Google Sheets
                  Update rows
                  (mark as Event Canceled)


Zap 2: Attendee Registration
        │
        ▼
Google Sheets — Updated Row (Form Response Sheet)
  Dedupe column: Invite Sent
  Timezone: Africa/Lagos
        │
        ▼
Branching — Split by Criteria
   ┌──────────────┬──────────────┐
   ▼              ▼
Path A            Path B
(Both F & G       (Only G empty
empty)            duplicate detected)
   │              │
   ▼              ▼
Google Sheets     Google Sheets
Lookup by         Delete duplicate row
Session ID
   │
   ▼
Google Calendar
Search Event
   │
   ▼
Google Calendar
Add Attendee
   │
   ▼
Google Sheets
Mark Invite Sent
(Column F = True)
```
---

## Workflow Schema
**Zap1-event-creation**
![Zap 1 Architecture](zap1-event-creation.png)

**Zap2-attendee-invite**
![Zap 2 Architecture](zap2-attendee-invite.png)

---
The **Done?** column is the single source of truth that drives 
all automation. Each status maps to a specific action:

| Status | Meaning | Automation triggered |
|---|---|---|
| Planning | Session being drafted | None |
| Approved | Session confirmed | Create calendar event + form |
| Planning On Hold | Paused before approval — event never created | None |
| Cancelled | Session called off | Delete calendar event + notify attendees |
| Done | Session completed | None |

---

## Planning Sheet — Required Columns

| Column | Purpose |
|---|---|
| Session Name | Name of the session |
| Date | Session start date and time |
| End Date | Session end date and time |
| Speaker | Speaker name |
| Speaker Email | Speaker email for calendar invite |
| Done? | Status dropdown — drives all automation |
| Session ID | Unique identifier linking planning sheet to attendee sheet |
| Calendar Event ID | Written by Zapier after event creation |
| Calendar Event Link | Written by Zapier after event creation |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Google Sheets | Session planning, data collection, duplicate prevention |
| Google Forms | Attendee registration |
| Google Calendar | Session events and attendee invites |
| Zapier | Automation engine |

---

## Key Design Decisions

**Data cleaning on the sheet, not inside the Zap.**
Duplicate prevention and invite-sent checks are handled by Google 
Sheets formulas before the automation fires. This keeps the Zap 
logic simple, reduces unnecessary task consumption, and makes 
the system easier to maintain and troubleshoot without needing 
to understand the automation internals.

**Two Zaps instead of one.**
Splitting session planning and attendee invite delivery into 
separate Zaps eliminates the risk of update loops — where 
writing data back to the sheet triggers the automation again. 
Each Zap has a single, clear responsibility.

**Status column as single source of truth.**
All automation decisions flow from the Done? column. This makes 
the system predictable and easy to audit — if something did not 
trigger correctly, the first place to check is the status column.

**Session ID as the link between sheets.**
Each session has a unique ID that connects the planning sheet 
to its attendee response sheet. This allows the cancellation 
flow to find and update all registered attendees for a specific 
session reliably.

---

## Limitations

This workflow was designed and tested using Zapier's free trial. 
Several features used in this build are not available on Zapier's 
free plan. See the service tier comparison below for full details.

**On the free tier, Zapier allows 100 tasks per month.**
Each action step that runs counts as one task. For this workflow:
- Zap 1 (Approved path) uses approximately 3–4 tasks per session 
  created
- Zap 2 uses approximately 3 tasks per attendee registration

This means the free tier supports approximately 25–30 attendee 
registrations or 8–10 session creations per month before the 
limit is reached — suitable only for very low-volume testing.

---

## Service Tier Comparison — Zapier vs Make

### Features used in this build

| Feature | Zapier Free | Zapier Professional | Make Free | Make Core |
|---|---|---|---|---|
| Monthly task/operation limit | 100 tasks | 750–50,000 tasks | 1,000 ops | 10,000 ops |
| Multi-step automations | ❌ Two-step only | ✅ Unlimited steps | ✅ | ✅ |
| Conditional branching (Paths/Router) | ❌ | ✅ Paths included free* | ✅ Router module | ✅ |
| Loop/iterator module | ❌ | ✅ | ✅ | ✅ |
| Google Sheets integration | ✅ | ✅ | ✅ | ✅ |
| Google Calendar integration | ✅ | ✅ | ✅ | ✅ |
| Google Forms integration | ✅ | ✅ | ✅ | ✅ |
| Write-back to Google Sheets | ❌ (1 action only) | ✅ | ✅ | ✅ |
| Polling interval (how fast it checks) | 15 minutes | 2 minutes | 15 minutes | 15 minutes |
| Number of active automations | Unlimited | Unlimited | Unlimited | Unlimited |
| Scenario/Zap scheduling | ❌ | ✅ | ✅ | ✅ |
| Monthly price (billed annually) | Free | From $19.99/mo | Free | From $9/mo |

*Paths, Filter, and Formatter steps do not count toward Zapier task usage on any plan.

### Verdict

**For testing and low volume:** Zapier free plan works for 
initial testing but hits its limits quickly with a multi-step 
workflow like this one. The two-step restriction alone blocks 
the full build from running on free Zapier.

**For production use on a budget:** Make's free plan is the 
better choice. 1,000 operations per month, full multi-step 
support, Router module for branching, and Loop support — all 
on the free tier. This entire workflow can run on Make's free 
plan for a small organisation.

**Recommendation:** Rebuild this workflow on Make for 
production deployment. The logic is identical — only the 
interface changes.

---

## Proposed Make Migration

The Make equivalent of this workflow uses:
- **Watch Rows module** (Google Sheets) — trigger
- **Router module** — replaces Zapier Paths
- **Iterator module** — replaces Zapier Loop
- **Google Calendar modules** — create, update, delete event
- **Google Sheets modules** — lookup, update, search rows

A full Make scenario rebuild is proposed as the next version 
of this project.

---

## Built by

Ibiene West — HR & Business Operations Specialist | 
Workflow & Systems Designer  
[LinkedIn](https://linkedin.com/in/ibiene-west)

---

*This project was built as part of a personal initiative to 
solve a real problem observed during a volunteer engagement. 
It is offered as a free, open solution for community 
organisations and professional associations with limited 
technical budgets.*
