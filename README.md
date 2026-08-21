# Project Wave — Moderator Checklist & Admin Tracking App

## What is this project?

Project Wave is a data-collection moderator app + admin dashboards for tracking video collection progress toward a 100-hour target. Moderators use a Kilo-style checklist interface to work through structured workflows. Admins see real-time dashboards tracking sessions, sites, and progress.

**Core features**:

**Moderator app:**
- **Checklist interface**: Sidebar showing today's assigned sessions. Click a session to see detailed steps and reference materials (clips/GIFs).
- **Step tracking**: Each session has multiple steps. Moderators tick off as they go. The app tracks timestamps and completion status.
- **Reference media**: Embedded clips/GIFs for moderators to see correct form or expected outcomes.
- **Session logging**: Every step completion is logged to the backend (SessionLog table) for full audit trail.
- **Reminders & guidance**: Menu panel with key reminders and troubleshooting.

**Admin app:**
- **Progress dashboards**: Project leadership can see total hours collected, progress toward 100-hour target, active sites/moderators, completion rates, and trends.
- **Session tracking**: View all sessions, filter by site/moderator/date, see completion status and useable minutes.
- **Read-only**: Admins don't enter data; they monitor and report.

**Scope**: Mobile-first (designed for phones), but responsive on tablets and desktop.

**Backend**: SharePoint Lists + Power Automate flows. No approval workflow — moderators mark sessions complete, data flows to backend immediately.

**Build**: Single `.html` file (moderator + optional admin view) deployed via GitHub Pages. Vanilla JavaScript, Kilo-style UI patterns.

---

## What's in this folder?

| Path | What it is | For whom |
|---|---|---|
| `README.md` | This file — project overview | Anyone (before other docs) |
| `Dev_Notes.md` | Full working context: architecture decisions, to-do list, roadmap | Claude chats picking this up later |
| `Changelog.md` | Version-by-version history of changes and why | Anyone curious about project evolution |
| `Team_Handoff.md` | How to maintain/modify the app without the original owner | Human teammates inheriting this project |
| `Reference Files/` | Project Orbit architectural reference, design system, team guidelines | All developers |
| `app.html` | The complete app (not yet created) | Deployment & local development |
| `index.html` | Symbolic link or copy of `app.html` for GitHub Pages | Deployed version |

---

## How the app is used

### For Moderators
1. Open Wave on their phone/tablet at the collection site
2. Sign in with their username (validated against Moderators list)
3. See today's assigned sessions in the sidebar
4. Click a session to open the checklist
5. Reference embedded clips/GIFs as needed
6. Tick off each step as they complete it
7. When done, mark session complete + record useable minutes
8. All data syncs to SharePoint Lists in real-time
9. Move to next session or end day

**Typical flow**: Sign in → See sessions → Click session → Reference clips → Tick steps → Mark complete → Repeat

**Data**: Every step completion, session start/end, and useable minutes is logged to SessionLog (immutable audit trail).

### For Project Leadership (Admin view)
1. Sign in as admin
2. View progress dashboards:
   - Total hours collected vs. 100-hour target
   - Breakdown by session type (single vs. paired)
   - Active sites and moderators today
   - Completion rate and trending
   - Quality flags (abandoned sessions, very short videos, etc.)
3. Drill down: View all sessions, filter by site/date/moderator
4. No data entry — read-only monitoring and reporting

---

## How it fits into the bigger picture

**Upstream**: A scheduling system (separate from Wave) assigns moderators to sites and sessions. Wave reads the assigned sessions at login.

**Wave's role**: Ensures every step of the data-collection protocol is followed correctly. Provides reference materials (clips/GIFs). Logs every step completion for audit trail.

**Downstream**:
- **QA process** (separate): The data team reviews recorded video against the SessionLog. If video doesn't match checklist, it's flagged. Wave's audit trail makes QA much easier.
- **Analytics**: Project leadership uses admin dashboards to track progress toward 100-hour target, identify bottlenecks, and make staffing decisions.
- **Reporting**: HR/scheduling can see which moderators were productive, which sites had issues, etc.

**Integration points**:
- SharePoint Lists (backend): Read Sessions + Participants, Write SessionLog
- Power Automate flows: Sync session data to admin dashboards
- Reference media: Links to video clips/GIFs stored in SharePoint Document Library or shared drive

---

## Where to go from here

| Your goal | Read this | Why |
|---|---|---|
| **I'm setting up the project for the first time** | `Dev_Notes.md` → `ARCHITECTURE_REFERENCE.md` | Dev_Notes tells you what's set up and what's not; Architecture explains how to extend it |
| **I need to fix a bug or add a feature** | `Dev_Notes.md` → `ARCHITECTURE_REFERENCE.md` → then the code | Dev_Notes has recent context; Architecture covers patterns you'll need |
| **I'm taking over this project from someone else** | `Team_Handoff.md` | Written for you specifically — how to maintain it |
| **I need the full history of what changed** | `Changelog.md` | Each version documents why changes were made |
| **I'm writing code and need to understand the current state** | `Dev_Notes.md` → **look at `state` object** in app.html | Dev_Notes tells you what's shipped, what's in progress, and why certain decisions were made |
| **I want to know about architectural patterns (buttons, auth, sync, etc.)** | `Reference Files/ARCHITECTURE_REFERENCE.md` | Covers design system, components, state management, backend patterns — all inherited from Project Orbit |

---

## Quick start for a new session with Claude

If you're reopening this project in a new Claude chat:

1. **Read this README** (5 min) — you're here
2. **Read `Dev_Notes.md`** (10 min) — tells you what's built, what's pending, and recent decisions
3. **Read `Reference Files/ARCHITECTURE_REFERENCE.md`** (20 min) — if you're writing code, this is essential
4. **Then describe what you need** — Claude will know the context

---

## Version and status

**Current version**: [See `Changelog.md`]  
**Status**: [In progress / Shipped / Maintenance mode — documented in `Dev_Notes.md`]

For deployment instructions, see `Team_Handoff.md` → "Maintaining the application."

---

## Questions?

- **"How does X work?"** → Check `Dev_Notes.md` (design decisions) or `ARCHITECTURE_REFERENCE.md` (patterns)
- **"What changed in the last update?"** → Check `Changelog.md`
- **"How do I deploy this?"** → Check `Team_Handoff.md` → "Maintaining the application"
- **"How do I add a new backend table?"** → Check `Team_Handoff.md` → "Things that must not change" + `ARCHITECTURE_REFERENCE.md` section 8
