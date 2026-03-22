# Frontend Layout (`static/index.html`)

## Architecture
Everything is in a single `index.html` — HTML structure, CSS styles, and JavaScript logic. No build step, no npm, no framework.

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│ .app (display:flex, height:100vh)                                   │
├──────────┬──┬───────────────────────────────────────────────────────┤
│ .sidebar │↔│ .center                                                │
│ (175px)  │  │                                                       │
│          │  │ ┌─ .header ─────────────────────────────────────────┐ │
│ POSITIONS│  │ │ CB P&L Attribution [PROD]    info  [Reset Layout] │ │
│          │  │ └───────────────────────────────────────────────────┘ │
│ ▸ HK (5) │  │                                                       │
│   ● 6886 │  │ ┌─ .grid-wrap (overflow:auto, max-height:50vh) ────┐ │
│   ● 9618 │  │ │ <table class="mg"> unified bond grid              │ │
│   ● 0700 │  │ │ (frozen cols: radio, ticker, issuer)              │ │
│          │  │ └───────────────────────────────────────────────────┘ │
│ ▸ US (1) │  │ ↕ resize-v                                           │
│   ● BABA │  │ ┌─ .bottom-split (display:flex) ───────────────────┐ │
│          │  │ │ .bottom-panel │↔│ .bottom-panel │↔│ .bottom-panel │ │
│──────────│  │ │  Waterfall    │  │  Breakdown    │  │  Detail      │ │
│ TOTAL PNL│  │ │  (canvas)    │  │  (table)      │  │  (sections)  │ │
│ EQ PNL   │  │ │              │  │               │  │              │ │
│ NET DELTA│  │ └───────────────────────────────────────────────────┘ │
└──────────┴──┴───────────────────────────────────────────────────────┘
  ↔ = draggable horizontal resize handle (.resize-h)
  ↕ = draggable vertical resize handle (.resize-v)
```

## Color Palette (CSS variables)

```css
--bg: #0f1923          /* main background */
--bg2: rgba(255,255,255,0.02)   /* subtle alternate row */
--bg3: rgba(255,255,255,0.04)   /* hover state */
--border: rgba(255,255,255,0.08) /* borders */
--text: #e2e8f0        /* primary text */
--text2: #94a3b8       /* secondary text */
--text3: #64748b       /* muted labels */
--green: #10b981       /* positive values, profit */
--red: #ef4444         /* negative values, loss */
--amber: #f59e0b       /* warnings, totals, editable highlights */
--cyan: #06b6d4        /* accents, selected state, headers */
--purple: #a78bfa      /* greeks section */
--blue: #3b82f6        /* DV01 */
--edit-bg: rgba(245,158,11,0.07)     /* editable cell background */
--edit-border: rgba(245,158,11,0.25) /* editable cell underline */
```

## Typography

```css
--font-mono: 'Consolas','Monaco','Courier New',monospace  /* ALL numeric values */
--font-sans: 'Inter','Segoe UI','Helvetica Neue',sans-serif  /* labels, headers */
```

| Element | Font | Size | Style |
|---------|------|------|-------|
| Header title | sans | 14px | bold |
| Section headers | sans | 9-10px | bold, uppercase, letter-spacing |
| Column headers | sans | 9px | 600, uppercase |
| Data cells | mono | 11px | normal |
| KV labels | sans | 9px | 500, uppercase, muted |
| KV values | mono | 12px | 600 |
| Badges | sans | 8px | 700, uppercase |

## Sidebar (`.sidebar`)

- Width: 175px (resizable, min 100px)
- Background: `#0c1520`
- Contains:
  - Title bar "POSITIONS" with ALL/NONE buttons
  - Bond groups by region (`st.expander`-like with ▸ HK (5))
  - Each bond: checkbox + colored dot (green/red by P&L) + ticker + issuer + mini P&L
  - Click bond row → set as detail view
  - Quick stats at bottom: Total P&L, Equity P&L, Net Delta

## Bottom 3-Panel Split

- **Panel 1 — Waterfall**: Canvas-drawn bar chart, auto-resizes
- **Panel 2 — Breakdown**: HTML table with per-bond P&L by component
- **Panel 3 — Detail**: Collapsible sections for selected bond
- Each panel has a `.bottom-panel-hdr` (cyan, uppercase, underlined)
- Separated by `.resize-h` drag handles
