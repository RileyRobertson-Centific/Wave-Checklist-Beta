# Team_Handoff — Project Wave Maintenance Guide

**This document is written for humans**, not AI. If you're using Claude to make changes to this project, point Claude at `Dev_Notes.md` first — it has the full technical context. Then return here to understand the operational side.

---

## Overview

You're inheriting the Wave / **mmWave** moderator checklist (see `README.md` for what the product is). Display name in the app is **mmWave** for now.

**Architecture**:
- **Frontend**: Single `app.html` (Kilo-style UI). `index.html` for GitHub Pages is not created yet.
- **Backend (planned)**: SharePoint Lists + Power Automate. Flow URL constants in `app.html` are empty; the current build uses placeholder data.
- **Moderators**: Sign in, see **only assigned sessions**, work through step checklists, mark complete. Client demo login: `moderator`.
- **Admins**: Sign in as `admin` for a read-only demo dashboard (not live SharePoint).

**Key fact**: Hybrid of Kilo's UI and Orbit's backend patterns. See `Dev_Notes.md` → "Key decisions & reasoning".

---

## Maintaining the application

### How to make a change

#### If you're not technical
1. Open a new Claude chat and paste this entire Project Wave folder as a project context
2. Describe what you want to change (e.g., "Add a new checkbox to the checklist," "Change the accent color to #00F0FF")
3. Claude will propose changes and explain them
4. When Claude is done, ask Claude to show you the changes before deploying
5. **Important**: Tell Claude about any manual edits you make to `app.html` — Claude won't know about them otherwise

#### If you're technical (direct editing)
1. Read `Reference Files/ARCHITECTURE_REFERENCE.md` sections 1–4 first (essential background)
2. Edit `app.html` directly in a text editor
3. After any changes:
   - Verify JS parses: `node -c app.html` or extract `<script>` and run `new Function(code)`
   - Verify CSS braces are balanced: count `{` and `}` in `<style>` — must end at 0
   - Bump the version number: change `APP_VERSION = 'X.Y.MMDDYY'` to today's date
4. Test locally in a browser before committing
5. **Commit message**: Describe WHAT changed and WHY (not just "fix" or "update")
6. Tell Riley about the change if he's not the one who made it (so he can update `Dev_Notes.md`)

#### If you're working with Claude
1. Follow the "not technical" path above
2. Provide Claude with the checklist: "Verify JS parses, CSS braces balance, version bumped, and show me the changes before we deploy"

### How to deploy

1. Verify the app works locally (open `app.html` in a browser, test on iOS and Android if possible)
2. Copy `app.html` to `index.html` (GitHub Pages serves `index.html` by default)
3. Commit both files to the `main` branch of the GitHub repo (location TBD by Riley)
4. GitHub Pages will auto-deploy within ~30 seconds
5. Test the live version at the GitHub Pages URL (also TBD by Riley)

#### Recurring maintenance tasks

##### Local testing (until SharePoint is live)
- Open `app.html` in a browser. Demo logins (amber **Demo Version Logins** banner): `moderator` (checklist — Riley’s four Redmond sessions; Sarah M. and Michael C. / Lisa R. already completed), `admin` (dashboard). Legacy stand-ins `riley.robertson` and `david.kang` still work. Each moderator should only see their own sessions. If the list looks stale, sign out or use a private window (`wave_session_v18_`).

##### App content & workflow
- **Update session steps**: If the data-collection protocol changes, update the session steps in the `SESSIONS` array and redeploy `app.html`. Test thoroughly before deploying.
- **Update reminders**: If safety guidelines or best practices change, update the "Key Reminders" section in the menu panel and redeploy. Menu order is: logged-in username, Sign out, then reminders/troubleshooting.
- **Update troubleshooting**: If new issues come up, add them to the troubleshooting table and redeploy.
- **Reference media updates**: If clips/GIFs need to be updated or added, upload to SharePoint Document Library and update ReferenceMedia list with new URLs.

#### SharePoint Lists maintenance
- **Moderators list**: Add/remove moderators as team composition changes. Active field controls login access.
- **Sites list**: Update site info (contact, location) as needed.
- **SessionLog backups**: SharePoint handles versioning automatically, but regularly export SessionLog for your records (valuable for QA review).
- **Participant list**: Monitor for duplicates or data quality issues. Consider adding validation on moderator side if problems arise.
- **Power Automate flow monitoring**: Check PA admin dashboard monthly for quota usage. If hitting limits, may need to archive old sessions.

#### Version control
- **Git history**: Keep `app.html` in Git. Easy to roll back if a change breaks something.
- **Version number bumping**: Update `APP_VERSION` constant (format: `MAJOR.MINOR.MMDDYY`) with every code change.

#### Users & access
- **Login credentials**: Username only, no passwords. Production: Moderators SharePoint list + `active=true`. Current skeleton: `PLACEHOLDER_MODERATORS` in `app.html`.
- **Admin access**: Usernames `admin` and `wave.admin` (`role: admin` in `PLACEHOLDER_MODERATORS`). Dashboard is demo data. Production: add admins to the Moderators list with an admin role — not implemented against SharePoint yet.

#### Monitoring
- **User reports**: If a moderator reports an issue:
  1. Ask them to open browser console (`F12`) and take a screenshot of any red errors
  2. Ask what device/browser (iPhone Safari vs. Android Chrome behavior differs)
  3. Check Power Automate flow status (sometimes flows fail silently)
  4. Ask them to hard-refresh (`Ctrl+Shift+R` or `Cmd+Shift+R`)
- **Session logging**: Check admin dashboard to verify sessions are being logged. If sessions aren't appearing, flow may have failed.
- **Performance**: If sessions load slowly, check SessionLog size (if >5000 rows, consider archiving old sessions).

---

## Things that must not change without care

### SessionLog data model
The SessionLog is append-only and immutable. Every step completion creates a new row (never update/delete existing rows). QA and reporting depend on this audit trail.

**If you need to change the event types** (e.g., add a new "paused" event):
1. Add the new event to the code
2. Test that it's written correctly to SessionLog
3. QA will need to understand the new event type when reviewing

**Never**:
- Delete SessionLog rows
- Update SessionLog rows
- Change the meaning of existing event types (e.g., "completed" must always mean the same thing)

### Power Automate flow URLs
These URLs are hardcoded in `app.html`. If a flow is regenerated, the URL changes and the app silently stops syncing.

**If a flow URL changes**:
1. Get the new URL from Power Automate
2. Find the matching constant in `app.html` (e.g., `SESSIONLOG_PA_WRITE_URL`)
3. Update it
4. Bump version number
5. Redeploy
6. Test immediately (check admin dashboard to verify data syncs)

**Critical flows** (if any fail, entire app is broken):
- Sessions read (loads assigned sessions)
- SessionLog write (logs every step completion)

### SharePoint List schema
The app expects exact column names and types in SharePoint Lists. If a column is renamed or deleted, the app breaks silently.

**Before renaming a column**:
1. Update the app code to read/write the new column name
2. Update Power Automate flows
3. Test end-to-end
4. Only then rename in SharePoint

**Critical columns** (app depends on these):
- Sessions: `id`, `moderator_id`, `session_type`, `status`, `steps` (JSON)
- SessionLog: `session_id`, `event`, `timestamp`
- Moderators: `username`

### Nested task keys and completion
Session Checklist tasks use stable `key` fields. Nested completion events will include both `parent_key` and `step_key`; changing keys after production data exists will fragment reporting.

Current keys:
- Top-level: `greet_confirm_ids`, `confirm_signed_nda`, `participant_orientation`, `complete_hydra_intake`
- Scenario groups: `scenario_t1`, `scenario_a1`, `scenario_a2`, `scenario_m1`
- Scenario checks: `{code}_{camera_orientation|camera_placement|camera_obstructions|video_check|audio_check|performance_check}`
- Section headers: `{code}_before_recording`, `{code}_after_recording` (plus `note` helper copy; not checkable)

- Grouped scenario tasks are never toggled directly. They derive completion from all nested checks.
- Scenario groups unlock in list order. T1 requires all earlier top-level tasks; A1, A2, and M1 each require the preceding scenario to be complete.
- `before_recording` and `after_recording` are visual section headers, not checkable steps and not part of progress totals.
- Title-only items on the main list and detailed scenario checks are the manually completed records.
- Desktop uses a two-pane layout for scenario groups; title-only items toggle in place. Empty pane copy is “Select a Scenario” / “Subtasks will appear here.” Mobile uses drill-in only for scenario groups. Test both whenever task markup or CSS changes.
- Post-Session always shows Notes and Complete session; the button stays disabled until every checkable step in all three phases is done.
- Desktop sidebar collapse is stored in `wave_sidebar_collapsed`.
- Do not surface the words “parent” or “child” in the UI. Those names are internal only.

### CSS class names
The CSS relies on specific class names (`.s-item`, `.step-dot`, `.s-check`, etc.). Renaming breaks styling.

**Before changing**: Search the file for all occurrences and update them together. Test on mobile.

### Version numbering
Every code change should bump the version (at minimum the date segment). Moderators check "what version am I on?" when troubleshooting.

**Format**: `MAJOR.MINOR.MMDDYY` (e.g., `1.0.081726`). Always use current build date, not historical date.

---

## Other things to know

### Mobile-first design
UI optimized for phones (sidebar → overlay on narrow screens). Always test on real iPhone + Android before deploying. Desktop works too, but not the priority.

### Dark mode + pink accent
Kilo design system: dark background (#0f0f0e), pink accent (#EF43B3), "SF Pro Display" font stack. On-screen title is **mmWave** (nav + login). Light mode toggle exists in the nav.

### No third-party dependencies
Vanilla HTML, CSS, JavaScript only. No frameworks, no npm. Easy to deploy, no dependency rot.

### Real-time data sync (requires internet)
Moderators need internet connectivity during collection. App syncs to SharePoint on every step completion. If offline, will cache and sync when connection returns (not yet implemented in v1.0 — future feature if needed).

### SharePoint permissions
Collection sites will need access to the SharePoint Lists and Document Library (for reference media). Ensure permissions are set up before deployment. Test with real moderators from field.

### Power Automate quotas
Microsoft 365 has limits on Power Automate API calls. With 200+ sessions, could hit monthly ceiling. Monitor usage in PA admin dashboard. If hitting limits, consider archiving old SessionLog entries.

### Audit trail importance
SessionLog is the source of truth for QA. Every step is logged with timestamp. QA team can cross-reference SessionLog against recorded video to catch protocol violations. Don't lose this data.

### Code comments are spec
Comments in `app.html` explain *why* decisions were made, not just *what* they do. They're part of the specification. Read them before making changes.

### Testing checklist
Before declaring any change done:
- [ ] Open the app on a phone (iOS if possible)
- [ ] Sign in as a moderator
- [ ] Sign in as an admin
- [ ] Test core flows (view session, complete checklist, check inventory, sign out)
- [ ] Open browser console — no red errors?
- [ ] Force-refresh the page — state still loads?
- [ ] Wait a few minutes — periodic cloud sync triggers and re-renders?

---

## Action items

### Week 1 (Critical) — infrastructure + skeleton
- [ ] **Set up SharePoint Lists**: Moderators, Sites, Participants, Sessions, SessionLog, ReferenceMedia
  - [ ] Define schema for each table
  - [ ] Validate column types and constraints
- [ ] **Create Power Automate flows**: Read/write flows for each table, get URLs
  - [ ] Sessions read
  - [ ] SessionLog write
  - [ ] Sessions update (status/useable_minutes)
  - [ ] Moderators read
- [ ] **Set up GitHub repo** for version control
- [x] **Build `app.html` skeleton**:
  - [x] Copy from Kilo Task Tracker.html (structure/tokens)
  - [x] Add login screen
  - [ ] Wire flow URLs (constants exist, still empty)
  - [x] Implement cloud sync stubs (read sessions, write SessionLog — no-op until URLs are set)
  - [ ] Test end-to-end: login → see sessions → complete session → verify in SharePoint

### Week 2 (Critical) — content + admin
- [ ] **Define workflow**: Exact steps for single-participant and paired-participant sessions
- [ ] **Define reference media**: What clips/GIFs needed, where stored, upload to SharePoint
- [ ] **Write reminders + troubleshooting**: For menu panel
- [ ] **Populate ReferenceMedia table**: Link clips/GIFs to session steps
- [ ] **Customize `app.html`**: Insert actual session steps, reference media, reminders
- [ ] **Build admin dashboard**: Progress view (hours collected, target, active sites/moderators)
- [ ] **Test on real iPhones + Android phones** in field (not just desktop)

### Week 3 (Critical) — final + deploy
- [ ] **Full end-to-end testing**: Login → see sessions → complete → verify admin dashboard
- [ ] **Fix bugs from testing**
- [ ] **Brief moderator team**: How to use app, what each step means, how to handle errors
- [ ] **Deploy to GitHub Pages**
- [ ] **Verify data is flowing correctly**: Check SharePoint Lists, SessionLog populated, admin dashboard updating

### Before using in production
- [ ] **Test with real moderators** at real collection sites (not just dev/QA)
- [ ] **Verify SharePoint permissions**: Moderators can access Lists + Document Library
- [ ] **Backup strategy**: Export SessionLog regularly for QA review
- [ ] **Monitor Power Automate quota**: Check usage dashboard weekly during collection

### Post-launch (if time/scope allows)
- [ ] Add session timer (track real time per session)
- [ ] Add offline mode (cache sessions, sync when online)
- [ ] Add data export (moderators can download session log at end of day)
- [ ] Add more admin analytics (completion by site/moderator, trends, etc.)
- [ ] Create moderator user guide document

---

## If something breaks

### App loads but shows blank / unstyled HTML
- **Likely cause**: CSS has unclosed braces (iOS Safari is strict about this). Look for a `{` without matching `}`.
- **Fix**: Count braces carefully in the `<style>` section. Bump version, redeploy, and tell moderators to hard-refresh.

### App loads but login fails (username not recognized)
- **Likely cause**: Username not in Moderators SharePoint List, or list not syncing.
- **Check**: 
  1. Verify username is spelled correctly in Moderators list
  2. Verify active=true in Moderators list
  3. Check browser console for Power Automate flow errors
- **Fix**: Add username to Moderators list, bump version, redeploy.

### Sessions don't appear (sidebar is empty after login)
- **Likely cause**: Sessions read Power Automate flow is failing or returning empty data.
- **Check**: 
  1. Open browser console (`F12`), look for network errors or JS errors
  2. Check Power Automate flow status in PA admin (is flow enabled? did it trigger?)
  3. Verify Sessions list has data in SharePoint
- **Fix**: Debug PA flow, check flow URL in `app.html`, test flow manually, bump version, redeploy.

### Steps aren't logging (data not appearing in SessionLog)
- **This is critical**. If SessionLog isn't getting data, audit trail is broken and QA can't review.
- **Likely cause**: SessionLog write Power Automate flow is failing silently.
- **Check**:
  1. Open browser console, look for network errors
  2. Manually check SessionLog table in SharePoint (should have rows)
  3. Check Power Automate run history (did the flow execute? did it fail?)
- **Fix**: Debug PA flow, test manually, verify flow URL, bump version, redeploy. Don't continue collection until this is fixed.

### Moderator can't see reference media (clips/GIFs don't load)
- **Likely cause**: Reference media URLs are wrong, or moderator doesn't have permission to access the file.
- **Check**:
  1. Verify URLs in ReferenceMedia list are correct (copy/paste URL into browser — can you access it?)
  2. Verify moderators have access to SharePoint Document Library (permissions set correctly?)
- **Fix**: Update URLs, adjust SharePoint permissions, bump version, redeploy.

### Admin dashboard isn't updating (shows old data)
- **Likely cause**: Power Automate flows for admin dashboard are failing, or dashboards weren't set up.
- **Check**: 
  1. Verify SessionLog has recent rows (is moderator app logging?)
  2. Verify Power Automate flows exist and are enabled
  3. Test dashboard queries manually
- **Fix**: Debug flows, recreate dashboard queries, verify data refresh rate.

### Sessions completed but data loss (restarting app shows completed sessions as incomplete)
- **This shouldn't happen**. SessionLog is write-once in SharePoint.
- **Likely cause**: Browser cache issue (showing stale data) OR SessionLog write actually failed silently.
- **Check**:
  1. Hard-refresh browser (`Ctrl+Shift+R` or `Cmd+Shift+R`)
  2. Check SharePoint SessionLog directly (was the session actually logged?)
- **Fix**: If data truly lost, check Power Automate flow history for errors. May need to manually add missing records to SessionLog.

### App version number isn't updating (users see old version)
- **Likely cause**: Browser cached old HTML file.
- **Fix**: Bump `APP_VERSION` constant (this changes the HTML), redeploy. Tell moderators to hard-refresh.

---

## Getting help

- **For architectural questions**: Read `Reference Files/ARCHITECTURE_REFERENCE.md` and `Dev_Notes.md`
- **For version history / what changed**: Read `Changelog.md`
- **For original spec**: Read `Reference Files/PROJECT_INSTRUCTIONS.md`
- **For design tokens / styling**: Read `Reference Files/CENTIFIC_DESIGN_SYSTEM_01.md` or `Reference Files/Project Kilo Files/`
- **For technical deep-dive**: Use Claude (point it at `Dev_Notes.md` first, then describe what you're trying to do)

---

## Contact

If you inherit this project and get stuck, ask Riley (riley.robertson@centific.com) for context on:
- Original design decisions
- Why certain patterns were locked in
- Which systems consume the app's output
- Moderator/admin contact list for testing

Good luck! 🚀
