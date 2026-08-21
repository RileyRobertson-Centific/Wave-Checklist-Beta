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
├── app.html                           [Single-file app: HTML + CSS + JS]
├── index.html                         [Symbolic link to app.html for GitHub Pages]
│
└── Reference Files/
    ├── Team Shared Guidelines/
    │   ├── Project_Documentation_Instructions.md    [The four-doc standard we follow]
    │   ├── PROJECT_INSTRUCTIONS.md                  [Original project brief]
    │   ├── ARCHITECTURE_REFERENCE.md                [Patterns from Project Orbit]
    │   └── CENTIFIC_DESIGN_SYSTEM_01.md             [Centific visual brand]
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

---

## Current state

### Version
**0.1.MMDDYY** — Schema established, no code yet. (See `Changelog.md` for date-specific version numbers.)

### Shipped
- [ ] Frontend skeleton (HTML + CSS + JS from Kilo, customized for Wave)
  - [ ] Sidebar with session list
  - [ ] Session detail panel with step checklists
  - [ ] Reference media support (embedded clips/GIFs)
  - [ ] Menu panel (reminders, troubleshooting, settings)
  - [ ] Mobile-responsive layout
- [ ] Login screen (validate against Moderators SharePoint List)
- [ ] Cloud sync to backend (Power Automate flows for read/write)
  - [ ] Read: Sessions, Participants, Moderators, ReferenceMedia
  - [ ] Write: SessionLog (append-only status events)
- [ ] Theme toggle (dark/light, Kilo design)

### In Progress
- Setting up documentation (this is Session 1)

### Not yet started
- Actual session/step content (from workflow definition)
- Reminders & troubleshooting content
- Reference media (clips/GIFs) URLs and display
- Admin dashboard (project progress view)
- Power Automate flow creation (URLs to be wired)
- SharePoint Lists setup + schema validation

---

## To-do list

### Before first release (v1.0) — 3 week deadline

#### Week 1: Backend + Frontend skeleton

**Backend (SharePoint + Power Automate):**
- [ ] Create SharePoint Lists:
  - [ ] **Moderators** (id, username, email, site_assigned, active)
  - [ ] **Sites** (id, name, location, contact_info)
  - [ ] **Participants** (id, participant_id_code, demographics?, consent_status)
  - [ ] **Sessions** (id, moderator_id, participant_id, site_id, session_type [single/paired], scheduled_time, start_time, end_time, status [in_progress/completed/abandoned], useable_minutes, notes)
  - [ ] **SessionLog** (id, session_id, event [started/step1_done/.../completed], timestamp, moderator_id) — append-only
  - [ ] **ReferenceMedia** (id, session_type, step_number, file_url, description)
- [ ] Create Power Automate flows:
  - [ ] Read flows for each table (return all rows as JSON)
  - [ ] Write flow for SessionLog (append only, no updates)
  - [ ] Write flow for Sessions (update status, useable_minutes)
  - [ ] Get flow URLs, wire into `app.html` constants

**Frontend skeleton:**
- [ ] Create `app.html` from Kilo Task Tracker.html:
  - [ ] Copy HTML structure, CSS, Kilo design system
  - [ ] Add login screen + username validation against Moderators list
  - [ ] Replace hardcoded TASKS with SESSIONS array (dynamically loaded from backend)
  - [ ] Add cloud sync: read Sessions at login, write SessionLog on every step completion
  - [ ] Verify: JS parses, CSS braces balance, version bumped to 0.1.MMDDYY

#### Week 2: Content + admin basics

**Workflow definition:**
- [ ] Define session types and step checklist for single-participant sessions
- [ ] Define step checklist for paired-participant sessions
- [ ] Identify reference media needs (clips/GIFs moderators will need)
- [ ] Write key reminders for menu panel
- [ ] Write troubleshooting tips

**Frontend:**
- [ ] Integrate reference media URLs (clips/GIFs embeddable in steps)
- [ ] Add session type logic (different step sequences for single vs. paired)
- [ ] Build menu panel (reminders, troubleshooting table, settings)
- [ ] Add session completion form (ask for useable_minutes, notes, etc.)
- [ ] Test on iPhone + Android
- [ ] Version bump to 0.2.MMDDYY

**Admin dashboard (basic):**
- [ ] Create admin login view
- [ ] Build progress dashboard: total hours collected, target, active sites/moderators
- [ ] Add session list view with filter/sort

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

**Schema**:
```js
// Loaded from backend at login
SESSIONS = [
  { 
    id, moderator_id, participant_id, site_id, 
    session_type: "single|paired",
    status: "in_progress|completed|abandoned",
    steps: [
      { t: "instruction", meta?: "time/distance/etc", ref_media_id?: "url" },
      { ... }
    ]
  }
]
// SessionLog (immutable, append-only)
SessionLog: [ { session_id, event, timestamp }, ... ]
```

---

### Cloud-first sync (Orbit pattern)
**Decision**: On login, fetch all assigned sessions from SharePoint. On every step completion, append to SessionLog. Session completion updates Sessions table.

**Why**:
- Audit trail: SessionLog shows exactly what happened, when
- Admin dashboards can query progress in real-time
- Supports QA process (video team can cross-reference SessionLog)
- Scales to 200+ sessions

**Implication**: Moderators need internet connectivity (or app will cache and sync when connection returns).

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
- **Before proposing a big architectural change**, check `Dev_Notes.md` and `ARCHITECTURE_REFERENCE.md` first. Many patterns are locked in for good reason.
- **Ask clarifying questions early**. Ambiguity about workflow/inventory/admin-freshness can compound into rework.
- **Iterative builds over big-bang**. Ship small features, verify, iterate.
- **Show verification results**. Every build should end with: JS parses ✓, CSS balanced ✓, feature checks pass ✓, version bumped ✓.

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
