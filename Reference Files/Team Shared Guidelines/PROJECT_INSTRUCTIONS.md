# PROJECT INSTRUCTIONS — Centific App (Inventory + Moderator Tracking)

Paste this into the "Project Instructions" field of your new Claude Project.

---

This project builds a **moderator-and-admin app for [PROJECT NAME]** with three core features:

1. **Moderator app**: data-collection checklists that moderators tick through during their workday. Each moderator has their own state, tracks progress through a defined workflow, and reports completion.

2. **Admin app**: read-only monitoring of moderator activity. Admins can see which moderators are currently working, what stage they're at, completion stats, and historical activity. Admins do not perform data collection themselves.

3. **Inventory tracking**: items going in and out of the team's inventory (equipment, supplies, samples — whatever this project tracks). Moderators check items in/out; admins see who has what.

The app runs on **iOS, Android, and desktop** (browser-based). Mobile is first-class — design and develop mobile-first, then enhance for desktop.

## Architecture inheritance

This app inherits from **Project Orbit** — a data-collection moderator app I built previously. The complete architectural reference is uploaded to this Project's knowledge as `ARCHITECTURE_REFERENCE.md`. **Read it before writing any code.** It covers:

- Single-file HTML + vanilla JS approach (no framework, no build pipeline)
- Design tokens (colors, type, spacing, accent color `#00F0FF`)
- Component library (.btn, .icon-btn, .modal, .toast, .badge, etc.)
- State management (`state` object + per-username localStorage)
- Power Automate + Excel backend (read/write URL pairs, soft-delete pattern, status-event logs)
- Cloud-first refresh architecture (with 5 trigger types)
- Mobile-specific patterns (safe-area handling, swipe gestures, orientation lock)
- Login flow + welcome modal variants (today_completed / next_session / no_session)
- Verification before ship (JS parse + CSS brace balance + feature regex checks)
- Common anti-patterns to avoid

The reference document also contains code snippets you should copy directly rather than reinvent — `showToast`, `appConfirm`, the swipe gesture handler, the modal builder, the cloud refresh helper, etc.

## What's the same as Orbit

- Single `.html` file deployed via GitHub Pages
- Dark-default theme with light variant; theme toggle in settings menu
- Accent color: `#00F0FF` (electric cyan)
- Power Automate + Excel as backend (read/write URL pairs per table)
- Per-username localStorage; cloud-first refresh with localStorage fallback
- Version scheme: `MAJOR.MINOR.MMDDYY`, bumped every code change
- Verification discipline: JS parse + CSS brace balance + feature regex spot-checks BEFORE ship
- Two apps in one HTML file (moderator + admin) with role detection at login
- Welcome modal variants at login (state-aware)
- iOS-style frosted glass overlays, Apple-style typography (negative letter-spacing)

## What's different from Orbit

- **Inventory tracking** is new. Use the "stock-delta journal" pattern from the reference doc Appendix — an `Inventory` table for current snapshots + an `InventoryLog` table for transactions. Items can be checked in/out by users with timestamps.
- **Mobile is universal** (iOS + Android). Orbit was iPhone-heavy; this app needs equal Android support. Key implications:
  - Use `dvh` units for full-height elements (Android keyboard quirk)
  - Add `overscroll-behavior-y: contain` to disable Android pull-to-refresh interfering
  - Orientation lock will fully enforce on Android Chrome (no PWA install needed)
- **Admin is read-only**. Admin app is dashboards + monitoring, NOT data entry. Different visual emphasis: more charts and tables, less form-filling.
- **No (or different) participant/team concept**. Orbit had teams of 2 moderators sharing assignments. Confirm with me whether this new app has teams, individuals, or some other unit before assuming.

## When starting work

1. **Read `ARCHITECTURE_REFERENCE.md` first.** Don't write any code until you've absorbed at least sections 1-4 (philosophy, structure, design tokens, components).
2. **Ask clarifying questions** if my spec is ambiguous. Common ambiguities:
   - "Inventory" — is it equipment, consumables, samples, or something else? How granular (1 row per item vs 1 row per SKU with a quantity)?
   - "Moderator workflow" — what are the actual steps? Linear (must complete A before B) or parallel?
   - "Admin monitoring" — real-time-ish (within minutes) or end-of-day reports?
3. **Follow the verification discipline.** Every build ends with: JS parses, CSS braces balanced, feature regex checks pass, version stamp bumped. Show the verification results before declaring the build done.
4. **Comment density: high.** WHY, not just WHAT. Especially around browser quirks, mobile-specific decisions, and trade-offs.

## When iterating

- Keep a work copy at `/home/claude/app.html`; only copy to `/mnt/user-data/outputs/app.html` AND `/mnt/user-data/outputs/index.html` after verification passes.
- Use `present_files` to share the final build with me.
- If you need to make a significant architectural change, propose it before implementing — these decisions compound.

## Tooling expectations

- I deploy to GitHub Pages. The deploy target may be a different repo than Orbit's — I'll provide the repo when relevant.
- Power Automate flows need to be set up separately (in PA's web UI) before their URLs can be wired. You'll guide me through creating each flow when I ask for a new backend table.
- I prefer iterative builds over big-bang. Ship small, verify, iterate.

## Style preferences

- Honest assessments over reassurance. If you think something is a bad idea, say so.
- Concrete examples over abstract advice.
- "Here's what I did and why" summaries after each major build.
- Flag things worth knowing even if I didn't ask (PA flow quotas, iOS PWA install caveats, etc.).
- Don't oversell. If a feature has caveats, name them.
