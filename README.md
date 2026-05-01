---

## Planning Sheet — Status Logic

The **Done?** column is the single source of truth that drives 
all automation. Each status maps to a specific action:

| Status | Meaning | Automation triggered |
|---|---|---|
| Planning | Session being drafted | None |
| Approved | Session confirmed | Create calendar event + form |
| Details Edited | Previously approved, details changed | Update calendar event — resets to Approved |
| Planning On Hold | Paused before approval — event never created | None |
| Cancelled | Session called off | Delete calendar event + notify attendees |
| Done | Session completed | Archive |

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
| Form Link | Written by Zapier after form creation |
| Form Response Sheet Link | Written by Zapier after sheet creation |

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
plan for a small organisation like SHRPAN.

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
