# Centific Kilo Design System

Paste this document at the start of a new Claude chat to apply the full visual design used in the Centific Project Kilo Data Collection app. Reference it as **"Centific Kilo Design"**.

---

## Identity

- **Product name:** Centific Data Collection — Project Kilo
- **Design inspiration:** Apple Human Interface Guidelines (iOS/macOS)
- **Primary brand colour:** `#EF43B3` (Centific Pink)
- **Default theme:** Dark mode
- **Target devices:** iPhone 15, desktop browsers

---

## Typography

```
Font stack: -apple-system, 'SF Pro Display', 'SF Pro Text', BlinkMacSystemFont, 'Helvetica Neue', sans-serif
Numbers / KPI displays: 'Avenir Next', 'Avenir', 'SF Pro Display', -apple-system, sans-serif

Font weights used:
  300 — Light (KPI numbers, subtitles)
  400 — Regular (body, labels)
  500 — Medium (nav items, chips, buttons)
  600 — Semibold (headings, panel titles)
  700 — Bold (step numbers, emphasis)

Letter spacing conventions:
  Headings:       -0.03em to -0.04em (tight)
  Body:           -0.01em (slightly tight)
  Labels/eyebrow: +0.05em to +0.08em (wide, uppercase)

Text sizes:
  Page title:   clamp(20px, 2.5vw, 26px)
  Welcome h2:   clamp(22px, 3vw, 32px)
  Panel title:  28px / weight 600
  Body:         14–15px
  Label:        11px / uppercase / weight 600
  Caption:      11–12px
  Tiny pill:    9–10px / uppercase / weight 600
```

---

## Colour Tokens

### CSS Variables — Dark Mode (`data-theme="dark"`)

```css
--bg:           #0f0f0e    /* page background */
--bg2:          #1a1a18    /* card, nav, sidebar */
--bg3:          #232320    /* hover states, inputs */
--bg4:          #2d2d29    /* borders fill, tracks */
--text:         #F7F5F0    /* primary text */
--text2:        #a8a8a0    /* secondary text */
--text3:        #7a7a72    /* muted / labels */
--border:       rgba(255,255,255,0.08)
--border-strong: rgba(255,255,255,0.14)
--input-bg:     #232320
--card-bg:      #1a1a18
--nav-bg:       rgba(15,15,14,0.88)
--sidebar-bg:   #1a1a18
--shadow-sm:    0 2px 12px rgba(0,0,0,0.4)
--shadow-md:    0 8px 40px rgba(0,0,0,0.5)
--shadow-lg:    0 20px 60px rgba(0,0,0,0.6)
```

### CSS Variables — Light Mode (`data-theme="light"`)

```css
--bg:           #f5f5f7
--bg2:          #ffffff
--bg3:          #fbfbfd
--bg4:          #e8e8ed
--text:         #1d1d1f
--text2:        #4a4a4f
--text3:        #8a8a8e
--border:       rgba(180,100,160,0.14)
--border-strong: rgba(150,80,130,0.22)
--input-bg:     #f5f5f7
--card-bg:      #ffffff
--nav-bg:       rgba(245,245,247,0.88)
--shadow-sm:    0 2px 12px rgba(0,0,0,0.06)
--shadow-md:    0 8px 40px rgba(0,0,0,0.10)
--shadow-lg:    0 20px 60px rgba(0,0,0,0.12)
```

### Accent Colours (shared both modes)

```css
--pink:         #EF43B3    /* primary brand — CTAs, active states */
--pink-dim:     #c4359a    /* hover state for pink */
--amber:        #F59E0B    /* warnings, in-progress */
--amber-bg:     #FFF8EC
--amber-text:   #92400E
--green:        #22c55e    /* success, complete */
--green-bg:     #EAF3DE
--green-text:   #27500A
--red:          #ef4444    /* errors, destructive */
--red-bg:       #FEF2F2
--red-text:     #991b1b
--blue:         #3b82f6    /* participant steps, info, links */
--blue-bg:      #EFF6FF
--blue-text:    #1e3a8a
```

### Participant Step Highlight (Blue — special case)

```css
/* Light mode */
.step-dot.participant:not(.done) {
  background: rgba(59,130,246,0.25);
  border-color: #3b82f6;
  border-width: 2px;
}
.step-dot.participant:not(.done) span { color: #2563eb; font-weight: 700; }

/* Dark mode */
[data-theme="dark"] .step-dot.participant:not(.done) {
  background: rgba(147,197,253,0.18);
  border-color: #60a5fa;
}
[data-theme="dark"] .step-dot.participant:not(.done) span { color: #93c5fd; }
```

---

## Border Radius

```css
--r:   12px    /* buttons, inputs, small cards */
--rl:  18px    /* large cards, modals, panels */
```

---

## Spacing System

| Use | Value |
|-----|-------|
| Nav height | 52px |
| Sidebar width | 280px |
| Content padding (desktop) | 28px 32px |
| Content padding (mobile) | 16px |
| Card padding | 22px 24px |
| Entry bar padding | 10px 24px |
| Gap between items | 14–16px |
| Small gap | 6–8px |

---

## Key Components

### Nav Bar
- `position: sticky; top: 0; z-index: 200`
- Frosted glass: `backdrop-filter: saturate(180%) blur(20px)`
- Height: 52px
- Right side: progress pill → user chip → Export → Reset → ☰

### Cards / Capture Cards
```css
background: var(--card-bg);
border: .5px solid var(--border);
border-radius: var(--rl);       /* 18px */
padding: 22px 24px;
box-shadow: var(--shadow-sm);
transition: box-shadow .22s, border-color .22s;
/* hover → shadow-md + border-strong */
```

### Buttons (Primary — Pink)
```css
background: var(--pink);
color: #fff;
border: none;
border-radius: var(--r);        /* 12px */
padding: 14px;
font-weight: 500;
font-size: 15px;
transition: all .25s cubic-bezier(.4,0,.2,1);
/* hover → translateY(-1px) + box-shadow */
/* active → scale(0.98) */
/* disabled → opacity 0.6 */
```

### Buttons (Secondary)
```css
background: var(--bg2);
border: .5px solid var(--border-strong);
color: var(--text2);
/* hover → var(--bg4) */
```

### Input Fields
```css
background: var(--input-bg);
border: .5px solid var(--border-strong);
border-radius: var(--r);
padding: 7px 10px;
font-size: 14px;
/* focus → border-color: var(--pink), box-shadow: 0 0 0 4px rgba(239,67,179,.10) */
/* error → border-color: var(--red) */
```

### Status Pills
```css
/* Pill base */
font-size: 9–11px; font-weight: 600;
letter-spacing: .04–.06em; text-transform: uppercase;
border-radius: 20px; padding: 2–3px 7–10px;

/* States */
Not Started: background var(--bg4), color var(--text3)
In Progress: background #FFF3DC, color #9a6200, border .5px solid #f5d08a
Complete:    background var(--green-bg), color var(--green-text)
Skipped:     background var(--bg3), color var(--text3)
```

### Progress Bar (Sidebar)
```css
height: 3px; background: var(--bg4); border-radius: 2px;
.fill { background: var(--amber); transition: width .3s cubic-bezier(.4,0,.2,1) }
```

### Step Dots
```css
width/height: 26px; border-radius: 50%;
border: 1.5px solid var(--border-strong);
background: var(--step-bg);
/* done → background: var(--pink); border-color: var(--pink) */
/* participant → blue highlight (see Colour Tokens above) */
```

### Toast Notifications
```css
position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
background: var(--text); color: var(--bg);
border-radius: var(--rl); padding: 12px 20px;
font-size: 14px; font-weight: 500;
box-shadow: var(--shadow-lg);
animation: fadeUp .3s cubic-bezier(.4,0,.2,1);
```

### Segmented Control (Tab Bar)
```css
display: flex; gap: 3px;
background: var(--bg3); border: .5px solid var(--border); border-radius: var(--r); padding: 3px;
/* active tab → background: var(--bg2), font-weight: 600, box-shadow: 0 1px 6px rgba(0,0,0,.10) */
```

### KPI Cards (Dashboard)
```css
background: var(--card-bg);
border: .5px solid var(--border);
border-radius: var(--rl);
padding: 20px 22px;
min-height: 110px;
/* Layout: label top-left, number bottom-right */

/* Number sizes by viewport */
> 960px:   110px / weight 300 / Avenir Next
960–700px:  80px / weight 300
700–480px:  64px / weight 300
< 480px:    48px / weight 300

/* Accent colours */
.accent → var(--pink)
.green  → var(--green)
.blue   → var(--blue)
.amber  → var(--amber)
```

---

## Animations

```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(14px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes blink {
  0%, 100% { opacity: 1; } 50% { opacity: .3; }
}
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 60%  { transform: translateX(-6px); }
  40%, 80%  { transform: translateX(6px); }
}
@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Standard transition */
transition: all .22s cubic-bezier(.4, 0, .2, 1);

/* Spring (for cards, buttons) */
transition: all .25s cubic-bezier(.34, 1.2, .64, 1);
```

---

## Mobile Breakpoints

```css
/* Main breakpoint — stacks to mobile layout */
@media (max-width: 700px) {
  /* Nav: hide brand text, show logo only */
  /* Entry bar: 2×2 CSS Grid */
  /* Sidebar: full-width overlay */
  /* Hide Export/Reset text, show icon only */
}

/* Small mobile */
@media (max-width: 480px) {
  /* KPI numbers: 48px */
  /* Charts: single column */
}
```

---

## Admin Password

```
DK2026
```

---

## App Admin Login ID

```
Admin-Kilo
```

---

## Chart Styling (Chart.js)

```js
// Font
Chart.defaults.font.family = "-apple-system, 'SF Pro Display', sans-serif";
Chart.defaults.font.size = 12;

// Tooltip
tooltip: {
  backgroundColor: dark ? 'rgba(26,26,24,0.96)' : 'rgba(255,255,255,0.96)',
  borderColor: dark ? 'rgba(255,255,255,0.12)' : 'rgba(0,0,0,0.10)',
  borderWidth: 1,
  cornerRadius: 10,
  padding: 12,
}

// Grid
grid: { color: dark ? 'rgba(255,255,255,0.05)' : 'rgba(0,0,0,0.05)' }

// Axes — integers only, no decimals
ticks: { precision: 0, stepSize: 1, callback: v => Number.isInteger(v) ? v : '' }

// Bar border radius: 6–8px
// Line tension: 0.4
// Fill: rgba(239,67,179,.10)
```

---

## Design Principles

1. **Hairline borders** — always `.5px solid` never `1px`
2. **Frosted glass nav** — `backdrop-filter: blur(20px)` on sticky navs
3. **Tight tracking on headings** — `-0.03em` letter spacing
4. **Light weight numbers** — KPI figures use `font-weight: 300`
5. **Pink as the single action colour** — all primary CTAs, active indicators, brand elements
6. **Layered depth** — `bg → bg2 → bg3 → bg4` for z-depth without heavy shadows
7. **Animate entrances** — `fadeUp` on panels, cards, modals
8. **Mobile-first entry bar** — CSS Grid `1fr 1fr` on mobile, flex row on desktop
9. **Sticky toolbars** — dashboard toolbar stays fixed while content scrolls
10. **Participant steps** — blue highlight reserved exclusively for participant-action steps

