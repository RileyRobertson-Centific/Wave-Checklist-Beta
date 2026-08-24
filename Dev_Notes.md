# Dev_Notes — Project Wave Working Context

This document is written for a Claude chat picking this project back up. See `README.md` for the big picture.

**Architecture note**: Project Wave has:
- **Frontend**: Kilo-style checklist UI (sidebar + main panel, mobile-first, Kilo design system)
- **Backend**: SharePoint Lists + Power Automate flows (not local-only, not Excel)
- **Scope**: Moderator app + optional admin dashboards. No mid-session approval workflow.
- **Goal**: Track every step of data-collection for 200+ sessions in 3 weeks, heading toward 100-hour video target.

---

## File & folder inventory

```
Project Wave Checklist App Test/
├── README.md                          [Front door — orientation, where to go]
├── Dev_Notes.md                       [This file — working context]
├── Changelog.md                       [Version history and why]
├── Team_Handoff.md                    [For humans maintaining the project]
│
├── app.html                           [Single-file app: HTML + CSS + JS — this is the working product]
├── index.html                         [Not created yet — copy of app.html for GitHub Pages when we deploy]
├── Session Checklist.xlsx             [Protocol source for Session Checklist copy]
│
└── Reference Files/
    ├── Team Shared Guidelines/
    │   ├── Project_Documentation_Instructions.md    [The four-doc standard we follow]
    │   ├── PROJECT_INSTRUCTIONS.md                  [Original project brief]
    │   ├── ARCHITECTURE_REFERENCE.md                [Patterns from Project Orbit]
    │   ├── CENTIFIC_DESIGN_SYSTEM_01.md             [Centific visual brand]
    │   └── App_Build_Workflow_and_Replication_Guide.md  [General build/deploy/Claude-workflow guide — backend section assumes Excel, see note below]
    │
    └── Project Kilo Files/
        ├── centific-kilo-design-system.md
        ├── index.html                              [Reference implementation]
        └── Kilo Task Tracker.html                  [Another reference app]
```

### What each reference file is for

- **Kilo Task Tracker.html** — UI reference. Copy HTML structure, CSS, and Kilo design system. Modify the task/step data structure for Wave's sessions/steps.
- **centific-kilo-design-system.md** — Visual tokens: colors (#EF43B3 pink accent, dark mode), spacing, typography. Use exactly.
- **ARCHITECTURE_REFERENCE.md** — Project Orbit's backend patterns. Wave uses similar concepts (cloud sync, status logging, soft deletes) but with SharePoint Lists instead of Excel. Reference sections 7-9 for state management, Power Automate + backend patterns, and refresh architecture.
- **PROJECT_INSTRUCTIONS.md** — Original brief. Updated to clarify Wave has backend (SharePoint Lists + Power Automate), unlike local-only Kilo.
- **App_Build_Workflow_and_Replication_Guide.md** — General Centific playbook for these single-file apps: tech stack, deploy, Claude workflow, pre-ship QA, replication steps. **Known divergence**: its section 4-5 (the database and connectors) is written entirely around Excel tables on SharePoint (Excel Online actions, all-Text columns, 256-row pagination cap). Wave intentionally uses SharePoint Lists instead — see "SharePoint Lists + Power Automate backend" under Key decisions below for why. When following this guide's replication steps for Wave, substitute: "Get items" (SharePoint) for "List rows" (Excel), List column filters for Excel column filters, and native SharePoint column types where the guide says Text-only. Everything else in the guide (single-file HTML, version stamping, pre-ship QA checklist, dormant-until-wired URLs, deployment) applies to Wave as written.

---

## Current state

### Version
**0.1.082426** (`APP_VERSION` in `app.html`). See `Changelog.md`.

### Shipped (local-only skeleton)
- [x] Frontend skeleton in `app.html` (Kilo tokens, login, app shell, menu, theme toggle, mobile layout)
- [x] Sidebar: **Your Assigned Sessions**, date/time, type chip, participant short name, phase dots; completed sessions show a full-width pink **completed** pill. Desktop collapse-to-rail control (remembered).
- [x] Session detail: three phases — **Device Prep**, **Session Checklist**, **Post-Session**. Notes + Complete session always visible on Post-Session; button disabled until `sessionComplete()`.
- [x] Nested Session Checklist: protocol from `Session Checklist.xlsx`; title-only items on the main list; sequentially unlocked T1 → A1 → A2 → M1; six detailed checks under **Before Recording** / **After Recording** (headers + helper notes, not steps); desktop two-pane view and mobile drill-in
- [x] Login against placeholder Moderators. Client demo: `moderator` / `admin` (amber **Demo Version Logins** banner). No username prefill. Legacy: `riley.robertson`, `david.kang`, `wave.admin`
- [x] Moderators only load/see sessions assigned to them (`sessionAssignedTo`). `moderator` sees the same four Redmond sessions as `riley.robertson`; Sarah M. and Michael C. / Lisa R. are pre-completed
- [x] localStorage per username (`wave_session_v19_<username>`)
- [x] Cloud sync **stubs**: `MODERATORS_READ_URL`, `SESSIONS_READ_URL`, `SESSIONS_WRITE_URL`, `SESSIONLOG_WRITE_URL` are `''`; reads fall back to placeholders; writes log to console
- [x] Menu: **Logged in as: [username]**, Sign out, then placeholder reminders/troubleshooting
- [ ] Theme toggle is in the nav (works); not a separate “settings” panel
- [ ] Reference media: steps may have `ref_media_id`; UI shows “not yet configured”
- [ ] Menu reminders/troubleshooting are still placeholder copy

### In Progress
- Iterating the moderator UI for a client demo (Riley, this Cursor session). Backend not started.

### Not yet started
- Real reminders & troubleshooting
- Reference media URLs
- Live admin metrics from SharePoint (UI exists; data is demo)
- Power Automate flows (URLs still empty)
- SharePoint Lists
- `index.html` / GitHub Pages deploy
- Clearing leftover demo logins / placeholder roster before real users

### How to run locally
Open `app.html` in a browser. Amber banner: `moderator` (checklist) and `admin` (dashboard). If the list looks stale, sign out or use a private window — storage key is currently `wave_session_v19_`.

---

## To-do list

### Before first release (v1.0) — 3 week deadline

#### Week 1: Backend + Frontend skeleton

**Backend (SharePoint + Power Automate):**
- [ ] Create SharePoint Lists:
  - [ ] **Moderators** (id, username, email, site_assigned, active)
  - [ ] **Sites** (id, name, location, contact_info)
  - [ ] **Participants** (id, participant_id_code, demographics?, consent_status)
  - [ ] **Sessions** (id, moderator_id, participant_id, site_id, session_type [single/paired], scheduled_time, start_time, end_time, status [in_progress/completed/abandoned], notes; useable_minutes optional — not collected in the current UI)
  - [ ] **SessionLog** (id, session_id, event [started/step1_done/.../completed], timestamp, moderator_id) — append-only
  - [ ] **ReferenceMedia** (id, session_type, step_number, file_url, description)
- [ ] Create Power Automate flows:
  - [ ] Read flows for each table (return all rows as JSON)
  - [ ] Write flow for SessionLog (append only, no updates)
  - [ ] Write flow for Sessions (update status, useable_minutes)
  - [ ] Get flow URLs, wire into `app.html` constants

**Frontend skeleton:**
- [x] Create `app.html` from Kilo patterns:
  - [x] HTML structure, CSS, Kilo design system
  - [x] Login screen + username validation against Moderators list (placeholder list until the read URL is wired)
  - [x] Sessions loaded from placeholder data (or backend when wired), filtered to the signed-in moderator
  - [x] Cloud sync stubs: read Sessions at login, write SessionLog on step completion (no-ops while URLs are empty)
  - [ ] Verify: JS parses, CSS braces balance — do this before calling a build “done”
  - [x] Version stamp `0.1.082026`

#### Week 2: Content + admin basics

**Workflow definition:**
- [x] Device Prep checklist (6 steps + condition notes) — landed in `app.html`
- [x] Session Checklist (greet → NDA/consent → explain/rehearse → Hydra M1 / A1 / A2 / T1); greet/explain wording still differs by session type
- [x] Post-Session checklist (Hydra ingestion, Feather task + metadata, TAR upload, pack/return)
- [ ] Identify reference media needs (clips/GIFs moderators will need)
- [ ] Write key reminders for menu panel
- [ ] Write troubleshooting tips

**Frontend:**
- [ ] Integrate reference media URLs (clips/GIFs embeddable in steps)
- [ ] Add session type logic (different step sequences for single vs. paired)
- [x] Build menu panel (reminders, troubleshooting table, settings)
- [x] Add session completion form (notes only; Complete session gated on all steps)
- [ ] Test on iPhone + Android
- [ ] Version bump to 0.2.MMDDYY

**Admin dashboard (basic):**
- [x] Create admin login view (`admin` / `wave.admin`, `role: admin`)
- [x] Build progress dashboard: hours vs 100h, sessions, sites/moderators (demo stats in `placeholderAdminStats()`)
- [ ] Add session list view with filter/sort
- [ ] Replace demo stats with live SessionLog / Sessions queries

#### Week 3: Polish + final testing + deployment

- [ ] Full end-to-end testing: login → see sessions → complete session → verify in admin dashboard
- [ ] Fix bugs from testing
- [ ] Populate all reference media
- [ ] Brief moderator team on workflow
- [ ] Deploy to GitHub Pages
- [ ] Version bump to 1.0.MMDDYY

### Post-release (if time/scope allows)
- [ ] More detailed analytics in admin dashboard (completion by site, by moderator, etc.)
- [ ] Export session data to CSV
- [ ] Session timer
- [ ] Offline mode (if needed)

---

## Roadmap of deferred work

### Critical questions to answer before Week 1 starts

1. **Workflow definition** (affects step checklist):
   - What are the exact steps for a single-participant session? (e.g., 1. Greet participant 2. Place camera 3. Record 5-7 min video 4. Verify audio/video quality 5. Review with participant 6. End session)
   - What are the steps for paired-participant sessions? (any differences?)
   - Are there warnings/special instructions? (e.g., "Do not reposition camera between sessions")
   - Are there conditional steps? (e.g., "If video quality < acceptable, re-record")

2. **Reference media** (for embedded clips/GIFs):
   - What reference clips/GIFs will moderators need? (e.g., "Here's correct camera placement", "Here's what acceptable video quality looks like")
   - Where will these be stored? (SharePoint Document Library, OneDrive, shared drive?)
   - Who will upload/maintain them? (video team, PM, or Wave admin?)
   - Will media be per-step or per-session-type?

3. **Key reminders & troubleshooting**:
   - What are the 5-10 most important things moderators should never forget?
   - What are the 5-10 most common issues they'll encounter? (and how to fix?)

4. **Participant tracking**:
   - Will participant_id be auto-generated or entered by moderator?
   - Do we need to validate that same participant doesn't do too many sessions? (or no constraints for now?)

5. **Moderator workflow clarification**:
   - Should Sessions be pre-populated in the app, or created by moderator at start of day?
   - Will Wave integrate with a scheduling system, or is scheduling out-of-band?

---

## Key decisions & reasoning

### Single-file HTML + vanilla JS (Kilo pattern)
**Decision**: Build as one `.html` file with inline CSS and JavaScript. No framework, no build pipeline.

**Why**: 
- Deployment is trivial (rename to `index.html`, push to GitHub Pages)
- No dependencies, no build step
- Fast loading on mobile devices in field
- Proven by Kilo for moderator apps

**Trade-off**: Larger single file (~8-12K lines expected). Mitigated by aggressive section comments and clear naming.

---

### SharePoint Lists + Power Automate backend (hybrid Orbit + Kilo)
**Decision**: Data lives in SharePoint Lists (not Excel, not localStorage). Power Automate flows handle read/write.

**Why**:
- SharePoint Lists have NO 256-row pagination ceiling (critical for 200+ sessions)
- Proper column types (Date, Number, Lookup) — easier than Excel all-Text approach
- Power Automate integrates seamlessly with Microsoft 365
- Admin can view/query data natively in SharePoint
- Scales better than Excel as data grows

**Trade-off**: Slightly more complex than local-only. Session logging happens in real-time (important for audit trail).

**Key pattern**: SessionLog is append-only (immutable). Sessions table is updated for completion status only. No soft-delete needed (no concurrent write collisions on same session).

---

### Session + Step structure (Kilo pattern, adapted for Sessions)
**Decision**: Each session has multiple required steps. Steps are ticked as completed. Progress tracked in real-time.

**Why**:
- Proven UX from Kilo (familiar to moderators)
- Sidebar overview of all sessions + progress bar
- Main panel shows current session's steps
- Easy to reference media (clips/GIFs) alongside steps

**Schema (as implemented in placeholder sessions):**
```js
{
  session_id, moderator_id, site_id,
  session_type: "single|paired",
  participants: ["Full Name", ...],  // UI shows first name + last initial via sessionLabel()
  scheduled_date: "YYYY-MM-DD",
  scheduled_time: "9:00 AM",
  scheduled_at: "YYYY-MM-DDTHH:MM:SS", // sort key
  status: "in_progress|completed",
  useable_minutes, notes,  // UI no longer collects useable_minutes; field remains on the session object for a future backend
  phases: [{
    key, title,
    steps: [
      { kind: "info", title, t },
      { kind: "child-title", key, t, done? }, // top-level; toggles on the main list
      {
        kind: "parent", key, t,
        children: [
          { kind: "child-title", key, t, done? },
          { kind: "child-detailed", key, t, description, done? }
        ]
      }
    ]
  }]
}
```
Flat phases (Device Prep and Post-Session) still use their existing simple step objects. SessionLog events (when wired): `step_completed` / `step_reopened` / `session_completed`; nested child events also include `parent_key`, `parent`, and `step_key`.

---

### Three-level task behavior (Session Checklist redesign)
**Decision**: The Session Checklist supports three task types. These names are for developers only and never appear in the UI:
- **Parent** (`kind: "parent"`): opens its children and cannot be toggled directly; complete only when every child is complete.
- **Child title-only** (`kind: "child-title"`): manually toggled, title only.
- **Child detailed** (`kind: "child-detailed"`): manually toggled, title plus smaller descriptive copy. Completion uses the tertiary color on both title and description; no strikethrough.
- **Section** (`kind: "section"`): visual header only (`before_recording`, `after_recording`). Optional `note` is helper copy under the header, not a step and not in progress totals.

**Responsive behavior**:
- Desktop: parent list on the left; selected child list in a pane to the right.
- Mobile: parent rows have right arrows; tapping drills into a full child list that slides in from the right, with an in-content back button.

**Phase header area**: The phase name is not repeated above the list — the segmented control is the only label. `.seg-control` carries `margin-bottom:43px`, which reproduces the old spacing (20px control margin + 11px label + 12px label margin); change it if the control or list padding changes. The selected segment uses `--seg-active`, a per-theme value deliberately darker than `--card-bg` so the selection reads at a glance.

**Current Session Checklist structure**: Top-level title-only — greet/IDs, signed NDA, participant orientation, then Hydra intake after the Hydra note. Scenario groups T1, A1, A2, M1 unlock sequentially. Each has six detailed checks: three under **Before Recording** (orientation/placement/obstructions) and three under **After Recording** (video, audio, performance). Section headers are not steps. Helper notes under the headers (full-contrast body text): Before Recording — do the checks for every camera placement, then mark them finished when the whole recording is done; After Recording — use Hydra preview for the three checks and re-record if anything fails. The Hydra instruction on the main list uses the same pink header + white body style, titled **Hydra App**. Section titles match Hydra App exactly: 12.5px, weight 700, uppercase, pink. Keep the title in a `span` (`.child-section-title`), not a `strong` — a nested `strong` resolves `bolder` against the 700 parent and renders at 900, which is what made the scenario headers look heavier than Hydra. Empty right pane copy: “Select a Scenario” / “Subtasks will appear here.” Same copy for single and paired until we get type-specific wording. Do not change `key` fields casually—they will be written to SessionLog.

---

### Cloud-first sync (Orbit pattern)
**Decision**: On login, fetch all assigned sessions from SharePoint. On every step completion, append to SessionLog. Session completion updates Sessions table.

**Why**:
- Audit trail: SessionLog shows exactly what happened, when
- Admin dashboards can query progress in real-time
- Supports QA process (video team can cross-reference SessionLog)
- Scales to 200+ sessions

**Implication**: Moderators need internet connectivity (or the app will cache and sync when connection returns — offline retry is not implemented).

---

### Moderators only see assigned sessions
**Decision**: Filter at login (and again in `allSessions()`) so `moderator_id` on the session must match the signed-in profile. Do not show a global session list.

**Why**: Field moderators should not see another site’s roster. Admins (`role: admin`) skip the session list and open the dashboard instead.

---

### Demo admin dashboard (Kilo KPI pattern)
**Decision**: Admin view is a read-only dashboard in the same `app.html`. KPIs follow Kilo (large numbers, site/type/day bars, moderator table). No Chart.js — CSS bars only, to stay zero-dependency.

**Why**: Leadership needs a progress surface now for design review. Live SessionLog queries come later; `placeholderAdminStats()` is explicitly labeled demo data.

---

### On-screen name is mmWave (for now)
**Decision**: Nav and login brand say `mmWave`. Folder and docs still say Project Wave.

**Why**: Riley asked for this display name during skeleton work. Treat as temporary until a final product name is locked.

---

### Placeholder data for UI development
**Decision**: Until flow URLs are non-empty, use `PLACEHOLDER_MODERATORS`, `PLACEHOLDER_CONTACTS`, and `placeholderSessions()`.

**Current stand-ins:**
| Username | Site | Sessions |
|---|---|---|
| `moderator` | Demo (Riley’s Redmond roster) | Same four Redmond sessions as `riley.robertson`; first two pre-completed |
| `admin` | All | Demo dashboard (not a session list) |
| `riley.robertson` | Redmond | 4 sessions, Aug 20–25 2026 (first two pre-completed) |
| `david.kang` | Las Vegas | 3 sessions, Aug 20–26 2026 |
| `wave.admin` | All | Demo dashboard (same as `admin`) |

Participants are the ten names from Riley’s `fake_contacts2.csv`, shown as first name + last initial (e.g. `Sarah M.`, paired `Michael C. / Lisa R.`).

---

### No mid-session approval (simpler than Orbit)
**Decision**: Moderators mark session complete, data goes to backend immediately. No "pending review" status.

**Why**:
- Simpler workflow (1 less state machine)
- QA happens separately (out of app)
- Keeps moderator app focused on data collection

---

### Reference media embedded in steps
**Decision**: Each step can reference a clip/GIF (stored in SharePoint, linked by URL).

**Why**:
- Moderators see "correct form" as they work
- Reduces misinterpretation of instructions
- Easy to update/fix without rebuilding app

**Implication**: ReferenceMedia table needed. URLs must be accessible from collection sites (consider permissions).

---

## Working conventions

### How to collaborate with me (Riley)
- **Keep the four docs current with the code**, per `Reference Files/Team Shared Guidelines/Project_Documentation_Instructions.md`. `Dev_Notes.md` updates with every change; `Changelog.md` for notable versions; `README.md` only when the big picture or folder layout changes; `Team_Handoff.md` when operating procedure or unfinished work changes. Do not skip docs until the end of a session.
- **Ask before bumping MAJOR.MINOR** (date segment in `APP_VERSION` can follow the build date). See Changelog versioning notes.
- **Before proposing a big architectural change**, check `Dev_Notes.md` and `ARCHITECTURE_REFERENCE.md` first. Many patterns are locked in for good reason.
- **Ask clarifying questions early**. Ambiguity about workflow/inventory/admin-freshness can compound into rework.
- **Iterative builds over big-bang**. Ship small features, verify, iterate.
- **Show verification results**. Every build should end with: JS parses ✓, CSS balanced ✓, feature checks pass ✓, version considered ✓.

### Code structure expectations
- **High comment density**. Document WHY, not just WHAT. Especially browser quirks, mobile-specific decisions, trade-offs.
- **Section comments for long files**. Use visual banners to mark major sections (cloud sync, login, inventory, etc.).
- **Diagnostic console logs**. `console.log('[App] ...')` at decision points so bug reports can be self-diagnosed.

### Manual edits
If you make a direct edit to `app.html` (not through Claude), **mention it next time** — Claude has no way to know about it otherwise. This prevents Claude from undoing your changes or conflicting with them.

---

## Session-to-session handoff checklist

When closing a session, if you've made progress:

- [ ] Update `Dev_Notes.md` → "Current state" section (what shipped, what's in progress)
- [ ] Update `Dev_Notes.md` → "To-do list" (check off completed items)
- [ ] Create or update `Changelog.md` with the version number and changes
- [ ] If you changed the app.html significantly, add a note to `Dev_Notes.md` → "Recent decisions" (context for next session)

This way, the next session can pick up without re-explaining everything.

---

## References

- **For architectural patterns**: `Reference Files/ARCHITECTURE_REFERENCE.md` (sections 1–4 are essential before writing code)
- **For the original spec**: `Reference Files/PROJECT_INSTRUCTIONS.md`
- **For deployment**: See `Team_Handoff.md`
- **For version history**: `Changelog.md`
