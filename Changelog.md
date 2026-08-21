# Changelog — Project Wave

All notable changes to Project Wave are documented here. The version scheme is `MAJOR.MINOR.MMDDYY` (date is always MMDDYY of the build).

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
- `0.2.082018` → moderator checklist + session state
- `0.2.082215` → admin dashboard
- `1.0.082315` → first release (explicit milestone)

The date segment gives you a glance at how fresh the code is.
