# Centific App Architecture Reference

This document captures the patterns, conventions, and battle-tested approaches from **Project Orbit** — a moderator-and-admin app for data collection — so they can be reused on related projects. Read this in full before writing code for any new app inheriting from Orbit's foundation.

Orbit is a single-file HTML app (~1.1 MB, ~24,000 lines) deployed via GitHub Pages, with Power Automate flows reading/writing to Excel tables as the backend. The patterns below have all been validated in production with moderators using the app on real iPhones in the field.

---

## Table of contents

1. Architecture philosophy
2. Project structure & build discipline
3. Design system — colors, type, spacing
4. Component library
5. Layout patterns — desktop & mobile
6. Modal & overlay patterns
7. State management
8. Power Automate + Excel backend
9. Cloud-first refresh architecture
10. Mobile-specific patterns (iOS + Android)
11. Login flow & welcome modals
12. Commenting & verification discipline
13. Anti-patterns — what NOT to do
14. Quick-start checklist for a new app

---

## 1. Architecture philosophy

**Single-file HTML, vanilla JS, no frameworks.** One `.html` file contains all HTML, CSS, and JavaScript. No React, no Vue, no build pipeline. Shipped to GitHub Pages as static files. This trades upfront code organization for zero deployment friction — anyone can fork the file, edit it locally in any browser, and push to deploy.

**Why no framework**: the app is for a small fixed team (5-20 users), runs in modern browsers only, and the cost of a build pipeline outweighs the benefit at this scale. Vanilla JS forces clearer thinking about state and DOM updates.

**Trade-offs**:
- Long files (24K+ lines). Mitigated by aggressive section comments and consistent naming.
- No type safety. Mitigated by careful function design + extensive inline comments documenting expected shapes.
- Manual DOM updates. Mitigated by a single `renderApp()` function that idempotently rebuilds visible UI from state.

**Two apps in one HTML file**: Orbit has both a moderator app and an admin app in the same file. Mode switching is done at login (admin usernames are hardcoded). This keeps deployment trivial and lets both apps share design system / state schema / backend.

**Backend = Excel tables via Power Automate flows.** No custom database, no Azure SQL, no Firebase. Excel sheets are the source of truth; Power Automate flows are the read/write API.

**Why this works**: the team already uses Excel/SharePoint. Operators trust spreadsheets they can open and inspect. PA flows give us authenticated, audited reads/writes without managing a server.

**Trade-offs**: Excel rows are capped at ~256 per single PA call (pagination required); Excel data types are flexible (we use Text columns for everything, parse as needed); concurrent writes need careful merge logic.

---

## 2. Project structure & build discipline

### File layout

```
/orbit.html              # The whole app — HTML + CSS + JS
/README.md               # Brief description, deploy instructions
/CHANGELOG.md            # Version history (optional but useful)
```

No `src/`, no `package.json`, no `node_modules`. Deploy = rename `orbit.html` to `index.html` and push to GitHub Pages.

### Version stamping

Every code change bumps a version constant:

```js
const APP_VERSION = '1.2.051436';  // MAJOR.MINOR.MMDDYY
```

**Convention**: `MAJOR.MINOR.MMDDYY`. Only bump MAJOR.MINOR for explicit user-requested feature milestones. The date suffix changes every code change. Lets you compare "the version on my phone" vs "what I just shipped" at a glance. Show the version in the UI footer or settings drawer.

### Verification before ship — non-negotiable

Before declaring any build done:

1. **JavaScript parses cleanly.** Extract all `<script>` content, run `new Function(src)` on each.
2. **CSS braces are balanced.** Walk every char in every `<style>` block, increment on `{`, decrement on `}`. Final depth MUST be 0. Even depth=1 catastrophically breaks iOS Safari (its CSS parser drops everything after the unclosed rule).
3. **Feature regex spot-checks.** For each change, write a regex that verifies the change landed correctly.
4. **Version stamp bumped.**

```js
const fs = require('fs');
const html = fs.readFileSync('app.html', 'utf8');

// 1. JS parses
const scripts = [...html.matchAll(/<script[^>]*>([\s\S]*?)<\/script>/g)].map(m => m[1]);
scripts.forEach((src, i) => { try { new Function(src); } catch (e) { console.error('JS', i, 'FAIL:', e.message); process.exit(1); } });

// 2. CSS balance
const styleMatch = html.match(/<style[^>]*>([\s\S]*?)<\/style>/);
let depth = 0;
for (const c of styleMatch[1]) { if (c === '{') depth++; else if (c === '}') depth--; }
if (depth !== 0) { console.error('CSS BRACE UNBALANCED:', depth); process.exit(1); }

// 3. Feature checks
const checks = {
  'feature foo exists': /function fooHandler\(/.test(html),
  'version bumped':     /APP_VERSION = '1\.2\.051436'/.test(html),
};
Object.entries(checks).forEach(([n, ok]) => console.log((ok ? '✓' : '❌'), n));
```

**The CSS brace balance check has saved us multiple times.** A duplicated selector left an unclosed rule, the app rendered as raw unstyled HTML on iPhone but worked on desktop. Skipping this check means the next iOS user sees a broken app.

---

## 3. Design system — colors, type, spacing

### Theme architecture

```html
<html data-theme="dark">
```

All colors are CSS variables defined twice — once for `[data-theme="dark"]` and once for `[data-theme="light"]`. Theme toggle flips the attribute; the browser handles the rest.

### Accent color — pick one and own it

Orbit's accent is electric cyan: `--accent: #00F0FF`. Used for primary buttons, selected states, logo, progress bars, focus rings.

**Pick one accent for your new app and use it ruthlessly.** Don't introduce a secondary accent unless absolutely necessary — multiple accents fragment visual hierarchy.

### Full token list

```css
:root {
  /* Brand accent */
  --accent:     #00F0FF;
  --accent-dim: #00d4e0;
  --accent-soft: rgba(0,240,255,0.12);
  --accent-ink: #00F0FF;     /* overridden in light theme for contrast */

  /* Semantic — badges, alerts, status */
  --amber: #F59E0B;  --amber-bg: #FFF8EC;  --amber-text: #92400E;
  --green: #22c55e;  --green-bg: #EAF3DE;  --green-text: #27500A;
  --red:   #ef4444;  --red-bg:   #FEF2F2;  --red-text:   #991b1b;
  --blue:  #3b82f6;  --blue-bg:  #EFF6FF;  --blue-text:  #1e3a8a;

  --r:  12px;
  --rl: 18px;

  --font:     -apple-system, 'SF Pro Display', 'SF Pro Text', BlinkMacSystemFont, 'Helvetica Neue', sans-serif;
  --font-num: 'Avenir Next', 'Avenir', 'SF Pro Display', -apple-system, sans-serif;
}

[data-theme="dark"] {
  --bg:  #0f0f0e;
  --bg2: #1a1a18;
  --bg3: #232320;
  --bg4: #2d2d29;
  --text:  #F7F5F0;
  --text2: #a8a8a0;
  --text3: #7a7a72;
  --border:        rgba(255,255,255,0.08);
  --border-strong: rgba(255,255,255,0.14);
  --input-bg: #232320;
  --card-bg:  #1a1a18;
  --nav-bg:   rgba(15,15,14,0.88);
  --sidebar-bg: #1a1a18;
  --shadow-sm: 0 2px 12px rgba(0,0,0,0.4);
  --shadow-md: 0 8px 40px rgba(0,0,0,0.5);
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.6);
}

[data-theme="light"] {
  --accent-ink: #0891B2;
  --bg:  #f5f5f7;
  --bg2: #ffffff;
  --bg3: #fbfbfd;
  --bg4: #e8e8ed;
  --text:  #1d1d1f;
  --text2: #4a4a4f;
  --text3: #8a8a8e;
  --border:        rgba(60,80,160,0.12);
  --border-strong: rgba(50,70,140,0.20);
  --input-bg: #f5f5f7;
  --card-bg:  #ffffff;
  --nav-bg:   rgba(245,245,247,0.88);
  --sidebar-bg: #ffffff;
  --shadow-sm: 0 2px 12px rgba(0,0,0,0.06);
  --shadow-md: 0 8px 40px rgba(0,0,0,0.10);
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.12);
}
```

### Type scale (informal but consistent)

| Use | Size | Weight | Letter-spacing |
|---|---|---|---|
| Hero / display | `clamp(28px, 5.5vw, 52px)` | 600 | -0.025em |
| Section title | 19-22px | 600 | -0.02em |
| Body | 14-15px | 400-500 | -0.01em |
| Label / caption | 11-13px | 500-600 | 0 to +0.04em |
| Uppercase label | 10.5-11.5px | 600 | 0.06-0.08em + uppercase |

**Font rule**: `var(--font)` for everything text. `var(--font-num)` (Avenir) for numbers in pills, counters, statistics. Avenir's numbers are tabular and visually heavier, better for at-a-glance numeric content.

### Letter-spacing — Apple-style

All headings get slight negative letter-spacing (-0.005 to -0.025em). Tighter on display sizes, looser on captions. Makes the app feel native on iOS.

### Spacing — multiples of 4

8, 12, 14, 16, 18, 22, 24, 32. Padding 14-18px on cards, gap 8-12px between buttons, 22-26px on modal interior.

---

## 4. Component library

### Buttons

```html
<button class="btn btn-primary">Continue</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-ghost">Cancel</button>
<button class="btn btn-danger">Sign out</button>
```

```css
.btn {
  font-family: var(--font);
  font-size: 15px; font-weight: 500;
  letter-spacing: -0.01em;
  padding: 10px 16px;
  border-radius: var(--r);
  border: 0.5px solid transparent;
  display: inline-flex; align-items: center; gap: 8px;
  transition: all 0.22s cubic-bezier(0.4,0,0.2,1);
}
.btn-primary {
  background: var(--accent); color: #001114; border: none;
  font-weight: 600;
}
.btn-primary:hover {
  background: var(--accent-dim); color: #001114;
  transform: translateY(-1px);
  box-shadow: 0 6px 18px rgba(0,240,255,0.45);
}
.btn-primary:active { transform: scale(0.98); }
.btn-primary:disabled { opacity: 0.45; cursor: not-allowed; transform: none; box-shadow: none; }

.btn-secondary { background: var(--bg2); color: var(--text); border: 0.5px solid var(--border-strong); }
.btn-secondary:hover { background: var(--bg4); }

.btn-ghost { background: transparent; color: var(--text2); border: 0.5px solid var(--border-strong); }
.btn-ghost:hover { background: var(--bg3); color: var(--text); }

.btn-danger { background: transparent; color: var(--red); border: 0.5px solid rgba(239,68,68,0.30); }
.btn-danger:hover { background: rgba(239,68,68,0.10); }
```

**Signature lift-on-hover** (`translateY(-1px)`) + colored shadow on primary. **Press-shrink** (`scale(0.98)`) on `:active` for tactile feedback. Don't drop these — they're distinctive.

### Icon buttons

```html
<button class="icon-btn" aria-label="Refresh">
  <svg width="18" height="18" viewBox="0 0 24 24">...</svg>
</button>
```

```css
.icon-btn {
  width: 34px; height: 34px;
  border-radius: 10px;
  background: var(--bg3);
  display: flex; align-items: center; justify-content: center;
  color: var(--text2);
  border: none; cursor: pointer;
  transition: all 0.18s ease;
}
.icon-btn:hover  { background: var(--bg4); color: var(--text); }
.icon-btn:active { transform: scale(0.95); }

/* Mobile bumps to 40x40 for thumb-friendly tap targets */
@media (max-width: 760px) {
  .icon-btn { width: 40px; height: 40px; }
  .icon-btn svg { width: 18px; height: 18px; }
}
```

**Always include `aria-label`.** Use `title` for desktop tooltips.

### Toggle switches

```html
<span class="theme-toggle" id="myToggle"></span>
```

```css
.theme-toggle {
  width: 38px; height: 22px;
  background: var(--bg4);
  border-radius: 999px;
  position: relative; cursor: pointer;
  transition: background 0.18s ease;
}
.theme-toggle::after {
  content: '';
  position: absolute; top: 2px; left: 2px;
  width: 18px; height: 18px;
  background: #fff;
  border-radius: 50%;
  transition: transform 0.18s ease;
}
.theme-toggle.on { background: var(--accent); }
.theme-toggle.on::after { transform: translateX(18px); }
```

`.on` class is generic — use for any boolean toggle.

### Cards

```css
.card {
  background: var(--card-bg);
  border: 0.5px solid var(--border);
  border-radius: var(--rl);
  padding: 18px 20px;
  box-shadow: var(--shadow-sm);
}
.card-head { display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 14px; }
```

### Status badges

```css
.badge {
  display: inline-flex; align-items: center; gap: 4px;
  padding: 3px 9px;
  border-radius: 99px;
  font-size: 11.5px; font-weight: 600;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}
.badge-green { background: var(--green-bg); color: var(--green-text); border: 0.5px solid rgba(34,197,94,0.30); }
.badge-amber { background: var(--amber-bg); color: var(--amber-text); border: 0.5px solid rgba(245,158,11,0.30); }
.badge-red   { background: var(--red-bg);   color: var(--red-text);   border: 0.5px solid rgba(239,68,68,0.30); }
.badge-blue  { background: var(--blue-bg);  color: var(--blue-text);  border: 0.5px solid rgba(59,130,246,0.30); }
```

### Toast notifications

```js
function showToast(message, type = 'info', duration = 3000) {
  let toast = document.getElementById('globalToast');
  if (!toast) {
    toast = document.createElement('div');
    toast.id = 'globalToast';
    toast.className = 'toast';
    document.body.appendChild(toast);
  }
  toast.className = `toast toast-${type}`;
  toast.textContent = message;
  void toast.offsetWidth;  // force reflow so transition runs on rapid re-show
  toast.classList.add('show');
  clearTimeout(toast._hideTimer);
  toast._hideTimer = setTimeout(() => toast.classList.remove('show'), duration);
}
```

```css
.toast {
  position: fixed; bottom: 24px; left: 50%;
  transform: translateX(-50%) translateY(20px);
  z-index: 999;
  padding: 12px 20px;
  background: var(--bg2);
  border: 0.5px solid var(--border-strong);
  border-radius: 99px;
  color: var(--text);
  font-size: 14px;
  box-shadow: var(--shadow-md);
  opacity: 0; pointer-events: none;
  transition: opacity 0.25s ease, transform 0.25s ease;
}
.toast.show { opacity: 1; transform: translateX(-50%) translateY(0); pointer-events: auto; }
.toast.toast-success { border-color: rgba(34,197,94,0.40); }
.toast.toast-error   { border-color: rgba(239,68,68,0.40); }
```

### Form inputs

```css
input[type="text"], input[type="email"], input[type="search"], textarea, select {
  width: 100%;
  padding: 10px 14px;
  background: var(--input-bg);
  border: 0.5px solid var(--border);
  border-radius: var(--r);
  color: var(--text);
  font-family: var(--font);
  font-size: 15px;
  letter-spacing: -0.005em;
  transition: border-color 0.18s ease, box-shadow 0.18s ease;
}
input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(0,240,255,0.18);
}
input::placeholder { color: var(--text3); }
```

**Focus ring color = accent.** Easiest place to reinforce brand.

### Checkboxes — custom, square, accent-filled

```html
<label class="checkbox-row">
  <input type="checkbox">
  <span class="checkbox-box"></span>
  <span class="checkbox-label">Item label</span>
</label>
```

```css
.checkbox-row { display: flex; align-items: center; gap: 10px; cursor: pointer; }
.checkbox-row input[type="checkbox"] { display: none; }
.checkbox-box {
  width: 22px; height: 22px;
  border: 1.5px solid var(--border-strong);
  border-radius: 6px;
  background: transparent;
  flex-shrink: 0;
  position: relative;
  transition: background 0.15s ease, border-color 0.15s ease;
}
.checkbox-row input:checked + .checkbox-box { background: var(--accent); border-color: var(--accent); }
.checkbox-row input:checked + .checkbox-box::after {
  content: '';
  position: absolute;
  left: 6px; top: 2px;
  width: 6px; height: 11px;
  border-right: 2px solid #001114;
  border-bottom: 2px solid #001114;
  transform: rotate(45deg);
}
```

For checklists (data collection, inventory) this gives a weighted, tappable feel.

---

## 5. Layout patterns — desktop & mobile

### App shell — fluid, Apple-style

The whole app uses a single fluid layout that scales gracefully from 360px phones to 4K monitors. Don't hard-code a `max-width: 1180px` and stop — let the shell breathe on wide displays.

```css
.app-shell {
  --shell-max:  1180px;
  --shell-padx: 24px;
  max-width: var(--shell-max);
  margin: 0 auto;
  padding: 0 var(--shell-padx);
}

/* Tier 1: phones */
@media (max-width: 520px)  { .app-shell { --shell-padx: 16px; } }

/* Tier 2: tablets / small laptops (default) */

/* Tier 3: large desktops */
@media (min-width: 1440px) { .app-shell { --shell-max: 1320px; --shell-padx: 32px; } }

/* Tier 4: very large desktops */
@media (min-width: 1920px) { .app-shell { --shell-max: 1480px; --shell-padx: 48px; } }

/* Tier 5: 4K and above */
@media (min-width: 2560px) { .app-shell { --shell-max: 1680px; --shell-padx: 64px; } }
```

### Hero / display text — fluid scale

```css
.hero-title {
  font-size: clamp(28px, 5.5vw, 52px);
  font-weight: 600;
  letter-spacing: -0.025em;
}
```

`clamp(min, preferred, max)` is your friend — text that scales smoothly with viewport, capped at sensible bounds.

### Two-column layout that collapses to single column on mobile

```css
.layout {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 24px;
}
@media (max-width: 760px) {
  .layout { grid-template-columns: 1fr; }
  .sidebar {
    /* Becomes a slide-in drawer on mobile — see section 6 */
    position: fixed;
    top: calc(52px + env(safe-area-inset-top));
    left: 0; bottom: env(safe-area-inset-bottom);
    width: 88vw; max-width: 320px;
    transform: translateX(-100%);
    transition: transform 0.3s cubic-bezier(0.4,0,0.2,1);
    z-index: 150;
  }
  .sidebar.open { transform: translateX(0); }
}
```

### Sticky nav bar

```html
<nav>
  <div class="nav-left">
    <button class="icon-btn menu-toggle" id="sidebarToggle">☰</button>
    <div class="logo">...</div>
  </div>
  <div class="nav-right">
    <button class="icon-btn" id="refreshBtn">⟳</button>
    <button class="icon-btn" id="menuBtn">⋯</button>
  </div>
</nav>
```

```css
nav {
  position: sticky; top: 0; z-index: 200;
  padding-top: env(safe-area-inset-top);
  padding-left: calc(16px + env(safe-area-inset-left));
  padding-right: calc(16px + env(safe-area-inset-right));
  height: calc(52px + env(safe-area-inset-top));
  background: var(--nav-bg);
  backdrop-filter: saturate(180%) blur(20px);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 0.5px solid var(--border);
  display: flex;
  align-items: flex-end;  /* CRITICAL — see note below */
  box-sizing: border-box;
}
.nav-left, .nav-right { height: 52px; display: flex; align-items: center; }
.nav-left  { gap: 14px; flex: 1; min-width: 0; }
.nav-right { gap: 8px; }
```

**CRITICAL: `align-items: flex-end` on the nav.** The nav height is `52px + env(safe-area-inset-top)` — on iPhones with Dynamic Island the inset is ~59px. If you align items to `center`, they land in the vertical middle (~55px from top) — right where the Dynamic Island sits, eating tap targets. `flex-end` pushes items to the bottom 52px portion below the safe area, where they're tappable.

---

## 6. Modal & overlay patterns

### The "frosted glass" overlay — iOS Control Center style

When a modal or drawer opens, the page content behind it should **blur and dim** while the modal itself stays crisp. NOT the other way around.

```css
.modal-overlay {
  position: fixed; inset: 0; z-index: 700;
  background: rgba(0, 0, 0, 0.55);
  backdrop-filter: saturate(160%) blur(22px);
  -webkit-backdrop-filter: saturate(160%) blur(22px);
  opacity: 0; pointer-events: none;
  transition: opacity 0.28s ease;
}
.modal-overlay.open { opacity: 1; pointer-events: all; }

.modal {
  position: fixed;
  top: 50%; left: 50%;
  transform: translate(-50%, -46%) scale(0.97);
  width: min(92vw, 460px);
  max-height: 88vh;
  background: var(--card-bg);  /* solid, opaque */
  border: 0.5px solid var(--border);
  border-radius: 22px;
  box-shadow: var(--shadow-lg);
  z-index: 701;
  opacity: 0; pointer-events: none;
  transition: opacity 0.25s ease, transform 0.32s cubic-bezier(0.32, 0.72, 0, 1);
  padding: 26px;
}
.modal.open {
  transform: translate(-50%, -50%) scale(1);
  opacity: 1; pointer-events: all;
}
```

The `saturate(160%)` is the small touch that makes it feel like Apple's frosted glass vs. just "dark + blurred." Without it, dimmed page content goes flat-gray. With it, color hints bleed through.

### Generic modal show/hide with focus management

```js
function showModal({ overlayId, modalId, title, bodyHtml, actions, onClose }) {
  if (document.getElementById(overlayId)) return;  // dedupe
  
  const overlay = document.createElement('div');
  overlay.id = overlayId;
  overlay.className = 'modal-overlay';
  document.body.appendChild(overlay);

  const modal = document.createElement('div');
  modal.id = modalId;
  modal.className = 'modal';
  modal.setAttribute('role', 'dialog');
  modal.setAttribute('aria-modal', 'true');
  modal.innerHTML = `
    <button class="modal-close" aria-label="Close">×</button>
    <h3>${escapeHTML(title)}</h3>
    <div class="modal-body">${bodyHtml}</div>
    <div class="modal-actions">${
      (actions || []).map((a, i) => 
        `<button class="btn btn-${a.kind || 'primary'}" data-action="${i}">${escapeHTML(a.label)}</button>`
      ).join('')
    }</div>
  `;
  document.body.appendChild(modal);

  requestAnimationFrame(() => {
    overlay.classList.add('open');
    modal.classList.add('open');
  });

  const prevFocus = document.activeElement;
  
  const close = () => {
    overlay.classList.remove('open');
    modal.classList.remove('open');
    document.removeEventListener('keydown', onKey);
    setTimeout(() => { overlay.remove(); modal.remove(); }, 300);
    if (prevFocus) try { prevFocus.focus(); } catch (_) {}
    if (onClose) onClose();
  };

  const onKey = (e) => { if (e.key === 'Escape') { e.preventDefault(); close(); } };
  document.addEventListener('keydown', onKey);
  overlay.addEventListener('click', close);
  modal.querySelector('.modal-close').addEventListener('click', close);

  modal.addEventListener('click', (e) => {
    const btn = e.target.closest('button[data-action]');
    if (!btn) return;
    const action = actions[parseInt(btn.dataset.action, 10)];
    if (action) {
      if (!action.keepOpen) close();
      try { action.onClick && action.onClick(); } catch (e) { console.warn(e); }
    }
  });

  // Auto-focus the primary action
  setTimeout(() => {
    const first = modal.querySelector('button[data-action="0"]');
    if (first) first.focus();
  }, 80);
}
```

### Drawer pattern (side panel) — both sides

Left-side drawer (typically navigation/sidebar) and right-side drawer (typically settings/menu). Both use the same CSS pattern, just different `transform` directions.

```css
.menu-drawer {
  position: fixed;
  top: 0; right: 0; bottom: 0;
  z-index: 401;
  width: min(360px, 90vw);
  background: var(--bg2);
  border-left: 0.5px solid var(--border);
  box-shadow: -8px 0 48px rgba(0,0,0,0.30);
  transform: translateX(100%);
  transition: transform 0.35s cubic-bezier(0.4,0,0.2,1);
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-right: env(safe-area-inset-right);
}
.menu-drawer.open { transform: translateX(0); }

.menu-overlay {
  position: fixed; inset: 0; z-index: 400;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: saturate(160%) blur(22px);
  -webkit-backdrop-filter: saturate(160%) blur(22px);
  opacity: 0; pointer-events: none;
  transition: opacity 0.3s ease;
}
.menu-overlay.open { opacity: 1; pointer-events: all; }
```

### Swipe-to-open/close gestures

Mobile-first: users swipe from the screen edge to open drawers. Inside an open drawer, swipe toward its edge to close.

```js
const EDGE_THRESHOLD_PX = 24;
const COMMIT_THRESHOLD_RATIO = 0.4;
const FLICK_VELOCITY_PX_MS = 0.4;

const _swipeState = {
  active: false, target: null, direction: null,
  startX: 0, startY: 0, startTime: 0, lastX: 0, lastTime: 0,
  drawerWidth: 0, drawerEl: null, overlayEl: null,
};

document.addEventListener('touchstart', (e) => {
  if (e.touches.length !== 1) return;
  const t = e.touches[0];
  // ... detect edge-swipe-to-open OR drawer-swipe-to-close
}, { passive: true });

document.addEventListener('touchmove', (e) => {
  if (!_swipeState.active) return;
  const t = e.touches[0];
  const dx = t.clientX - _swipeState.startX;
  const dy = t.clientY - _swipeState.startY;
  // Cancel if gesture is clearly vertical (>1.5x ratio of Y to X)
  if (Math.abs(dy) > Math.abs(dx) * 1.5 && Math.abs(dy) > 12) {
    cancelSwipe();
    return;
  }
  // Add .dragging class to suspend CSS transitions, follow finger 1:1
  // Update drawer transform inline based on progress
}, { passive: true });

document.addEventListener('touchend', () => {
  if (!_swipeState.active) return;
  // Two commit conditions:
  // 1. Distance: dx > 40% of drawer width
  // 2. Velocity: dx > 0.4 px/ms (flick)
}, { passive: true });
```

`.dragging` class on the drawer suspends the CSS transition so the finger 1:1 tracks the drawer:

```css
.menu-drawer.dragging,
.sidebar.dragging { transition: none !important; }
```

### Confirmation dialogs — appConfirm pattern

Simple yes/no prompts shouldn't reinvent the modal. Have a one-liner:

```js
async function appConfirm({ title, message, confirmLabel = 'Confirm', cancelLabel = 'Cancel', variant = 'default' }) {
  return new Promise((resolve) => {
    showModal({
      overlayId: 'appConfirmOverlay',
      modalId: 'appConfirmModal',
      title,
      bodyHtml: `<p class="confirm-msg">${escapeHTML(message)}</p>`,
      actions: [
        { label: confirmLabel, kind: variant === 'danger' ? 'danger' : 'primary', onClick: () => resolve(true) },
        { label: cancelLabel, kind: 'ghost', onClick: () => resolve(false) },
      ],
      onClose: () => resolve(false),  // close without action = cancel
    });
  });
}

// Usage:
const ok = await appConfirm({
  title: 'Sign out?',
  message: 'Your progress is saved locally and will be restored next sign-in.',
  confirmLabel: 'Sign out',
  cancelLabel: 'Stay signed in',
});
if (ok) { /* proceed */ }
```

---

## 7. State management

### One global `state` object

```js
function defaultState() {
  return {
    // Identity
    username: '',
    userProfile: null,  // loaded from cloud after login

    // App state
    items: {},          // entity-keyed dictionary
    currentItemId: null,

    // Preferences
    theme: 'dark',
    
    // Sync metadata
    _lastSyncAt: null,
    _terminalLockUntil: null,
  };
}

let state = defaultState();
```

Single global, mutated directly. No Redux, no immutability. Trust yourself to be careful and to call `saveState()` after mutations.

### Per-username localStorage keys

**Critical**: don't use a single global localStorage key for state. If User A and User B share a device, they overwrite each other's work.

```js
function sessionKeyFor(username) {
  return `appname_session_${username.toLowerCase()}`;
}

function saveState() {
  if (!state.username) return;
  try {
    localStorage.setItem(sessionKeyFor(state.username), JSON.stringify(state));
  } catch (e) { console.warn('saveState failed:', e); }
  
  // Trigger cloud sync (debounced — see section 9)
  if (typeof scheduleSessionStateSync === 'function') scheduleSessionStateSync();
}

function loadState(username) {
  try {
    const raw = localStorage.getItem(sessionKeyFor(username));
    if (!raw) return null;
    return JSON.parse(raw);
  } catch (e) { return null; }
}

// At login:
const saved = loadState(orbitId);
if (saved && saved.username === orbitId) {
  state = saved;
} else {
  state = defaultState();
  state.username = orbitId;
}
```

### Auto-resume vs. fresh login

You want different behavior on:
- **Fresh login** (user typed credentials) — trigger welcome modals, check assignments, etc.
- **Auto-resume** (page refresh, returning user) — just restore silently

Distinguish via a `LAST_LOGIN_KEY` pointer:

```js
const LAST_LOGIN_KEY = 'appname_last_login';

function setLastLoginUsername(username) {
  try { localStorage.setItem(LAST_LOGIN_KEY, username); } catch (e) {}
}

function getLastLoginUsername() {
  try { return localStorage.getItem(LAST_LOGIN_KEY); } catch (e) { return null; }
}

// On page load:
const lastUser = getLastLoginUsername();
if (lastUser) {
  // Auto-resume — restore state without prompting
  const saved = loadState(lastUser);
  if (saved) state = saved;
  startApp();  // skip welcome modal
} else {
  // Show login screen
}
```

### `renderApp()` — idempotent UI rebuild

Have ONE function that, given current state, builds the visible UI. Call it after every state change.

```js
function renderApp() {
  document.getElementById('app').innerHTML = '';
  
  if (!state.username) return;  // not logged in

  // Render each section by calling specialized functions
  renderNav();
  renderSidebar();
  renderMainContent();
  renderFooter();
}
```

This is "dumb but reliable" — no virtual DOM diffing, just rebuild. For an app this size, it's fast enough and dramatically simpler than React.

### Optional: cached fragments for hot paths

If `renderApp()` becomes too slow (only happens if you have lots of DOM), cache individual fragments by state hash and only rebuild when state changes:

```js
let _lastNavHash = null;
function renderNav() {
  const hash = JSON.stringify([state.username, state.currentItemId]);
  if (hash === _lastNavHash) return;  // already current
  _lastNavHash = hash;
  // ... rebuild nav HTML
}
```

---

## 8. Power Automate + Excel backend

### Architecture overview

```
┌──────────┐         ┌────────────────┐         ┌───────────────┐
│  Browser │  HTTPS  │ Power Automate │  Graph  │ Excel on      │
│   App    │ ──────► │ flows (URLs)   │ ──────► │ SharePoint    │
│          │ ◄────── │                │ ◄────── │               │
└──────────┘         └────────────────┘         └───────────────┘
```

Each Excel table gets TWO Power Automate flows:
- **Write flow** — receives JSON POST → inserts/updates a row in the Excel table
- **Read flow** — receives empty POST → returns all rows as JSON

The browser calls the flow URLs directly. Authentication is baked into the URL via SAS-style signatures.

### Excel table schema conventions

**All columns are Text type.** Even dates, even numbers, even booleans. Why: Excel's typed columns cause subtle issues with PA flows (date format mismatch, empty cells becoming `null` vs `''`). Text columns are bulletproof. Parse to typed values on the client.

**Every table has a unique ID column.** Convention: `<entity>_id` (e.g., `assignment_id`, `worklog_id`, `inventory_item_id`). Generated client-side as `<prefix>_<userId>_<entityId>_<timestamp>`.

```js
function generateId(prefix, userId, entityId) {
  const ts = Date.now();
  return `${prefix}_${userId}_${entityId}_${ts}`;
}
// Example: "ss_jane.smith_asgn123_1734567890123"
```

### Standard table set for a moderator+admin+inventory app

```
Moderators       — moderator directory (username, name, email, role)
Assignments      — bookings (assignment_id, mod, participant, date, time, status)
WorkLog          — status events (worklog_id, assignment_id, status, timestamp, notes)
SessionState     — live session data for cross-device sync (state_id, user, asgn, json, last_active)
Availability     — moderator availability per week (avail_id, user, week_start, days_json)
Inventory        — items + counts (inventory_id, item_name, sku, current_qty, location, status)
InventoryLog     — inventory transactions (txn_id, inventory_id, delta, reason, user, timestamp)
DeleteLog        — soft-delete records (delete_id, table, row_id, timestamp, user)
```

### Flow URL constants — wire them at the top of the JS

```js
// === Power Automate flow URLs ===
// All Excel-table operations go through PA flows. Pairs of read/write
// URLs are wired here; client code uses the named constants only.
const MODERATORS_PA_READ_URL    = 'https://...';
const ASSIGNMENT_PA_READ_URL    = 'https://...';
const ASSIGNMENT_PA_WRITE_URL   = 'https://...';
const WORKLOG_PA_WRITE_URL      = 'https://...';
const WORKLOG_PA_READ_URL       = '';  // empty = not configured yet
const SESSIONSTATE_PA_WRITE_URL = 'https://...';
const SESSIONSTATE_PA_READ_URL  = 'https://...';
const INVENTORY_PA_READ_URL     = 'https://...';
const INVENTORY_PA_WRITE_URL    = 'https://...';
```

Use `''` (empty string) for flows that haven't been built yet. Code defensively:

```js
if (!INVENTORY_PA_READ_URL) {
  console.warn('Inventory read URL not configured');
  return [];
}
```

### Write pattern

```js
async function writeRow(url, row) {
  if (!url) return false;
  try {
    const res = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(row),
    });
    if (!res.ok) {
      console.warn(`Write failed: HTTP ${res.status}`);
      return false;
    }
    return true;
  } catch (e) {
    console.warn('Write threw:', e.message);
    return false;
  }
}
```

### Read pattern with row extraction

PA flows return rows in different envelopes depending on action used:

```js
function extractArray(data) {
  if (!data) return [];
  if (Array.isArray(data)) return data;
  if (Array.isArray(data.value)) return data.value;
  if (Array.isArray(data.rows)) return data.rows;
  return [];
}

async function readRows(url) {
  if (!url) return [];
  const res = await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({}),
  });
  if (!res.ok) return [];
  const text = await res.text();
  let data;
  try { data = JSON.parse(text); } catch (e) { return []; }
  return extractArray(data);
}
```

### 256-row pagination gotcha

PA flow actions that return Excel rows cap at 256 by default. If your read returns exactly 256 rows, it's almost certainly truncated. The fix:

**Option A** (preferred): inside the PA flow, set pagination on the "List rows" action — Settings → Pagination → On, threshold 5000. This makes the flow return everything.

**Option B**: implement client-side pagination — call repeatedly with skip/take, accumulate rows.

Add a defensive log:

```js
const rows = await readRows(url);
if (rows.length === 256) {
  console.warn('[App] Possible pagination ceiling hit — got exactly 256 rows. Check PA flow pagination settings.');
}
```

### Concurrent writes — soft-delete + last-write-wins

Multiple admins editing the same data on different devices will produce conflicts. The Orbit approach:

1. **Hard deletes are forbidden.** When an admin "deletes" a row, write a record to the `DeleteLog` table instead.
2. **Reads merge live rows minus deleted IDs.** Client reads both `Assignments` and `DeleteLog`, filters out any assignment whose ID appears in DeleteLog.
3. **Updates use latest-timestamp-wins.** Each write includes a timestamp. When merging multiple writes for the same row, the one with the latest timestamp wins.

```js
async function fetchAssignmentsWithDeletes() {
  const [rows, deletes] = await Promise.all([
    readRows(ASSIGNMENT_PA_READ_URL),
    readRows(DELETELOG_PA_READ_URL),
  ]);
  const deletedIds = new Set(deletes.filter(d => d.table === 'Assignments').map(d => d.row_id));
  return rows
    .filter(r => !deletedIds.has(r.assignment_id))
    .sort((a, b) => (b.last_modified || '').localeCompare(a.last_modified || ''));
}
```

### Status-event log (immutable history)

For state that changes frequently (assignment status: Pending → InProgress → Complete; inventory: in-stock → checked-out → returned), don't UPDATE rows. Instead, APPEND status events to a log table.

```
WorkLog columns: worklog_id | assignment_id | status | timestamp | user | notes
```

Each status change = one new row. To get the "current status," read all rows for that assignment, sort by timestamp, take the latest.

**Why**: gives you full audit history, no merge conflicts (writes never conflict because they're always inserts), and you can replay state to any point in time.

---

## 9. Cloud-first refresh architecture

**The mistake to avoid**: making the mod app read from `localStorage` and only `localStorage`. localStorage is a per-device cache. If admin makes a change on Browser A, the mod on Browser B never sees it. This breaks badly on first-login-on-new-device — the mod sees an empty app.

**The correct pattern**: cloud is the source of truth, localStorage is the offline fallback.

### The refresh helper

Wrap your cloud fetch in a helper with concurrency guards:

```js
const REFRESH_INTERVAL_MS  = 180_000;   // 3 min periodic refresh
const STALE_THRESHOLD_MS   = 90_000;    // 90s = stale on visibility return

let _fetchPromise = null;     // dedupe in-flight fetches
let _lastFetchAt  = 0;
let _refreshTimer = null;

async function refreshFromCloud(opts = {}) {
  // Dedupe: if a fetch is in flight, return its promise rather than firing parallel
  if (_fetchPromise && !opts.force) return _fetchPromise;
  
  _fetchPromise = (async () => {
    try {
      // Fetch from cloud, merge with delete log, persist to localStorage + state
      await fetchEverythingFromPA();
      _lastFetchAt = Date.now();
      
      // Re-render any visible UI that depends on this data
      if (typeof renderApp === 'function') renderApp();
      return true;
    } catch (e) {
      console.warn('Cloud refresh failed:', e.message);
      return false;
    } finally {
      _fetchPromise = null;
    }
  })();
  return _fetchPromise;
}
```

### Five refresh triggers

| Trigger | When | Behavior |
|---|---|---|
| **Login** | Every fresh login | Fires immediately, welcome modal awaits it (capped at 2.5s) |
| **Manual** | User taps refresh button | `force: true`, bypasses dedupe, shows toast on completion |
| **Open a view** | User opens calendar/inventory list | Background refresh; view re-renders when data lands |
| **Periodic** | Every 3 min while app visible | Silent; skipped when tab hidden |
| **Visibility return** | Tab refocused after >90s away | Stale-check; refresh if cache aged out |

```js
function startBackgroundRefresh() {
  if (_refreshTimer) return;
  _refreshTimer = setInterval(() => {
    if (document.visibilityState !== 'visible') return;
    refreshFromCloud({ silent: true });
  }, REFRESH_INTERVAL_MS);

  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState !== 'visible') return;
    const age = Date.now() - _lastFetchAt;
    if (age > STALE_THRESHOLD_MS) refreshFromCloud({ silent: true });
  });
}

function stopBackgroundRefresh() {
  if (_refreshTimer) { clearInterval(_refreshTimer); _refreshTimer = null; }
}
```

### Per-user session sync (cross-device)

For state that needs to survive device switches (user starts work on phone, continues on tablet), write a syncable subset to cloud:

```js
function extractSyncableState(s) {
  if (!s) return {};
  return {
    currentItemId:      s.currentItemId      || '',
    items:              s.items              || {},
    sessionCompletedAt: s.sessionCompletedAt || null,
    // Only include cross-device-meaningful fields; skip UI prefs.
  };
}

const SYNC_DEBOUNCE_MS = 5000;
const _syncState = { timer: null, inflight: false, pendingAgain: false };

function scheduleSync() {
  if (!SESSIONSTATE_PA_WRITE_URL) return;
  if (_syncState.inflight) { _syncState.pendingAgain = true; return; }
  if (_syncState.timer) clearTimeout(_syncState.timer);
  _syncState.timer = setTimeout(flushSync, SYNC_DEBOUNCE_MS);
}

async function flushSync() {
  _syncState.timer = null;
  if (!state.username) return;
  const syncable = extractSyncableState(state);
  const id = `ss_${state.username}_${state.currentItemId}_${Date.now()}`;
  _syncState.inflight = true;
  try {
    await fetch(SESSIONSTATE_PA_WRITE_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        state_id: id,
        user: state.username,
        item_id: state.currentItemId,
        state_json: JSON.stringify(syncable),
        last_active: new Date().toISOString(),
      }),
    });
  } finally {
    _syncState.inflight = false;
    if (_syncState.pendingAgain) { _syncState.pendingAgain = false; scheduleSync(); }
  }
}
```

### `beforeunload` flush via sendBeacon

When the user closes the tab, regular `fetch()` calls get cancelled. Use `navigator.sendBeacon` for last-gasp writes:

```js
window.addEventListener('beforeunload', () => {
  if (!state.username || !SESSIONSTATE_PA_WRITE_URL) return;
  const id = `ss_${state.username}_${state.currentItemId}_${Date.now()}`;
  const payload = JSON.stringify({
    state_id: id,
    user: state.username,
    item_id: state.currentItemId,
    state_json: JSON.stringify(extractSyncableState(state)),
    last_active: new Date().toISOString(),
  });
  navigator.sendBeacon(SESSIONSTATE_PA_WRITE_URL, payload);
});
```

`sendBeacon` is fire-and-forget — the browser guarantees the request goes out even after page unload.

---

## 10. Mobile-specific patterns (iOS + Android)

### Universal mobile: target both iOS Safari and Android Chrome

The two browsers behave similarly enough that one codebase can serve both. Specific tactics:

### Viewport — required meta tag

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="theme-color" content="#0f0f0e">
```

`viewport-fit=cover` lets your app draw under the notch / status bar. You MUST then handle safe-area insets explicitly.

### Safe-area insets — for notches, status bars, home indicators

```css
nav {
  padding-top: env(safe-area-inset-top);
  padding-left: calc(16px + env(safe-area-inset-left));
  padding-right: calc(16px + env(safe-area-inset-right));
  height: calc(52px + env(safe-area-inset-top));
}
.fixed-bottom-bar {
  padding-bottom: env(safe-area-inset-bottom);
}
.drawer {
  bottom: env(safe-area-inset-bottom);
}
```

**These env() values are 0 on non-notched devices.** No conditional logic needed — write the insets in everywhere, they cost nothing on Android phones without cutouts.

### Tap targets — 40-44px minimum

iOS HIG recommends 44×44px. Android Material is 48×48dp. Use 40px as a minimum, 44px when possible. Icon buttons on mobile especially need this — desktop's 32-34px is too small for thumbs.

### Touch-action — prevent unwanted gestures

```css
.no-pinch-zoom { touch-action: pan-y; }
button, .btn { touch-action: manipulation; }  /* removes 300ms tap delay */
```

### iOS Safari quirks

**iOS Safari is stricter on CSS parsing** — a single unclosed brace cascades catastrophically. Always verify CSS brace balance before ship.

**Orientation lock requires PWA install.** `screen.orientation.lock('portrait')` only works when the app is added to the home screen. In a regular Safari tab it throws. Fallback: detect landscape via `matchMedia('(orientation: landscape)')` and show a "Rotate to portrait" overlay if the user has the lock preference enabled.

```js
async function tryLockPortrait() {
  if (!screen.orientation || !screen.orientation.lock) return false;
  try {
    await screen.orientation.lock('portrait');
    return true;
  } catch (e) {
    // iOS Safari throws here — fall back to CSS overlay
    return false;
  }
}

function updateOrientationOverlay() {
  const enabled = getOrientationLockEnabled();
  if (!enabled) {
    document.getElementById('orientationOverlay').classList.remove('active');
    return;
  }
  const isLandscape = window.matchMedia('(orientation: landscape)').matches;
  document.getElementById('orientationOverlay').classList.toggle('active', isLandscape);
}

window.matchMedia('(orientation: landscape)').addEventListener('change', updateOrientationOverlay);
```

### Android quirks

**Android Chrome respects `screen.orientation.lock` even in tabs** — no PWA install required. Your iOS fallback overlay just won't fire on Android.

**The Android keyboard pushes the viewport up.** Use `dvh` (dynamic viewport height) instead of `vh` for full-height elements that need to stay visible above the keyboard:

```css
.full-height { height: 100dvh; }  /* respects keyboard, address bar */
```

**Android's pull-to-refresh can interfere** with custom scroll behavior. Disable it on full-app containers:

```css
body { overscroll-behavior-y: contain; }
```

### iPhone Dynamic Island — align to bottom

If the nav uses `align-items: center` with safe-area-inset-top padding, icons land in the Dynamic Island zone and become un-tappable. Solution:

```css
nav {
  height: calc(52px + env(safe-area-inset-top));
  align-items: flex-end;  /* icons sit in the bottom 52px, below the island */
}
.nav-left, .nav-right {
  height: 52px;
  display: flex; align-items: center;
}
```

### PST/timezone handling

For team-wide reference dates (NOT each user's local time), use an explicit timezone:

```js
function getPSTDateString() {
  try {
    return new Intl.DateTimeFormat('en-CA', {
      timeZone: 'America/Los_Angeles',
      year: 'numeric', month: '2-digit', day: '2-digit',
    }).format(new Date());
  } catch (e) {
    return new Date().toISOString().slice(0, 10);
  }
}
```

`Intl.DateTimeFormat` auto-handles DST shifts (PST ↔ PDT). The `'en-CA'` locale conveniently produces YYYY-MM-DD format directly.

### Midnight watcher pattern

When the workday rolls over, you may need to reset session state, refresh assignments, re-trigger welcome modals. Use a polling tick + visibilitychange listener:

```js
let _lastSeenDate = null;
let _midnightTimer = null;

function startMidnightWatcher() {
  if (_midnightTimer) return;
  _lastSeenDate = getPSTDateString();
  _midnightTimer = setInterval(() => {
    const now = getPSTDateString();
    if (now !== _lastSeenDate) {
      _lastSeenDate = now;
      handleMidnightTransition();
    }
  }, 60_000);

  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState !== 'visible') return;
    const now = getPSTDateString();
    if (now !== _lastSeenDate) {
      _lastSeenDate = now;
      handleMidnightTransition();
    }
  });
}
```

Visibilitychange catches phones that slept past midnight and missed the polling ticks.

---

## 11. Login flow & welcome modals

### Identity validation against directory

```js
async function doLogin(username) {
  setLoginLoading(true);
  try {
    const directory = await readRows(MODERATORS_PA_READ_URL);
    const orbitId = canonicalize(username);
    const profile = directory.find(m => 
      canonicalize(m.orbit_login_id) === orbitId
    );
    if (!profile) {
      shakeLoginCard();
      setLoginError("Username not recognized. Check spelling and try again.");
      return;
    }

    // Load saved state for THIS user (per-username localStorage)
    const saved = loadState(orbitId);
    if (saved && saved.username === orbitId) {
      state = saved;
    } else {
      state = defaultState();
      state.username = orbitId;
    }
    state.userProfile = profile;
    saveState();
    setLastLoginUsername(orbitId);

    _loginWelcomePending = true;  // distinguishes fresh login from auto-resume
    startApp();
    
    // Welcome modal after the initial cloud refresh resolves
    setTimeout(async () => {
      if (!_loginWelcomePending) return;
      _loginWelcomePending = false;
      try {
        await Promise.race([
          window._initialRefresh || Promise.resolve(),
          new Promise(r => setTimeout(r, 2500)),  // safety timeout
        ]);
        checkLoginWelcome();
      } catch (e) {
        try { checkLoginWelcome(); } catch (_) {}
      }
    }, 900);
  } catch (e) {
    setLoginError("Couldn't reach the directory. Check your connection.");
  } finally {
    setLoginLoading(false);
  }
}
```

### Welcome modal variants

Three states drive three different modals:

```js
function determineWelcomeState() {
  if (!state.userProfile) return null;
  const today = getPSTDateString();

  // Variant 1: today's session is already complete
  if (state.sessionCompletedAt && state.sessionDate === today) {
    return { kind: 'today_completed' };
  }

  // Variant 2: has an upcoming assignment
  const upcoming = getMyUpcomingAssignments(today);
  if (upcoming.length > 0) {
    return { kind: 'next_session', assignment: upcoming[0] };
  }

  // Variant 3: no assignments at all
  return { kind: 'no_session' };
}

function checkLoginWelcome() {
  const w = determineWelcomeState();
  if (!w) return;
  if (w.kind === 'today_completed')  showTodayCompletedModal(w);
  else if (w.kind === 'next_session') showNextSessionModal(w);
  else if (w.kind === 'no_session')   showNoSessionModal();
}
```

Each modal has its own action set:

- **today_completed**: [Review the session] [Start a new session] [Sign out]
- **next_session**: [Got it] with a summary block (participant, address, team, time)
- **no_session**: [Continue to app] pointing user to availability calendar

### Auto-resume vs fresh login

The `_loginWelcomePending` flag is set TRUE in `doLogin()` and consumed by the welcome timer. Auto-resume on page refresh skips `doLogin` entirely, so the flag stays FALSE and no welcome modal fires. Important: this prevents the modal from re-prompting every time the user refreshes the page.

---

## 12. Commenting & verification discipline

### Comment density — high

Inline comments are mandatory on:
- Why a specific value was chosen (`/* 22px blur matches iOS Control Center */`)
- Anything non-obvious about ordering or timing (`/* must come before X because... */`)
- Workarounds for browser quirks (`/* iOS Safari requires this prefix */`)
- Trade-offs that future-you might want to reconsider
- Bug fixes (mention what the bug was)

Example from production code:

```js
// CRITICAL: align-items must be `flex-end` (bottom), NOT `center`.
// With `center`, the icons sit in the vertical middle of the full
// nav height (which includes the safe-area inset padding) — which
// on iPhones with Dynamic Island puts them BEHIND or overlapping
// the Island, making them visually obscured (Amz chip got cut in
// half) and hard to tap.
```

This is verbose by typical code-style standards. It's intentional. Future-you (or the next dev, or Claude in 6 months) needs to know WHY, not just WHAT.

### Section comments

Use big visual banners to mark sections in long files:

```js
/* =====================================================================
   CLOUD-FIRST ASSIGNMENT DATA
   =====================================================================
   The mod app's view of "my schedule" / "my next session" used to read
   ONLY from localStorage, which was populated by the admin app on the
   same device. On a fresh device where the mod has never opened the
   admin app, localStorage was empty → the mod saw "no booked session"
   even when they HAD bookings.
   
   This module fixes that by treating the cloud as source of truth
   and localStorage as offline fallback.
*/
```

In a 24K-line file, you literally search with Cmd+F for these banners to navigate. Make them findable.

### Diagnostic console logs

Log key state at decision points so the next bug report can be diagnosed from the user's console:

```js
console.log(
  '[App] Progress summary —',
  'stations:', stationsDone, '/', stationsTotal,
  '· scenarios:', scenariosDone, '/', scenariosTotal,
  '(from user:', userId, ')'
);
```

Prefix with `[App]` or your app name so logs are filterable.

### Test snippets

For non-trivial logic (counting, date math, status resolution), write a small Node.js test that doesn't need the DOM:

```js
// test-counting.js
function isComplete(item) { /* same logic as in app.js */ }
const cases = [
  { items: {...}, expected: 3 },
  { items: {...}, expected: 0 },
];
cases.forEach((c, i) => {
  const got = countComplete(c.items);
  console.log(`Case ${i}: ${got === c.expected ? '✓' : '❌'} got ${got}, expected ${c.expected}`);
});
```

Run with `node test-counting.js`. Keep these in a `/test` folder. They protect you from regressions when refactoring.

---

## 13. Anti-patterns — what NOT to do

### ❌ Single global localStorage key for state
Two users on the same device overwrite each other's work. Use `appname_session_<username>`.

### ❌ Trusting `align-items: center` in the nav
On iPhones with Dynamic Island, items center into the Island zone. Use `flex-end`.

### ❌ Skipping CSS brace verification
You'll ship a broken stylesheet to iOS Safari and not know until a user reports unstyled HTML.

### ❌ Reading status by checking only the latest column on a row
Use a separate WorkLog table with status events. Update-in-place creates merge conflicts.

### ❌ Hard-coded `max-width: 1180px`
The shell should scale up to 4K. Use the tiered `--shell-max` CSS variable pattern.

### ❌ Translucent drawers over the page content
Looks like you're reading text on a busy background. Make drawers OPAQUE, blur the OVERLAY behind them.

### ❌ One welcome popup that always shows
Should be state-aware: completed vs upcoming vs none. Forcing the user through "yes I see I have no bookings" daily is annoying.

### ❌ Treating localStorage as the source of truth for shared data
Localstorage is per-device. Other admins making changes elsewhere are invisible. Cloud is source of truth.

### ❌ `setTimeout(..., 0)` for "after render" effects
Use `requestAnimationFrame()`. setTimeout(0) doesn't actually wait for layout, while `rAF` does.

### ❌ Hard delete from Excel tables
Soft-delete via a DeleteLog table. Otherwise you have no audit trail and concurrent admin edits collide badly.

### ❌ Tooltip-only error reporting
Show errors in a visible alert (toast or inline message). Tooltip-only errors are missed by 80% of users.

### ❌ Mobile dropdowns / select menus
iOS and Android render `<select>` poorly. Build custom popup menus for any list of >3 choices.

### ❌ Aggressive auto-save with no debounce
A typing event firing a cloud write on every keystroke = denial of service against your own PA flow. Always debounce (~3-5s).

### ❌ Forgetting `beforeunload` flushes
Last 5 seconds of edits get lost when the user closes the tab. Use `navigator.sendBeacon` for fire-and-forget last writes.

---

## 14. Quick-start checklist for a new app

When starting a new app in this lineage, work this checklist top-to-bottom:

### Day 1: skeleton

- [ ] Create `app.html` with `<!DOCTYPE html>`, viewport meta with `viewport-fit=cover`, theme-color meta
- [ ] Drop in the full token list (`:root`, `[data-theme="dark"]`, `[data-theme="light"]`)
- [ ] Drop in `.btn`, `.icon-btn`, `.theme-toggle`, `.card`, `.badge`, `.toast` styles
- [ ] Wire `showToast()` function + add the global toast slot to `<body>`
- [ ] Wire theme toggle (`document.documentElement.setAttribute('data-theme', ...)`)
- [ ] Define `state = defaultState()` global + `saveState()` per-username
- [ ] Define `APP_VERSION = '0.1.<MMDDYY>'`
- [ ] Verify CSS brace balance + JS parses
- [ ] Push to GitHub Pages

### Day 2: login + welcome

- [ ] Wire login screen with username input
- [ ] Read MODERATORS table from PA → directory check
- [ ] Per-username localStorage key + auto-resume vs fresh login distinction
- [ ] `LAST_LOGIN_KEY` for auto-resume
- [ ] `_loginWelcomePending` flag for welcome trigger
- [ ] `determineWelcomeState()` returning today_completed / next_session / no_session / null
- [ ] Three welcome modal variants
- [ ] Cloud refresh on login (await before welcome decision)

### Day 3: main app shell

- [ ] Nav bar with safe-area handling + `align-items: flex-end`
- [ ] Sidebar collapsing to drawer on mobile
- [ ] Menu drawer (right side) with theme toggle, logout, etc.
- [ ] Swipe gestures to open both drawers
- [ ] Frosted-glass overlay pattern for drawers + modals
- [ ] `renderApp()` idempotent rebuild function

### Day 4: backend integration

- [ ] PA flow URLs declared at top
- [ ] `readRows(url)` + `writeRow(url, data)` helpers + `extractArray()` envelope handling
- [ ] DeleteLog table + merge-on-read pattern
- [ ] WorkLog (or equivalent status events table) + latest-wins resolution
- [ ] SessionState table for cross-device sync (debounced writes)
- [ ] `beforeunload` flush via `sendBeacon`
- [ ] Refresh helpers: `refreshFromCloud({silent, force})` with concurrency guards

### Day 5: refresh triggers

- [ ] Refresh on login (awaited, with 2.5s safety cap)
- [ ] Refresh on nav refresh button (force, with toast)
- [ ] Refresh on opening calendar/list views
- [ ] 3-minute periodic background refresh (only when visible)
- [ ] Visibility-change refresh if cache > 90s old
- [ ] PST midnight watcher if your app has day boundaries

### Day 6+: feature work

Now you're set up to build the actual feature work — checklists, inventory, admin dashboards — on solid foundation.

### When to deviate from this playbook

- **App needs to be a true SPA with deep routing** → consider a framework, you'll outgrow the single-file approach
- **Backend has more than ~20 tables** → consider a proper API/database, Excel will get unwieldy
- **Team is >50 users** → re-evaluate Power Automate quotas
- **You need real-time push** (sub-second updates) → polling is too slow, you'll want WebSocket / SignalR
- **Desktop-only app** → you can skip the safe-area / swipe / orientation sections, simplify drastically

For anything resembling Orbit's scope (small team, mobile-first, infrequent updates, structured data), this playbook is the right baseline.

---

## Appendix: snippets you'll need often

### `escapeHTML` — safely interpolate user content into HTML strings

```js
function escapeHTML(s) {
  if (s == null) return '';
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}
```

### `canonicalize` — case-insensitive username matching

```js
function canonicalize(s) {
  return String(s || '').trim().toLowerCase();
}
```

### `ymd(date)` — date to YYYY-MM-DD in LOCAL time

```js
function ymd(d) {
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, '0');
  const dd = String(d.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;
}
```

For PST-anchored use `getPSTDateString()` from section 10.

### `mondayOf(date)` — Monday of the date's week

```js
function mondayOf(d) {
  const day = (d.getDay() + 6) % 7;  // Mon=0..Sun=6
  return new Date(d.getFullYear(), d.getMonth(), d.getDate() - day);
}
```

### `addDays(date, n)`

```js
function addDays(d, n) {
  return new Date(d.getFullYear(), d.getMonth(), d.getDate() + n);
}
```

### Inventory-tracking specific: stock-delta journal pattern

For the new app's inventory feature, use the same status-event pattern but as a transaction log:

```
Inventory      — current snapshot (inventory_id | item_name | sku | qty | location | status)
InventoryLog   — transactions      (txn_id | inventory_id | delta | reason | user | timestamp)
```

- Reads: SELECT from Inventory directly. The current snapshot is correct as of last write.
- Writes: append a row to InventoryLog with delta (+5 for receiving, -3 for issued, -1 for adjustment, etc.) AND update the Inventory row's qty by the same delta.
- For audit: reconstruct any past quantity by summing the log up to a point in time.

The InventoryLog is also where you check IN/OUT items by user. Add fields like `checked_out_to`, `expected_return_date`, `actual_return_date`.

```js
async function logInventoryTransaction({ inventoryId, delta, reason, user, notes }) {
  const txnId = `txn_${user}_${inventoryId}_${Date.now()}`;
  await writeRow(INVENTORYLOG_PA_WRITE_URL, {
    txn_id: txnId,
    inventory_id: inventoryId,
    delta: String(delta),
    reason,
    user,
    notes: notes || '',
    timestamp: new Date().toISOString(),
  });
  // Optimistically update local state
  if (state.inventory[inventoryId]) {
    state.inventory[inventoryId].qty = (state.inventory[inventoryId].qty || 0) + delta;
    saveState();
  }
  // Trigger periodic cloud refresh for cross-device consistency
  scheduleSync();
}
```

This pattern gives you: real-time inventory updates, full audit history, the ability to flag discrepancies, and a path to add features like "low stock alerts" later by querying log deltas.
