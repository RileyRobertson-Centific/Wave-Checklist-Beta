# Centific Field Data-Collection Apps: Build Workflow and Replication Guide

A handoff document for a teammate who will use Claude to maintain one of these apps and replicate the pattern for their own project. It covers the technology, the backend (database and connectors) and how to configure it, the way we work with Claude, deployment, and a step-by-step replication checklist.

If you only read one section first, read section 2 (Technology) and section 5 (Connectors), then section 11 (Replication).

---

## Table of contents

1. What these apps are
2. Technology stack
3. Architecture overview
4. The database: Excel tables on SharePoint
5. The connectors: Power Automate HTTP flows
6. Wiring flows into the app
7. Working with Claude (the build workflow)
8. Deployment (GitHub Pages)
9. Pre-ship QA checklist
10. Key learnings and gotchas
11. Replicating for a new project
12. Files to give Claude

---

## 1. What these apps are

Each app is a single, self-contained web app used by field staff during data-collection sessions. A typical app has two roles in one file:

* A moderator app: checklists and forms that field staff work through during a session.
* An admin app: dashboards and monitoring that leads use to track progress, review quality, and export data.

The role is decided at login from a lookup table, so there is one file, one deployment, and one shared design system. Project Kilo (Apple Isaac face-video collection) and Project Orbit (multi-station camera calibration) are the two reference builds.

The defining constraints:

* One static `index.html` file. All HTML, CSS, and JavaScript live in it. No framework and no build step.
* The backend is Power Automate HTTP flows reading and writing Excel tables on SharePoint. There is no separate database or server.
* It deploys to GitHub Pages as a static file.

This trades tidy code organization for zero deployment friction: anyone can open the file in a browser, edit it, and push to deploy. It suits a small fixed team (roughly 5 to 30 users) on modern browsers.

---

## 2. Technology stack

| Layer | What we use | Notes |
|---|---|---|
| Frontend | One static `index.html`, vanilla JavaScript | No React, no build pipeline, no npm at runtime |
| Styling | Plain CSS with CSS variables, dark and light themes | Apple-inspired: frosted glass, spring easing, SF Pro font stack |
| Charts | Chart.js v4 (Kilo), Plotly (Orbit) | Loaded from a CDN via a script tag |
| Excel export | SheetJS (xlsx) for data, ExcelJS plus html2canvas when charts must be embedded as images | SheetJS cannot embed images |
| Backend API | Power Automate HTTP flows | One flow per read or write operation |
| Database | Excel tables on SharePoint | Every column stored as Text; parsed in the browser |
| Auth to flows | Signature baked into the flow trigger URL | The trigger is set to accept requests from "Anyone" |
| Hosting | GitHub Pages | Serves `index.html` at the repo root |
| Build assistant | Claude | Used for all code changes, iteratively |
| Local checks | Node.js for parse and structure checks, headless Chrome (Puppeteer) for render checks | Run before every delivery |

There is no package.json shipped with the app. Node and Puppeteer are only used locally by Claude to verify a build before it ships.

---

## 3. Architecture overview

```
  Browser (index.html)                Power Automate                 SharePoint
  +--------------------+   HTTPS POST  +------------------+  Graph   +----------------+
  |  Moderator app     | ------------> |  Write flow      | -------> |  Excel table   |
  |  Admin app         |              |  (add a row)      |         |  (a worksheet) |
  |                    | <----------- |  Read flow        | <------- |                |
  |  vanilla JS state  |   JSON rows   |  (list rows)      |         |                |
  +--------------------+              +------------------+          +----------------+
```

Key ideas:

* Two apps in one file. Login reads a moderator or admin lookup table and shows the correct app. No hardcoded backdoors.
* State lives in a single global object in the browser, saved to `localStorage` per username, with the cloud (Excel via flows) treated as the source of truth and `localStorage` as an offline fallback.
* Each Excel table gets a pair of flows: one to write (append or update a row) and one to read (return rows). Some tables only need one direction.
* Reads refresh in the background so the app is never blank while it waits.

---

## 4. The database: Excel tables on SharePoint

The "database" is a set of Excel worksheets, each formatted as an Excel Table (Insert, then Table), stored in workbooks on a SharePoint document library. For our team that library is the Data Collection US HUB site, but any SharePoint site the flows can reach will work.

### Conventions that keep this reliable

* Every column is Text format. This includes dates, numbers, and yes or no values. Excel's typed columns cause subtle mismatches with flows (date formats, empty cells becoming null). Text is predictable, and the browser parses values into real types.
* Every table has one unique ID column, generated in the browser as something like `prefix_user_entity_timestamp`.
* Keep column names simple. Avoid spaces, slashes, and question marks in any column you plan to filter on inside a flow (more on this in section 5).
* Prefer append-only logs over editing rows in place. To record a status change, add a new row rather than overwriting. Reading the latest row by timestamp gives you the current state plus a full history and avoids write conflicts.

### A typical table set

| Table | Purpose |
|---|---|
| Moderators or Users | Login directory: login ID, name, role (drives which app loads) |
| Assignments or Bookings | Scheduled sessions |
| WorkLog or StatusLog | Append-only status events |
| SessionState | Cross-device session sync |
| BugReport or Feedback | Issue reports (and, in Kilo, survey submissions) |
| Inventory and InventoryLog | Equipment tracking, when the app needs it |

You do not need all of these. Start with the login directory plus whatever your first feature writes and reads.

---

## 5. The connectors: Power Automate HTTP flows

A "connector" here means a Power Automate cloud flow that starts with an HTTP request trigger and either writes to or reads from one Excel table. The browser calls the flow's URL directly. This section is the part most worth getting right.

### 5.1 The write flow (append a row)

Build it in the Power Automate web portal:

1. Create an Instant cloud flow, trigger: "When an HTTP request is received".
2. On the trigger, set "Who can trigger the flow" to "Anyone". This puts a signature in the URL so no separate login is needed.
3. Provide a request body JSON schema (paste a sample payload and let it generate the schema), for example a body with `loginId`, `issueType`, and `description`.
4. Add an Excel Online (Business) action: "Add a row into a table". Point it at the workbook, worksheet, and table, and map each Excel column to the matching field from the trigger body.
5. Save. Open the trigger again and copy the "HTTP POST URL". That full URL, signature and all, is what the app uses.

The app calls it like this:

```
await fetch(WRITE_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ loginId: id, issueType: 'Survey Submitted', description: jsonString })
});
```

### 5.2 The read flow (return rows)

1. Same trigger: "When an HTTP request is received", "Anyone".
2. Add "List rows present in a table" (Excel Online Business), pointing at the same table.
3. Add a "Response" action (or "Respond to a PowerApp or flow") that returns the rows as JSON, typically the output of List rows.
4. Turn pagination ON for the List rows action (see 5.5).
5. Copy the trigger URL.

The app calls it with an empty body and reads the array back:

```
const res = await fetch(READ_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({})
});
const data = await res.json(); // an array, or { value: [...] }, depending on how you shaped the Response
```

Write the client to accept a few shapes (a bare array, `{ value: [...] }`, or `{ rows: [...] }`) so small differences in how you build the Response do not break it.

### 5.3 The login flow (filter and branch)

The login flow must not return success for everyone. Build it to filter by the submitted login ID and return a clear found or not-found result:

1. Trigger: HTTP request with a body containing the login ID.
2. "List rows" with a Filter Query on the login ID column only.
3. A Condition on whether any row came back.
4. Response returns `{ "found": true }` with the profile fields, or `{ "found": false }`, echoing the login ID back.

The app treats `found: false` as "unknown user" and refuses the login.

### 5.4 Filter safely

Only put a Filter Query on columns whose names have no spaces, slashes, or question marks. Odata filters on awkward column names fail in confusing ways. Filter on a safe column (like a login ID) and let the browser do any remaining filtering.

### 5.5 The 256-row cap

"List rows" returns at most 256 rows by default. On any table that grows past that, the newest rows silently vanish from reads. Fix it inside the flow: open the List rows action, Settings, Pagination On, threshold something large like 5000. If a read ever returns exactly 256 rows, assume it was truncated.

### 5.6 Other flow rules we learned the hard way

* Content-Type is always `application/json`. Do not use `text/plain`.
* Any dynamic value in a flow (an expression) must be entered through the expression (fx) tab so it evaluates. Typed literal text does not evaluate.
* If a read is being cached by the browser or an edge, add cache-busting on the client: `cache: 'no-store'`, no-cache headers, and a changing value in the body.
* Dates can come back as Excel serial numbers. Convert from the 1899-12-30 epoch in the browser, and never compare timestamps as text; compare them as numbers (epoch milliseconds).

---

## 6. Wiring flows into the app

Flow URLs live as named constants near the top of the JavaScript. The pattern we use is dormant-until-wired: a feature ships with its URL set to an empty string, so it silently does nothing until someone pastes the real URL in.

```
const ASSIGNMENT_READ_URL  = 'https://...';   // live
const BUGREPORT_WRITE_URL   = 'https://...';  // live
const NEW_FEATURE_READ_URL  = '';             // dormant until pasted
```

Client code checks for the empty string and no-ops cleanly when a URL is missing, so half-built features never crash the app.

### Worked example: the Kilo survey

The moderator survey reuses the existing BugReport connector rather than a new table:

* Write: it posts one row to the bug-report write flow with `loginId` set to the moderator, `issueType` set to `Survey Submitted` (this acts as the status marker), and `description` set to the answers as a JSON string.
* Read (per moderator): the login-time check posts the login ID to a read flow that returns whether a `Survey Submitted` row exists for that person, so a moderator is only ever asked once.
* Read (admin view): the admin Survey Results page reads all rows and keeps only `issueType == 'Survey Submitted'`, then parses each `description` back into answers for charts and a breakdown.

The important detail: because bug reports and survey rows share one table, any read that decides "already submitted" must filter on both the login ID and the `Survey Submitted` marker, or an unrelated bug report would be mistaken for a completed survey.

---

## 7. Working with Claude (the build workflow)

This is the process to hand off. It is less about any single feature and more about how changes get made safely.

### Set up a Claude Project

Create a Claude Project and add these reference files to its knowledge so every chat starts with full context:

* `ARCHITECTURE_REFERENCE.md`: the patterns, component library, state model, and backend recipes.
* A design system file (for example the Kilo or Orbit design tokens).
* `PROJECT_INSTRUCTIONS.md`: how the app should behave and be built.
* The App Starter Kit: a ready-to-paste kickoff prompt plus an intake questionnaire.

Put the live `index.html` in the Project too, or upload it at the start of a working session.

### How a change happens

1. Describe the change. If it is ambiguous, Claude asks a short multiple-choice question before building.
2. For anything sizeable (new screen, new feature, layout change), Claude shows a quick mockup or a plan and the two or three decisions it made, and waits for a yes before writing code.
3. Claude makes surgical edits to the one file, anchored to surrounding context, rather than rewriting the file.
4. Claude runs the pre-ship checks (section 9) and reports what changed, how to test it, what was deliberately left alone, and any gotchas.
5. You deploy (section 8).

### Conventions Claude follows

* Version stamp `MAJOR.MINOR.MMDDYY`. The date part changes on every code change; the major and minor only change when you ask.
* One accent color per app, used consistently.
* High comment density, especially around browser quirks and trade-offs.
* Dormant-until-wired for anything that depends on a flow you have not built yet.

---

## 8. Deployment (GitHub Pages)

1. The repo serves `index.html` from its root. If the working file has another name, rename it to `index.html` before committing.
2. Commit and push to the branch GitHub Pages serves (commonly `main`).
3. GitHub Pages publishes it at `https://<user-or-org>.github.io/<repo>/`.
4. Hard-refresh to get past the browser cache. The visible version stamp confirms which build you are on.

There is no build step. What you commit is what runs.

---

## 9. Pre-ship QA checklist

Run before every delivery. These have each caught real breakage.

* JavaScript parses. Extract every script block and parse it with a real engine (Node's `new Function(src)`), not a linter.
* CSS braces balance. Walk every character in each style block, plus one for each open brace and minus one for each close brace. The final count must be zero. An unbalanced brace makes iOS Safari drop the rest of the stylesheet and render raw HTML, while desktop looks fine.
* Template-literal backticks are even. An odd count means a broken string.
* Feature presence. For each change, grep for the new function names, element IDs, or text to confirm the edit landed.
* Version stamp updated.
* When practical, a headless Chrome render check that exercises the new logic.

---

## 10. Key learnings and gotchas

* Cloud is the source of truth; `localStorage` is a per-device cache. Reading only from `localStorage` breaks the first login on a new device.
* Use a per-username `localStorage` key. A single shared key lets two people on one device overwrite each other.
* Never rebuild the DOM (a full innerHTML replace) while a text field is focused. On mobile it collapses the keyboard. Guard with a defer-while-editing flag.
* Compare IDs as strings on both sides, never by loose equality.
* Sort timestamps as numbers (epoch), never as text.
* Debounce auto-save. A cloud write on every keystroke is a self-inflicted denial of service on your flow. Three to five seconds is fine.
* Use a fire-and-forget beacon on page unload so the last few seconds of edits are not lost when the tab closes.
* Keep a tolerant column matcher (case-insensitive, trims, falls back) so a renamed or oddly-cased column does not silently drop rows.
* Turn on pagination for any growing table (the 256 cap).

---

## 11. Replicating for a new project

A clean path for a teammate starting their own app.

**Step 1: Set up the Claude Project.** Create a new Claude Project and add the reference files from section 12. Paste the Starter Kit kickoff prompt, then fill in the intake questionnaire (app name, who uses it, the workflow steps, the data captured, branding).

**Step 2: Pick one accent color** and confirm light plus dark or a single theme.

**Step 3: Plan the tables.** List the Excel tables you need. Start with the login directory plus the first feature's table. Make every column Text and give each table a unique ID column.

**Step 4: Build the Excel workbook on SharePoint.** Create the worksheet, format the range as a Table, and add the columns.

**Step 5: Build the flows.** For each table, build a write flow and a read flow as in section 5. Build the login flow that filters and returns found or not found. Copy each trigger URL.

**Step 6: Wire the URLs.** Paste each URL into its constant near the top of the JavaScript. Leave any not-yet-built flow as an empty string.

**Step 7: Build features iteratively with Claude.** Ship small, verify with the QA checklist, deploy, repeat.

**Step 8: Deploy to GitHub Pages** as `index.html`.

A useful rule of thumb: this pattern fits a small team, mobile-first, infrequent updates, and structured data. If you outgrow it (more than roughly twenty tables, more than fifty users, or you need sub-second real-time updates), that is the signal to move to a real database and API.

---

## 12. Files to give Claude

Hand these to your teammate so their Claude Project starts with the same foundation:

| File | What it provides |
|---|---|
| `ARCHITECTURE_REFERENCE.md` | The full architecture, component library, state model, and backend recipes |
| Design system file | CSS tokens, typography, spacing, component styles |
| `PROJECT_INSTRUCTIONS.md` | How the app behaves and how to build it |
| App Starter Kit | The kickoff prompt plus the intake questionnaire |
| This guide | The workflow, backend configuration, and replication steps |

With those in a Claude Project, a new chat can go straight from an intake questionnaire to a working single-file app, and your teammate can maintain it the same way this one is maintained.
