# Grid Component

## Overview
The main data grid is a single `<table class="mg">` rendered by `renderGrid()` in JavaScript. It combines bond info, snapshot, editable moves, computed deltas, T-1 references, reference sources, greeks, and P&L attribution in one unified table.

## Column Groups (left to right)

### Row 1: Group Headers (`.mg th.grp`)
| Group | Colspan | Color | Columns |
|-------|---------|-------|---------|
| (frozen) | 3 | cyan | radio, Ticker, Issuer |
| Bond | 5 | cyan | ISIN, Ccy, Cpn, Mat, Life |
| Snapshot | 5 | grey | FV, Mkt, Flr, Par, Mny |
| Moves (editable) | 13 | amber | Spot T, ΔS, ΔS%, Y T, Δy, CS T, Δcs, Vol T, Δσ, Spot₀, Y₀, CS₀, Vol₀ |
| Reference Source | 4 | grey | Spot Ref, Rate Ref, Credit Ref, Vol Ref |
| Greeks | 6 | purple | δ, Γs, DV01, CR01, ν, θ |
| P&L Attribution | 8 | green | Delta, Gamma, Rates, Credit, Vega, Theta, FX, Total |

### Row 2: Column Sub-Headers
Each `<th>` gets a `data-colidx` attribute for persistent width tracking.

## Frozen Columns
First 3 columns are position-sticky:
```css
.mg .frz { position:sticky; z-index:3; background:var(--bg); }
.mg .frz0 { left:0; }      /* radio button, 22px */
.mg .frz1 { left:22px; }   /* Ticker, 68px */
.mg .frz2 { left:90px; }   /* Issuer, 60px, right border */
```
Row backgrounds (even, hover, selected) must be explicitly set on `.frz` cells to prevent transparency showing through.

## Editable Cells (`.mg td.edit`)
Columns: Spot T, Y T, Δy, CS T, Δcs, Vol T, Δσ

```css
.mg td.edit { background: rgba(245,158,11,0.07); }
.mg td.edit input {
  width:100%; background:transparent;
  border:none; border-bottom:1px solid rgba(245,158,11,0.25);
  color:#e2e8f0; font:11px monospace; text-align:right;
}
```

### Edit → Sync → Compute Flow
```
User edits input
  → onChange fires
  → Parse value
  → Update moves[idx] (sync T-level ↔ delta pairs)
  → POST /api/compute with all selected bonds' moves
  → Update results{}
  → renderAll() (preserves column widths from layoutState)
```

### T-Level ↔ Delta Sync Logic
```javascript
if (field === 'yld_t')  { m.yld_t = val; m.dy = val - yld0; }
if (field === 'dy')     { m.dy = val;    m.yld_t = yld0 + val; }
if (field === 'cs_t')   { m.cs_t = val;  m.ds = val - cs0; }
if (field === 'ds')     { m.ds = val;    m.cs_t = cs0 + val; }
if (field === 'vol_t')  { m.vol_t = val/100; m.dv = (val/100) - vol0; }
if (field === 'dv')     { m.dv = val;    m.vol_t = vol0 + val; }
```

## Computed Columns (read-only, derived)
- **ΔS**: `spot_t1 - SpotPrice`, colored green/red
- **ΔS%**: `ΔS / SpotPrice × 100`
- These are rendered as plain `<td>` (not inputs)

## Column Resize
Each sub-header `<th>` gets a 4px invisible drag handle appended as a child `<div>`:
```javascript
th.appendChild(handle);  // position:absolute, right:0, width:4px
handle.onmousedown → track drag → apply width to th + all td in same column
  → on mouseup: setColWidth(colId, finalW) → saves to localStorage
```

Widths are restored from `layoutState.colWidths` on each `renderGrid()` call via the `W()` helper:
```javascript
const W = (defW, cls, label) => {
  const idx = _colIdx++;
  const w = getColWidth(idx, defW);  // saved or default
  return `<th ... style="width:${w}px;min-width:20px">`;
};
```

## Default Column Widths (px)
| Column | Width | Column | Width |
|--------|-------|--------|-------|
| Radio | 22 | Spot T | 62 |
| Ticker | 68 | ΔS | 52 |
| Issuer | 60 | ΔS% | 46 |
| ISIN | 95 | Y T | 50 |
| Ccy | 32 | Δy | 42 |
| Cpn | 30 | CS T | 50 |
| Mat | 72 | Δcs | 42 |
| Life | 30 | Vol T | 52 |
| FV | 52 | Δσ | 48 |
| Mkt | 52 | Spot₀ | 52 |
| Flr | 50 | Y₀ | 42 |
| Par | 42 | CS₀ | 42 |
| Mny | 40 | Vol₀ | 48 |
| Spot Ref | 100 | DV01 | 62 |
| Rate Ref | 155 | CR01 | 62 |
| Credit Ref | 155 | P&L cols | 58 each |
| Vol Ref | 110 | Total | 72 |

## Selection
- Radio button in first column → sets `detailIdx`, re-renders detail panel
- Selected row gets `.sel` class (cyan tinted background)
- Ticker cell gets `.sel-mark` (left cyan border)
- Star `★` appended to ticker text

## Portfolio Total Row
- Class `.port-total` — top border amber, bold values
- Frozen cells show "PORT" / "TOTAL"
- P&L columns show aggregated sums across all selected bonds
