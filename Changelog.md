# Changelog — Project Wave

All notable changes to Project Wave are documented here. The version scheme is `MAJOR.MINOR.MMDDYY` (date is always MMDDYY of the build).

---

## 0.2.082426

**Status**: Client demo polish (everything since the last GitHub push of `0.1.082426`)

### Major additions
- `moderator` now sees **all seven** placeholder sessions (one per day, Aug 23–29), including Las Vegas copies. The two sessions scheduled before Aug 25 load completed; the rest are in progress. `admin` is unchanged. Legacy `riley.robertson` / `david.kang` / `wave.admin` still work.
- Button added to menu: **Reset app (demo only)**. Keeps the user signed in and restores placeholder checklist ticks, notes, and session status.

### Minor changes
- Menu: placeholder page links (**Checklist**, **Project Updates**, **Guidelines**, **Troubleshooting**), then Sign out, then Reset app, then Key reminders and a **Latest Update** card (Aug 24, 2026 — New SSDs). The in-menu troubleshooting table is gone. Links do not navigate yet; **Checklist** is marked current (pink text, pink left bar, no chevron).
- Username moved to the top header (left of theme toggle, user icon to the right). Menu no longer shows “Logged in as”.
- Menu version stamp is **mmWave v…** (was **Wave v…**).
- Third Session Checklist step is **Claim new Task in Feather**; **Participant orientation** follows it.
- Scenario titles include protocol labels: T1 Walking and tracking, A1 Walking with Pose Transitions, A2 Static presence and actions, M1 Mixed actions and gestures.
- Post-Session: first step is **Complete ingestion Hydra while still connected to X5 Wi-Fi**; metadata step is **Finish metadata entry in Feather and mark task as Completed**; last step is **Return and sign in equipment**. Notes are required (helper: “Please describe how the session went and note anything out of the ordinary”). Complete session stays disabled until every step is done and notes are filled.
- Working file renamed `app.html` → `demo.html`. `index.html` is a mirrored copy for GitHub Pages; keep both in sync until the demo is finalized, then work only on `index.html`.
- GitHub Pages is on (`main` / `/`). Live demo: https://RileyRobertson-Centific.github.io/Wave-Checklist-Beta/
- localStorage key is `wave_session_v26_` so cached checklists pick up the current schema.

---

## 0.1.082426

**Status**: Client demo refinements

### Major additions
- Added simple demo logins: `moderator` opens Riley’s four Redmond sessions; `admin` opens the read-only demo dashboard. Legacy `riley.robertson` / `david.kang` / `wave.admin` still work.
- Scenario sequencing requires all preceding Session Checklist tasks to be complete. T1 unlocks after orientation/intake, then A1, A2, and M1 in order.
- Scenario details have non-checkable **Before Recording** / **After Recording** headers, each with helper copy, around the first and last three checks.

### Minor changes
- **Device Prep** (was Equipment Pickup): kit pickup removed; added mic gain (+24 dB) and camera config (360 mode, 8K, 30 FPS).
- Third Session Checklist step is **Participant orientation**.
- Phase tabs always show a status dot: green check when complete, amber dash when incomplete. The selected tab uses a darker `--seg-active` background so it stands out from the track.
- Removed the small grey phase title that repeated the selected tab's name above each checklist; the spacing it occupied is preserved on the segmented control.
- Completed sessions in the sidebar use a full-width pink **completed** pill instead of phase bars.
- Desktop session list can collapse to a thin rail (choice remembered). Session-count pill removed from the top nav.
- Login field is empty. Demo accounts sit in an **amber** “Demo Version Logins” banner (`moderator` / `admin`), not a red error-style banner.
- Menu shows **Logged in as: [username]** and **Sign out** above Key reminders.
- Post-Session: no **Upload metadata**; metadata step is **Create Feather task and complete metadata entry**. Notes + Complete session always show on that tab; the button stays disabled until every step in the session is done. No useable-minutes field.
- Empty scenario pane: **Select a Scenario** / **Subtasks will appear here.**
- Hydra instruction block uses the same pink section-header + white body style as Before/After Recording, titled **Hydra App** (no card). Before/After Recording headers now render at the same weight as Hydra App (they were previously heavier because a nested `<strong>` bumped them past 700).
- Demo data: Sarah M. and Michael C. / Lisa R. load already completed.
- localStorage key is `wave_session_v19_` so cached checklists pick up the current schema.

---

## 0.1.082126

**Status**: Session Checklist protocol copy from the provided CSV

### Major additions
- **Session Checklist content**: Briefing tasks (greet/IDs, signed NDA, walkthrough), Hydra note, Hydra intake, then scenarios T1 → A1 → A2 → M1. Each scenario has the same six checks (orientation, placement, obstructions, video, audio, performance).

### Minor changes
- Title-only briefing/intake items sit on the main list and toggle in place. Scenario groups still open a detail pane. Hydra instructions sit in sequence, not above the whole list.
- localStorage key bumped to `wave_session_v11_` so cached placeholder checklists are replaced.

### Still not in this version
- Real reminders, reference media, live admin metrics, SharePoint lists, Power Automate URLs, `index.html` / GitHub Pages.

### Next steps
1. Equipment / post-session copy if those CSVs follow
2. Stand up SharePoint Lists + flows and wire the four URL constants
3. Clear `DEV_PREFILL_USERNAME` before any real-user test

---

## 0.1.082026

**Status**: Moderator UI skeleton, local placeholder data (Cowork spend-limit pickup + follow-on UI tweaks)

### Major additions
- **`app.html` exists**: Login, sidebar of assigned sessions, phased checklists, completion form, menu, theme toggle, localStorage. Power Automate URLs are empty (dormant-until-wired); the UI is testable without SharePoint.
- **Assignment scoping**: After login, a moderator only sees sessions whose `moderator_id` matches their profile.
- **Demo admin dashboard**: Sign in as `wave.admin` for a Kilo-style KPI view (hours toward 100h, sessions, sites, daily bars, moderator table). Numbers are stand-in data, not live SharePoint.
- **Nested task framework**: Session Checklist now supports derived-completion parent tasks plus title-only and detailed children. Desktop uses parent/child panes; mobile drills into children with slide navigation and a back control.
- **Placeholder roster** (dev only): participants from `fake_contacts2.csv`, labeled as first name + last initial; sites Redmond (Riley) and Las Vegas (David); sessions spread across several days in late August 2026.

### Minor changes
- On-screen product name is **mmWave** (nav and login brand) while the project is still Wave in docs/folder names.
- Sidebar header: **Your Assigned Sessions**.
- Dev convenience: login field pre-filled with `riley.robertson` (`DEV_PREFILL_USERNAME` in `app.html` — remove before ship).
- **Equipment Pickup** checklist rewritten (5 steps): pickup from equipment team, inspect all devices (with **Condition notes** field), Insta360 lens inspect with “do not proceed if cracked or occluded” banner, empty/formatted card, battery ≥80%. Tripod/mount step removed.
- **Session Checklist** rewritten around Hydra: greet/confirm IDs, **Confirm NDA and Consent form**, explain + rehearse, then record/verify scenarios M1, A1, A2, T1. A Hydra instruction block sits between briefing and the scenario ticks (not itself a step).
- **Post-Session** checklist rewritten: Hydra ingestion, metadata in the MacBook local app, upload TAR, upload metadata, inspect/clean/repack, return equipment.
- Hydra scenario parents currently contain reusable title-only and detailed child placeholders pending final task copy.
- Completed tasks use muted color only (no strikethrough).
- Session Checklist UI copy no longer says “parent” or “child”; those remain internal task-type names only.

### Still not in this version
- Real reminders, reference media, admin dashboard, SharePoint lists, Power Automate URLs, `index.html` / GitHub Pages.

### Next steps
1. Replace placeholder steps/reminders with the real protocol
2. Stand up SharePoint Lists + flows and wire the four URL constants
3. Admin dashboard
4. Clear `DEV_PREFILL_USERNAME` before any real-user test

---

## 0.1.081726

**Status**: Foundation & architecture finalization

### Major additions
- **Documentation structure**: Established the four-document standard (`README.md`, `Dev_Notes.md`, `Changelog.md`, `Team_Handoff.md`)
- **Architecture finalized**: 
  - Frontend: Kilo-style UI (familiar to moderators)
  - Backend: SharePoint Lists + Power Automate (no 256-row ceiling, supports 200+ sessions)
  - Hybrid approach: Kilo's UX + Orbit's backend patterns
- **Reference materials**: Organized for both Kilo (UI) and Orbit (backend patterns)

### Key decisions locked in
- **SharePoint Lists backend** (not Excel, not local-only): Handles scale, proper types, no pagination ceiling
- **Session + SessionLog model**: Sessions table + immutable SessionLog for audit trail
- **Kilo design system**: Pink accent (#EF43B3), dark theme, sidebar + main panel
- **Cloud-first sync**: Every step logs to SessionLog in real-time
- **No approval workflow**: Moderators mark complete, data flows to backend immediately
- **Reference media support**: Steps can embed clips/GIFs from SharePoint Document Library
- **Mobile-first**: Sidebar → overlay on mobile
- **Login required**: Moderators validate against Moderators list

### Scope clarification
- ~200+ sessions needed in 3 weeks to hit 100-hour target
- Two session types: single-participant (5-7 min useable) + paired (15-21 min useable)
- 10-15 collection sites estimated
- Admin dashboards for project leadership (progress toward 100 hours, active sites/moderators)
- No mid-session approval (QA separate)

### Deferred items (Week 1 blockers)
- Exact workflow definition (session steps for single + paired)
- Reference media specs (what clips/GIFs needed, where stored)
- Key reminders + troubleshooting content
- SharePoint Lists schema validation
- Power Automate flow URLs

### Next steps
1. Finalize workflow definition (steps for single/paired sessions)
2. Set up SharePoint Lists + Power Automate flows (Week 1)
3. Build `app.html` skeleton with cloud sync (Week 1)
4. Integrate content + reference media (Week 2)
5. Build admin dashboard (Week 2)
6. Full testing + deployment (Week 3)

---

## How to add entries to this file

1. **New version**: Add a new top-level heading with the version number and a "Status" line if it's a milestone.
2. **Sections**: Use headings like "Major additions," "Breaking changes," "Bug fixes," "Minor changes," "Deferred items."
3. **Format**: Keep each bullet short (1-2 lines). Link to related docs if needed.
4. **Rationale**: Include the *why* for significant changes, not just the what.
5. **Revert note**: If a change is made and then reverted within the same version before shipping, remove the mention from Changelog but add a note to `Dev_Notes.md` (see Project_Documentation_Instructions.md).

---

## Release checklist

Before bumping the version number:

- [ ] All intended features / fixes are implemented
- [ ] `README.md` is still accurate (or has been updated if scope changed)
- [ ] `Dev_Notes.md` is current (to-do list updated, deferred work documented)
- [ ] JS parses cleanly
- [ ] CSS braces are balanced
- [ ] Feature regex spot-checks pass
- [ ] New Changelog entry is complete with reasoning

---

## Version numbering scheme

- **MAJOR**: Reserved for fundamental architecture or user-facing release milestones. Bump only with explicit sign-off from Riley.
- **MINOR**: Reserved for feature groups (login, checklist, inventory, admin dashboard). Bump when a major subsystem ships.
- **MMDDYY**: Date of the build. Changes on every code change, automatically bumped by the verification script.

**Example progression**:
- `0.1.081726` → foundation setup
- `0.1.081927` → login flow + welcome modal
- `0.1.082426` → first client-demo landing (last GitHub push of 0.1)
- `0.2.082426` → client demo polish (seven-session moderator roster, menu, reset)
- `1.0.082315` → first release (explicit milestone)

The date segment gives you a glance at how fresh the code is.
