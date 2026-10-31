# Design System & UI/UX Guidelines
## Autometa School Enterprise Resource Planning (ERP) Platform
**Document Version:** 1.0 (Baseline V3 Alignment)  
**Document Status:** Approved Design Specification  
**Design Paradigm:** Utility-First with Tailwind CSS | Responsive Multi-Portal System | WCAG 2.1 Level AA Compliant  

---

## 1. Design Vision & Guiding Principles

The **Autometa Design System** delivers a modern, clean, high-performance institutional UI engineered for four distinct user archetypes across desktop workstations, tablets, and low-end mobile devices.

### 1.1 Core Principles
1. **Utility & Clarity First**: Minimize visual clutter. Academic coordinators, teachers, and accountants require high data density without cognitive fatigue.
2. **Speed & Offline Feedback**: Instant UI feedback (< 50ms) for field operations. The interface clearly communicates network states (*Online*, *Offline Mode - Local Writes Active*, *Synchronizing...*).
3. **Accessibility (WCAG 2.1 Level AA)**: Strict contrast ratios (minimum 4.5:1 for body text, 3:1 for large display text and UI components), visible focus indicators, and touch targets $\ge 44 \times 44\text{ px}$.
4. **Adaptive Portal Personalities**:
   * **Admin & Accounts**: High-density tables, multi-step wizards, compact KPI cards.
   * **Teacher PWA**: Large touch-friendly roll-call toggles, high-contrast states, swipeable lists.
   * **Parent & Student**: Consumer-grade card summaries, intuitive progress bars, simplified ledger overviews.

---

## 2. Color Palette & Theming

The color architecture is built around an authoritative **Institutional Navy** primary, an engaging **Cobalt Blue** interactive accent, neutral enterprise surfaces, and strict semantic indicators.

### 2.1 Primary Brand & Institutional Palette

| Token Name | Hex Code | Tailwind Class | Contrast (on White) | Usage / Context |
| :--- | :--- | :--- | :--- | :--- |
| `brand-navy-900` | `#0B192C` | `bg-slate-950` / `#0B192C` | **15.4:1** (AAA) | Primary brand header, sidebar background in dark theme, high-emphasis text. |
| `brand-navy-800` | `#1E3E62` | `#1E3E62` | **8.6:1** (AAA) | Primary navigation bars, main action buttons, active tab indicators. |
| `brand-blue-600` | `#0066FF` | `bg-blue-600` / `#0066FF` | **4.6:1** (AA) | Primary interactive actions, links, focus rings, active toggle states. |
| `brand-blue-500` | `#2563EB` | `bg-blue-500` | **4.5:1** (AA) | Hover state for buttons, active portal badges. |
| `brand-blue-50` | `#EFF6FF` | `bg-blue-50` | N/A | Active table row highlight, selected menu item surface. |

### 2.2 Semantic & Status Indicators
Every semantic color is paired with a distinct icon and text label to ensure information is never conveyed by color alone.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        SEMANTIC COLOR TOKENS                           │
│                                                                        │
│  SUCCESS (Present / Paid / Approved)                                   │
│  • Surface: #ECFDF5 (emerald-50)   • Text/Icon: #065F46 (emerald-800)  │
│  • Badge:   #059669 (emerald-600)  • Border:    #A7F3D0 (emerald-200)  │
│                                                                        │
│  WARNING (Late / Pending Approval / Due Soon)                          │
│  • Surface: #FFFBEB (amber-50)     • Text/Icon: #92400E (amber-800)    │
│  • Badge:   #D97706 (amber-600)    • Border:    #FDE68A (amber-200)    │
│                                                                        │
│  DANGER / ERROR (Absent / Overdue / System Collision)                  │
│  • Surface: #FEF2F2 (red-50)       • Text/Icon: #991B1B (red-800)      │
│  • Badge:   #DC2626 (red-600)      • Border:    #FECACA (red-200)      │
│                                                                        │
│  INFO / SYSTEM (Notices / Offline Sync / Excused)                      │
│  • Surface: #F0F9FF (sky-50)       • Text/Icon: #075985 (sky-800)      │
│  • Badge:   #0284C7 (sky-600)      • Border:    #BAE6FD (sky-200)      │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Attendance-Specific Color Tokens (FR-07 Roll-Call)
* **PRESENT**: Background `#ECFDF5`, Border `#10B981`, Text `#065F46` (Green state).
* **ABSENT**: Background `#FEF2F2`, Border `#EF4444`, Text `#991B1B` (Red state).
* **LATE**: Background `#FFFBEB`, Border `#F59E0B`, Text `#92400E` (Amber state).
* **EXCUSED**: Background `#F3F4F6`, Border `#9CA3AF`, Text `#374151` (Gray state).

### 2.4 Neutral Surfaces & Typography Grays

| Token Name | Hex Code | Tailwind Equivalent | Purpose |
| :--- | :--- | :--- | :--- |
| `surface-ground` | `#F8FAFC` | `bg-slate-50` | Universal application background for portal views. |
| `surface-card` | `#FFFFFF` | `bg-white` | Elevated cards, data table wrappers, modals. |
| `surface-inset` | `#F1F5F9` | `bg-slate-100` | Input backgrounds, disabled elements, table header rows. |
| `border-subtle` | `#E2E8F0` | `border-slate-200` | Card borders, table cell dividers. |
| `border-strong` | `#CBD5E1` | `border-slate-300` | Input borders, button outlines. |
| `text-primary` | `#0F172A` | `text-slate-900` | Primary headings, table data values, high-contrast labels. |
| `text-secondary`| `#475569` | `text-slate-600` | Supporting text, input labels, table column headers. |
| `text-muted` | `#94A3B8` | `text-slate-400` | Disabled text, placeholder text, breadcrumbs. |

---

## 3. Typography & Hierarchy

Autometa ERP utilizes **Inter** as the primary typographic font family due to its exceptional x-height, screen legibility at small sizes (11px–13px in financial ledgers), and extensive tabular figure support (`font-variant-numeric: tabular-nums`).

For official student certificates (Transfer Certificates, Bonafide) and formal report card headers, a dignified serif like **Cinzel** or **Merriweather** is utilized.

```
Primary Sans-Serif:     "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif
Monospace / Tabular:    "JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, monospace
Formal Print Headings:  "Merriweather", Georgia, serif
```

### 3.1 Type Scale & Hierarchy

| Level | Size (rem / px) | Weight | Line Height | Tailwind Class | Usage Context |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Display 1** | `2.25rem / 36px` | Bold (700) | `1.2 (44px)` | `text-3xl font-bold` | Main portal welcome banner, landing page hero titles. |
| **Heading 1** | `1.875rem / 30px`| Bold (700) | `1.25 (38px)`| `text-2xl font-bold` | Top-level section titles (e.g., "Fee Ledger", "Attendance Engine"). |
| **Heading 2** | `1.5rem / 24px` | SemiBold (600)| `1.3 (32px)` | `text-xl font-semibold`| Modal titles, panel headers, KPI card category labels. |
| **Heading 3** | `1.25rem / 20px` | SemiBold (600)| `1.4 (28px)` | `text-lg font-semibold`| Sub-section headers, widget titles, student profile headers. |
| **Body Large** | `1.0rem / 16px` | Regular (400) | `1.5 (24px)` | `text-base` | Primary content prose, form instructions, parent notices. |
| **Body Default**| `0.875rem / 14px`| Regular (400) | `1.45 (20px)`| `text-sm` | Default application font: table rows, form inputs, button text. |
| **Body Medium** | `0.875rem / 14px`| Medium (500) | `1.45 (20px)`| `text-sm font-medium` | Navigation menu items, active tabs, table headers. |
| **Caption** | `0.75rem / 12px` | Regular (400) | `1.35 (16px)`| `text-xs` | Timestamps, metadata, fine calculation notes, helper text. |
| **Micro / Tag** | `0.6875rem / 11px`| Bold (700) | `1.2 (14px)` | `text-[11px] font-bold uppercase tracking-wider` | Status badges, board identifiers (CBSE/ICSE), table badges. |
| **Numerical Data**| `0.875rem / 14px`| Medium (500) | Tabular Nums | `text-sm font-mono tabular-nums` | Financial ledger amounts, roll numbers, percentage scores. |

---

## 4. Layout Architecture & Responsive Breakpoints

The client layouts conform dynamically across the three key hardware baselines specified in SRS Section 4.1.1:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        RESPONSIVE BREAKPOINTS                          │
│                                                                        │
│  [Mobile Viewports]       360px  - 639px  (sm)  Teachers & Parents PWA │
│  [Tablet Devices]         640px  - 1023px (md)  Classroom Tablets      │
│  [Standard Desktops]      1024px - 1439px (lg)  Admin Workstations     │
│  [High-Res Displays]      1440px - 1920px (xl+) Federated Management   │
└────────────────────────────────────────────────────────────────────────┘
```

### 4.1 Master Portal Shell Architecture
* **Administrator Portal (Desktop-Centric)**:
  * Persistent 260px collapsible sidebar navigation on left.
  * Sticky 64px top navigation bar featuring Tenant Selector, Search Bar (`Cmd + K`), and User Profile.
  * Fluid content container (`max-w-[1600px]`) supporting multi-column data grids.
* **Teacher Portal (PWA Mobile-First)**:
  * Persistent bottom navigation bar (56px) on mobile for thumb-reach access: *Roster*, *Roll-Call*, *Timetable*, *Substitutions*.
  * Floating action button (FAB) for rapid attendance commit.
  * Sticky offline connectivity status banner.
* **Parent & Student Portals (Card-Based)**:
  * Top navigation with prominent **Child Switcher Dropdown** on parent accounts.
  * Single-column stream on mobile, shifting to 2-column or 3-column masonry grid on desktop screens.

---

## 5. UI Component Tokens & Interaction Patterns

### 5.1 Connectivity & PWA Status Banners (SRS Section 4.1.1)
Must be placed persistently at the top of the viewport when offline or synchronizing:

```html
<!-- Offline Banner: Amber State -->
<div class="bg-amber-500 text-white px-4 py-2 text-xs font-semibold flex items-center justify-between shadow-sm">
  <div class="flex items-center gap-2">
    <span class="w-2 h-2 rounded-full bg-white animate-pulse"></span>
    <span>Offline Mode — Local Writes Active (5 pending mutations)</span>
  </div>
  <span class="text-[11px] opacity-90">Auto-syncs upon connection</span>
</div>

<!-- Syncing Banner: Blue State -->
<div class="bg-blue-600 text-white px-4 py-2 text-xs font-semibold flex items-center gap-2 shadow-sm">
  <svg class="animate-spin h-3.5 w-3.5 text-white" fill="none" viewBox="0 0 24 24">
    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"></path>
  </svg>
  <span>Synchronizing attendance mutations with server...</span>
</div>
```

### 5.2 Buttons & Interactive Elements

```
┌────────────────────────────────────────────────────────────────────────┐
│  PRIMARY ACTION:                                                       │
│  class="inline-flex items-center justify-center px-4 py-2.5            │
│         text-sm font-medium text-white bg-blue-600 rounded-lg          │
│         hover:bg-blue-700 focus:outline-none focus:ring-2              │
│         focus:ring-blue-500 focus:ring-offset-2 transition-all         │
│         active:scale-[0.98] disabled:opacity-50 disabled:cursor-not-allowed"
│                                                                        │
│  SECONDARY ACTION:                                                     │
│  class="inline-flex items-center justify-center px-4 py-2.5            │
│         text-sm font-medium text-slate-700 bg-white border             │
│         border-slate-300 rounded-lg hover:bg-slate-50 focus:ring-2     │
│         focus:ring-slate-400 focus:ring-offset-2 transition-all"       │
│                                                                        │
│  DANGER ACTION (Cancel / Void):                                        │
│  class="inline-flex items-center justify-center px-4 py-2.5            │
│         text-sm font-medium text-red-700 bg-red-50 border              │
│         border-red-200 rounded-lg hover:bg-red-100 focus:ring-2        │
│         focus:ring-red-500 transition-all"                             │
└────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Teacher Roll-Call Toggle States (PWA Touch Target $\ge 44 \times 44\text{ px}$)

```html
<!-- Roll-Call Status Segmented Control -->
<div class="inline-flex rounded-lg border border-slate-200 bg-slate-100 p-1">
  <!-- Present (Active) -->
  <button type="button" class="min-h-[44px] min-w-[54px] px-3 py-1.5 text-xs font-bold rounded-md bg-emerald-600 text-white shadow-sm transition-all">
    P
  </button>
  <!-- Late -->
  <button type="button" class="min-h-[44px] min-w-[54px] px-3 py-1.5 text-xs font-medium text-slate-700 hover:text-amber-700 hover:bg-white rounded-md transition-all">
    L
  </button>
  <!-- Absent -->
  <button type="button" class="min-h-[44px] min-w-[54px] px-3 py-1.5 text-xs font-medium text-slate-700 hover:text-red-700 hover:bg-white rounded-md transition-all">
    A
  </button>
  <!-- Excused -->
  <button type="button" class="min-h-[44px] min-w-[54px] px-3 py-1.5 text-xs font-medium text-slate-700 hover:text-slate-900 hover:bg-white rounded-md transition-all">
    E
  </button>
</div>
```

### 5.4 Data Tables (High-Density Administration & Ledger)
* **Header Style**: `bg-slate-50 text-slate-600 text-[11px] font-bold uppercase tracking-wider py-3 px-4 border-b border-slate-200`.
* **Row Style**: `hover:bg-blue-50/50 transition-colors border-b border-slate-100 py-3.5 px-4 text-sm text-slate-800`.
* **Numeric Values**: Tabular numbers right-aligned (`text-right font-mono tabular-nums`).
* **Zebra Striping**: Optional on datasets $> 50$ rows (`even:bg-slate-50/40`).

### 5.5 Form Controls & Client Validation (SRS Section 4.1.2)
* **Standard Input Field**:
  ```html
  <label class="block text-xs font-semibold text-slate-700 mb-1.5">
    Admission Number <span class="text-red-500">*</span>
  </label>
  <input type="text" 
         class="w-full h-10 px-3 text-sm bg-white border border-slate-300 rounded-lg text-slate-900 placeholder:text-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
         placeholder="e.g. ADM-2026-0891" />
  ```
* **Validation Error State**:
  Border shifts to `border-red-500 focus:ring-red-400`, accompanied by an immediate inline error message below the input:
  `<p class="mt-1 text-xs text-red-600 font-medium">Please enter a valid admission code.</p>`

### 5.6 Toast Feedback System (Non-Blocking)
Asynchronous operations (background PDF generation, batch export, offline sync) use toast notifications anchored to the bottom-right corner (`z-50`), never blocking user interaction with modal dialogs:
* Container: `fixed bottom-4 right-4 z-50 flex flex-col gap-2 max-w-sm w-full`.
* Auto-dismiss after 4 seconds for success; persistent with retry button for sync errors.

---

## 6. Print Media Styles (SRS Section 4.2)

Autometa ERP incorporates dedicated CSS `@media print` style blocks to format documents for two specific hardware targets:
1. **80 mm Thermal POS Printers** (Cashier fee receipts)
2. **A4 Sheet Desktop Printers** (Report cards, Transfer Certificates)

```css
/* Print Media Stylesheet */
@media print {
  /* 1. Global Print Clean-up */
  nav, sidebar, .connectivity-banner, .action-buttons, footer, .toast-container {
    display: none !important;
  }
  body {
    background: #FFFFFF !important;
    color: #000000 !important;
    font-size: 12pt;
  }

  /* 2. Standard 80mm Thermal Receipt Layout */
  .thermal-receipt {
    width: 80mm !important;
    max-width: 80mm !important;
    padding: 2mm !important;
    font-family: 'JetBrains Mono', monospace !important;
    font-size: 9pt !important;
    line-height: 1.2 !important;
  }
  .thermal-receipt table {
    width: 100% !important;
    border-collapse: collapse !important;
  }
  .thermal-receipt .border-dashed {
    border-top: 1px dashed #000 !important;
  }

  /* 3. Standard A4 Document / Report Card Layout */
  .page-a4 {
    width: 210mm !important;
    min-height: 297mm !important;
    padding: 15mm 20mm !important;
    margin: 0 auto !important;
    box-shadow: none !important;
    page-break-after: always !important;
  }
  .page-a4 h1, .page-a4 h2 {
    font-family: 'Merriweather', serif !important;
    color: #0B192C !important;
  }
}
```

---

## 7. Tailwind CSS Configuration Token Export

Here is the exact `tailwind.config.js` design system token configuration for Autometa ERP:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./templates/**/*.html",
    "./static/js/**/*.js",
    "./apps/**/templates/**/*.html",
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          navy: {
            900: '#0B192C',
            800: '#1E3E62',
            700: '#2A5584',
          },
          blue: {
            50: '#EFF6FF',
            100: '#DBEAFE',
            500: '#3B82F6',
            600: '#0066FF',
            700: '#1D4ED8',
          },
        },
        surface: {
          ground: '#F8FAFC',
          card: '#FFFFFF',
          inset: '#F1F5F9',
        },
        attendance: {
          present: '#059669',
          absent: '#DC2626',
          late: '#D97706',
          excused: '#4B5563',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['"JetBrains Mono"', 'monospace'],
        formal: ['Merriweather', 'Georgia', 'serif'],
      },
      boxShadow: {
        'subtle': '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
        'card': '0 1px 3px 0 rgba(0, 0, 0, 0.08), 0 1px 2px -1px rgba(0, 0, 0, 0.08)',
        'elevated': '0 4px 6px -1px rgba(0, 0, 0, 0.08), 0 2px 4px -2px rgba(0, 0, 0, 0.08)',
      },
      screens: {
        'xs': '360px',  // Mobile baseline
        'sm': '640px',
        'md': '768px',  // Tablet baseline
        'lg': '1024px', // Workstation baseline
        'xl': '1280px',
        '2xl': '1536px',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

---

## 8. Accessibility & Quality Checklist (WCAG 2.1 AA)

Before any UI template is merged into the staging environment:
* [x] **Color Contrast Ratio**: Text to background contrast verified at $\ge 4.5:1$ using automated Lighthouse/axe audits.
* [x] **Touch Target Sizes**: All interactive elements (especially teacher roll-call toggles and mobile portal buttons) maintain a minimum target area of $44 \times 44\text{ px}$.
* [x] **Focus Ring Visibility**: Every interactive input, button, and link provides an unmistakable `:focus-visible` ring (`ring-2 ring-blue-500 ring-offset-2`).
* [x] **Form Labels & ARIA**: Form inputs have associated `<label>` tags with matching `for` attributes; icon-only buttons include `aria-label`.
* [x] **Reduced Motion**: Respects `prefers-reduced-motion` to disable animations for users with vestibular sensitivities.
