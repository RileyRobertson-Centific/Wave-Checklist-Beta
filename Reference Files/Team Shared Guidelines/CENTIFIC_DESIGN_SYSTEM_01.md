# Centific Design System
> Paste this file at the start of any new Claude chat to instantly apply the full visual design system used in the Centific Project Readiness app.

---

## How to Use

Upload this file to a new Claude chat and say:

> *"Use the design system in this file for everything you build. Match the visual style, CSS variables, typography, spacing, component patterns, and motion exactly."*

Then describe your app and Claude will generate it in the same visual language.

---

## Brand

- **Company:** Centific
- **Logo:** Transparent PNG (embed as base64 in HTML)
- **Primary color:** `#0071e3` (Apple blue)
- **Style:** Apple-inspired, minimalist, premium

---

## Core Philosophy

Clean white surfaces, generous whitespace, tight letter-spacing, SF Pro Display font stack. Every element earns its place. Inspired by apple.com — less is more, products (content) speak for themselves.

---

## CSS Variables

```css
:root {
  --bg: #f5f5f7;
  --surface: #ffffff;
  --surface2: #fbfbfd;
  --text-primary: #1d1d1f;
  --text-secondary: #6e6e73;
  --text-tertiary: #aeaeb2;
  --accent: #0071e3;
  --accent-light: #e8f1fc;
  --success: #34c759;
  --success-light: #e8f9ed;
  --warning: #f5a623;
  --warning-light: #fff3dc;
  --danger: #ff3b30;
  --border: rgba(0,0,0,0.08);
  --border-strong: rgba(0,0,0,0.14);
  --shadow-sm: 0 2px 12px rgba(0,0,0,0.06);
  --shadow-md: 0 8px 40px rgba(0,0,0,0.10);
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.12);
  --radius: 18px;
  --radius-sm: 12px;
  --font: -apple-system, 'SF Pro Display', 'Helvetica Neue', sans-serif;
}
```

---

## Base Styles

```css
html { -webkit-font-smoothing: antialiased; }
body {
  font-family: var(--font);
  background: var(--bg);
  color: var(--text-primary);
  min-height: 100vh;
}
```

---

## Navigation Bar

Fixed top, frosted glass effect.

```css
nav {
  position: fixed; top: 0; left: 0; right: 0;
  height: 52px;
  background: rgba(245,245,247,0.85);
  backdrop-filter: saturate(180%) blur(20px);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 0.5px solid var(--border);
  display: flex; align-items: center; justify-content: center;
  z-index: 100;
}
.nav-inner {
  width: 100%; max-width: 860px;
  padding: 0 24px;
  display: flex; align-items: center; justify-content: space-between;
}
```

**Nav elements:**
- Logo: `height: 22px; width: auto`
- Project name: `font-size: 15px; font-weight: 600; letter-spacing: -0.3px`
- Client/subtitle: `font-size: 11px; color: var(--text-tertiary)`
- Count pill: `font-size: 13px; background: var(--surface); border: 0.5px solid var(--border-strong); border-radius: 99px; padding: 4px 12px`

---

## Hero Section

```css
.hero {
  padding: 56px 24px 40px;
  max-width: 860px;
  margin: 0 auto;
  text-align: center;
}
.hero-eyebrow {
  font-size: 12px; font-weight: 500;
  letter-spacing: 0.06em; text-transform: uppercase;
  color: var(--accent); margin-bottom: 12px;
}
.hero-title {
  font-size: clamp(30px, 5vw, 48px);
  font-weight: 600; letter-spacing: -0.03em;
  line-height: 1.07; margin-bottom: 12px;
}
.hero-sub {
  font-size: 17px; font-weight: 300;
  color: var(--text-secondary); letter-spacing: -0.01em;
  max-width: 480px; margin: 0 auto 36px; line-height: 1.5;
}
```

---

## Cards

Standard content card used throughout the app.

```css
.card {
  background: var(--surface);
  border-radius: var(--radius);
  border: 0.5px solid var(--border);
  padding: 20px 24px;
  box-shadow: var(--shadow-sm);
  transition: all 0.22s cubic-bezier(0.4,0,0.2,1);
  position: relative; overflow: hidden;
}
.card:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
  border-color: var(--border-strong);
}
```

**Left accent bar (for status):**
```css
.card::before {
  content: ''; position: absolute;
  top: 0; left: 0; bottom: 0; width: 3px;
  border-radius: 0; opacity: 0;
  transition: opacity 0.2s;
}
.card.pending::before    { background: var(--border-strong); opacity: 1; }
.card.inprogress::before { background: var(--warning); opacity: 1; }
.card.done::before       { background: var(--success); opacity: 1; }
.card.skipped::before    { background: var(--text-tertiary); opacity: 1; }
```

---

## Home Action Tiles

Large clickable option cards for home/landing screens.

```css
.home-btn {
  display: flex; align-items: center; gap: 18px;
  background: var(--surface); border: 0.5px solid var(--border-strong);
  border-radius: 20px; padding: 24px 26px;
  cursor: pointer; text-align: left;
  box-shadow: var(--shadow-sm);
  transition: all 0.22s cubic-bezier(0.4,0,0.2,1);
}
.home-btn:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
  border-color: rgba(0,0,0,0.18);
}
.home-btn-icon {
  width: 52px; height: 52px; border-radius: 14px;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
  box-shadow: 0 2px 8px rgba(0,0,0,0.10), inset 0 1px 0 rgba(255,255,255,0.6);
}
/* Color variants */
.home-btn-icon.blue  { background: linear-gradient(145deg, #e8f3fd 0%, #cce3fb 100%); }
.home-btn-icon.green { background: linear-gradient(145deg, #e3f9eb 0%, #c5f0d4 100%); }
.home-btn-icon.amber { background: linear-gradient(145deg, #fff3dc 0%, #ffe9b0 100%); }
.home-btn-icon.red   { background: linear-gradient(145deg, #ffeaea 0%, #ffd0d0 100%); }

.home-btn-label { font-size: 19px; font-weight: 600; letter-spacing: -0.02em; color: var(--text-primary); margin-bottom: 5px; }
.home-btn-desc  { font-size: 15px; color: var(--text-secondary); font-weight: 400; letter-spacing: -0.01em; line-height: 1.4; }
```

---

## Buttons

### Primary Button
```css
.btn-primary {
  font-family: var(--font); font-size: 16px; font-weight: 500;
  letter-spacing: -0.01em; color: white; background: var(--accent);
  border: none; border-radius: 14px; padding: 15px 24px;
  cursor: pointer; transition: all 0.25s cubic-bezier(0.4,0,0.2,1);
  display: flex; align-items: center; justify-content: center; gap: 8px;
}
.btn-primary:hover {
  background: #0077ed; transform: translateY(-1px);
  box-shadow: 0 8px 24px rgba(0,113,227,0.28);
}
.btn-primary:active { transform: scale(0.98); }
```

### Ghost / Secondary Button
```css
.btn-ghost {
  font-family: var(--font); font-size: 13px; font-weight: 500;
  color: var(--text-secondary); background: var(--bg);
  border: 0.5px solid var(--border-strong); border-radius: 10px;
  padding: 8px 16px; cursor: pointer; transition: all 0.18s ease;
  letter-spacing: -0.01em;
}
.btn-ghost:hover { background: #ececec; color: var(--text-primary); }
```

### Pill / Nav Button
```css
.btn-pill {
  font-family: var(--font); font-size: 13px; font-weight: 500;
  color: var(--text-secondary); background: var(--surface);
  border: 0.5px solid var(--border-strong); border-radius: 99px;
  padding: 5px 13px; cursor: pointer; transition: all 0.2s ease;
  display: inline-flex; align-items: center; gap: 6px;
}
.btn-pill:hover { background: var(--text-primary); color: white; border-color: var(--text-primary); }
```

### Danger Button
```css
.btn-danger {
  font-family: var(--font); font-size: 13px; font-weight: 500;
  color: var(--danger); background: none;
  border: 0.5px solid rgba(255,59,48,0.25); border-radius: 10px;
  padding: 8px 16px; cursor: pointer; transition: all 0.18s;
}
.btn-danger:hover { background: rgba(255,59,48,0.06); }
```

---

## Form Fields

```css
.field-label {
  display: block; font-size: 11px; font-weight: 500;
  letter-spacing: 0.05em; text-transform: uppercase;
  color: var(--text-tertiary); margin-bottom: 8px;
}
.field-input {
  width: 100%; font-family: var(--font); font-size: 16px;
  color: var(--text-primary); background: var(--bg);
  border: 0.5px solid var(--border-strong);
  border-radius: var(--radius-sm); padding: 14px 16px;
  outline: none; transition: all 0.2s ease;
  letter-spacing: -0.01em; -webkit-appearance: none;
}
.field-input::placeholder { color: var(--text-tertiary); }
.field-input:focus {
  border-color: var(--accent); background: white;
  box-shadow: 0 0 0 4px rgba(0,113,227,0.10);
}
```

---

## Status Badges / Pills

```css
.badge {
  font-size: 11px; font-weight: 500;
  padding: 4px 10px; border-radius: 99px;
  letter-spacing: 0.01em;
}
.badge-pending    { background: var(--bg); color: var(--text-tertiary); border: 0.5px solid var(--border-strong); }
.badge-inprogress { background: var(--warning-light); color: #9a6200; border: 0.5px solid #f5d08a; }
.badge-done       { background: var(--success-light); color: #1a7a33; }
.badge-skipped    { background: #f2f2f7; color: var(--text-tertiary); border: 0.5px solid var(--border-strong); }
.badge-error      { background: #fff0ef; color: var(--danger); border: 0.5px solid rgba(255,59,48,0.25); }
```

---

## Filter / Tab Bar

Segmented control style.

```css
.tab-bar {
  display: flex; gap: 6px; background: var(--surface);
  border: 0.5px solid var(--border); border-radius: 14px;
  padding: 5px; box-shadow: var(--shadow-sm);
  overflow-x: auto; -webkit-overflow-scrolling: touch;
}
.tab-btn {
  flex: 1; min-width: fit-content; white-space: nowrap;
  font-family: var(--font); font-size: 14px; font-weight: 500;
  color: var(--text-secondary); background: none; border: none;
  border-radius: 10px; padding: 9px 18px; cursor: pointer;
  transition: all 0.2s ease; letter-spacing: -0.01em;
  display: flex; align-items: center; justify-content: center; gap: 8px;
}
.tab-btn:hover { color: var(--text-primary); background: var(--bg); }
.tab-btn.active {
  background: white; color: var(--text-primary);
  font-weight: 600; box-shadow: 0 1px 6px rgba(0,0,0,0.10);
}
.tab-count {
  font-size: 11px; font-weight: 500; padding: 2px 7px;
  border-radius: 99px; background: rgba(0,0,0,0.06);
  color: var(--text-tertiary); transition: all 0.2s;
}
.tab-btn.active .tab-count { background: var(--accent-light); color: var(--accent); }
.tab-count.complete { background: var(--success-light) !important; color: #1a7a33 !important; }
```

---

## Modals

```css
.modal-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.4);
  z-index: 500; opacity: 0; pointer-events: none;
  transition: opacity 0.3s ease;
  backdrop-filter: blur(6px); -webkit-backdrop-filter: blur(6px);
}
.modal-overlay.open { opacity: 1; pointer-events: all; }

.modal {
  position: fixed; top: 50%; left: 50%; z-index: 501;
  transform: translate(-50%, -44%) scale(0.95);
  width: min(780px, 94vw); max-height: 86vh;
  background: var(--surface); border-radius: 24px;
  border: 0.5px solid var(--border);
  box-shadow: 0 32px 80px rgba(0,0,0,0.22);
  display: flex; flex-direction: column;
  opacity: 0; pointer-events: none;
  transition: all 0.35s cubic-bezier(0.34,1.2,0.64,1);
}
.modal.open {
  opacity: 1; pointer-events: all;
  transform: translate(-50%, -50%) scale(1);
}
.modal-header {
  display: flex; align-items: flex-start; justify-content: space-between;
  padding: 24px 28px 18px; border-bottom: 0.5px solid var(--border);
  flex-shrink: 0;
}
.modal-title { font-size: 20px; font-weight: 600; letter-spacing: -0.03em; }
.modal-body  { flex: 1; overflow-y: auto; padding: 24px 28px 28px; }
```

---

## Side Drawer

```css
.drawer {
  position: fixed; top: 0; right: 0; bottom: 0;
  width: min(480px, 95vw); z-index: 201;
  background: var(--surface);
  border-left: 0.5px solid var(--border);
  box-shadow: -8px 0 48px rgba(0,0,0,0.14);
  display: flex; flex-direction: column;
  transform: translateX(100%);
  transition: transform 0.35s cubic-bezier(0.4,0,0.2,1);
}
.drawer.open { transform: translateX(0); }
.drawer-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 18px 20px 14px; border-bottom: 0.5px solid var(--border);
  flex-shrink: 0;
}
.drawer-body { flex: 1; overflow-y: auto; padding: 14px 20px 20px; }
```

---

## Toast Notifications

```css
.toast {
  position: fixed; bottom: 28px; right: 28px; z-index: 999;
  display: flex; align-items: center; gap: 10px;
  background: var(--surface); border: 0.5px solid var(--border-strong);
  border-radius: 14px; padding: 11px 18px 11px 14px;
  box-shadow: var(--shadow-md);
  opacity: 0; transform: translateY(10px);
  transition: opacity 0.3s ease, transform 0.3s ease;
  pointer-events: none;
}
.toast.show { opacity: 1; transform: translateY(0); }
.toast.success { border-color: rgba(52,199,89,0.3); }
.toast.error   { border-color: rgba(255,59,48,0.25); }
```

---

## Entry / Onboarding Card

Centered card for login, onboarding, or project creation screens.

```css
.entry-card {
  background: var(--surface); border-radius: 24px;
  border: 0.5px solid var(--border); box-shadow: var(--shadow-lg);
  padding: 52px 48px; width: 100%; max-width: 480px;
  animation: entryFadeUp 0.6s cubic-bezier(0.4,0,0.2,1) both;
}
@keyframes entryFadeUp {
  from { opacity: 0; transform: translateY(28px); }
  to   { opacity: 1; transform: translateY(0); }
}
.entry-eyebrow {
  font-size: 12px; font-weight: 500; letter-spacing: 0.06em;
  text-transform: uppercase; color: var(--accent); margin-bottom: 10px;
}
.entry-title {
  font-size: 30px; font-weight: 600; letter-spacing: -0.03em;
  line-height: 1.1; color: var(--text-primary); margin-bottom: 8px;
}
.entry-sub {
  font-size: 15px; font-weight: 300; color: var(--text-secondary);
  letter-spacing: -0.01em; line-height: 1.5; margin-bottom: 36px;
}
```

---

## Typography Scale

| Use | Size | Weight | Letter Spacing |
|---|---|---|---|
| Hero title | `clamp(30px,5vw,48px)` | 600 | `-0.03em` |
| Section title | `20px` | 600 | `-0.02em` |
| Card title | `15-17px` | 500-600 | `-0.015em` |
| Body | `15px` | 300-400 | `-0.01em` |
| Secondary body | `13-14px` | 400 | `-0.005em` |
| Label (uppercase) | `11-12px` | 500-600 | `0.05-0.07em` |
| Badge / tag | `11px` | 500 | `0.01em` |

---

## Icons

SF Symbols style — match these specs exactly:

- **Stroke width:** 1.4–1.8px
- **Stroke linecap:** `round`
- **Stroke linejoin:** `round`
- **Fill:** `none` (outline only, except for small filled indicators)
- **Size:** 13–22px viewBox
- **Color:** inherit from parent (`currentColor`) or explicit brand color

**Back/forward chevrons:**
```html
<!-- chevron.left -->
<svg width="18" height="18" viewBox="0 0 18 18" fill="none">
  <path d="M11.5 4L6.5 9L11.5 14" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
</svg>

<!-- chevron.right -->
<svg width="16" height="16" viewBox="0 0 16 16" fill="none">
  <path d="M6 3.5L10.5 8L6 12.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</svg>

<!-- chevron.down -->
<svg viewBox="0 0 18 18" fill="none">
  <path d="M4.5 6.75L9 11.25L13.5 6.75" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
```

**Checkmark:**
```html
<svg width="13" height="10" viewBox="0 0 13 10" fill="none">
  <path d="M1.5 5L5 8.5L11.5 1.5" stroke="white" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
```

---

## Animations

```css
/* Page / card entrance */
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(16px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Staggered list items */
.list-item { animation: fadeUp 0.4s ease both; }
.list-item:nth-child(1) { animation-delay: 0.05s; }
.list-item:nth-child(2) { animation-delay: 0.10s; }
/* ...continue pattern at 0.05s intervals */

/* Spring modal entrance */
.modal.open {
  transform: translate(-50%, -50%) scale(1);
  transition: all 0.35s cubic-bezier(0.34,1.2,0.64,1);
}

/* Checkbox spring */
.checkbox { transition: all 0.22s cubic-bezier(0.34,1.56,0.64,1); }

/* Smooth expand panel (comment/detail sections) */
.expand-panel {
  max-height: 0; overflow: hidden;
  transition: max-height 0.35s cubic-bezier(0.4,0,0.2,1),
              padding 0.35s ease;
}
.expand-panel.open { max-height: 320px; }

/* Loading spinner */
.spin-ring {
  width: 36px; height: 36px; border-radius: 50%;
  border: 3px solid var(--border-strong);
  border-top-color: var(--accent);
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
```

---

## Section Headers (within lists)

```css
.section-header {
  font-size: 11px; font-weight: 600; letter-spacing: 0.07em;
  text-transform: uppercase; color: var(--text-tertiary);
  padding: 20px 4px 10px;
}
```

---

## Page System (multi-page SPA)

```css
.page { display: none; min-height: 100vh; }
.page.active { display: flex; flex-direction: column; }
```

```javascript
function showPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  window.scrollTo(0, 0);
}
```

---

## Progress Ring

```html
<svg class="ring-svg" width="96" height="96" viewBox="0 0 96 96" style="transform:rotate(-90deg)">
  <circle fill="none" stroke="var(--border-strong)" stroke-width="4" cx="48" cy="48" r="42"/>
  <circle id="ringFill" fill="none" stroke="var(--accent)" stroke-width="4"
    stroke-linecap="round" stroke-dasharray="263" stroke-dashoffset="263"
    cx="48" cy="48" r="42"
    style="transition: stroke-dashoffset 0.6s cubic-bezier(0.4,0,0.2,1)"/>
</svg>
```

```javascript
// Update: offset = 263 - (pct/100) * 263
document.getElementById('ringFill').style.strokeDashoffset = 263 - (pct/100) * 263;
```

---

## Empty States

```css
.empty-state { text-align: center; padding: 60px 24px; }
.empty-icon  { display: flex; justify-content: center; margin-bottom: 20px; }
/* Use an SVG illustration in a rounded rect bg: fill #f2f2f7, rx 16 */
.empty-title { font-size: 20px; font-weight: 600; letter-spacing: -0.02em; margin-bottom: 8px; }
.empty-desc  { font-size: 14px; color: var(--text-secondary); line-height: 1.55; max-width: 320px; margin: 0 auto 28px; }
```

---

## Max Width & Spacing

- **Page max-width:** `860px` centered with `margin: 0 auto`
- **Page padding:** `0 24px`
- **Nav height:** `52px` → `padding-top: 52px` on page content
- **Section gap:** `48-56px` vertical between major sections
- **Card gap:** `10-12px` in lists
- **Component internal padding:** `20px 24px` (cards), `14px 16px` (inputs)

---

## localStorage Pattern

```javascript
function loadData()   { try { return JSON.parse(localStorage.getItem('app_key') || '[]'); } catch(_) { return []; } }
function saveData(d)  { try { localStorage.setItem('app_key', JSON.stringify(d)); } catch(_) {} }
```

---

## Example Prompt to Use This File

> *"I've uploaded the Centific design system. Using those exact CSS variables, typography, motion specs, and component patterns, build me a [daily standup tracker / client intake form / project timeline viewer / KPI dashboard]. It should have a home page with action tiles, a form page, and a results/view page. Use the same nav bar, card styles, badges, and animations."*
