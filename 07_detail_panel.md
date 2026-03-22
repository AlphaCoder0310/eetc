# Detail Panel (Bottom-Right)

## Overview
The rightmost panel in the bottom 3-panel split. Shows structured detail for the bond selected via radio button. Rendered by `renderRight()`.

## Structure
All sections use collapsible `<div>` with click-to-toggle:
```html
<div class="section">
  <div class="section-hdr" onclick="toggle">
    <span class="chev open">▸</span> SECTION TITLE
  </div>
  <div class="section-body">...content...</div>
</div>
```

## Header
```
6886.HK [HKD] [LONG]
6886.HK 0.000 02/08/2027
```
- Ticker in 14px bold monospace
- Badges: `.badge-ccy` (cyan), `.badge-long` (green)
- Description in 10px muted

## Sections

### 1. BOND INFO (expanded by default)
3-column key-value grid (`.kv.three`):
- Issuer, ISIN, Currency, Coupon, Maturity, Life, Conv Ratio, Spot FX, Underlying

### 2. SNAPSHOT (expanded)
2-column key-value grid:
- Fair Value (large bold), Market Price (large bold)
- Bond Floor, Parity, Moneyness, Inv Value

### 3. CURRENT MOVES (expanded)
3-column grid:
- ΔSpot (green/red, large), ΔSpot% (green/red, large), Spot T
- Δ Yield (bp), Δ Spread (bp), Δ Vol

### 4. FACTOR REFERENCES (collapsed by default)
2-column grid:
- Spot (ΔS) → "3690.HK Equity"
- Rate (Δy) → "US 1Y Tsy (USGG1YR Index)"
- Credit (Δcs) → "Meituan 5Y CDS / iTraxx Asia IG"
- Vol (Δσ) → "CB model-implied vol"
- Rate Tenor → "1Y (life 0.9y)" in cyan
- FX → "USDHKD Curncy"

### 5. GREEKS (expanded)
3-column grid with per-greek colors:
- Delta (cyan), Gamma (cyan), DV01 (blue)
- CR01 (amber), Vega (purple), Theta (grey)

### 6. P&L ATTRIBUTION (expanded)
Mini horizontal bar chart per factor:
```
DELTA  ████████████████████████████████  +0.037629
GAMMA  ████                              +0.002673
RATES  ░                                 +0.000000
...
─────────────────────────────────────────
TOTAL                                    +0.038992
RESID                                    +0.000000
```

Each row: `.bar-row` with:
- `.bar-label` (44px, uppercase, 9px sans)
- `.bar-track` (flex:1, 5px height, rounded)
  - `.bar-fill` (width proportional to max, green or red)
- `.bar-val` (72px, right-aligned, monospace)

Total/Resid separated by top border, Total in large bold.

### 7. ANALYTICS (collapsed by default)
2-column grid:
- Spot (with T-1 → T transition), Change%, Forward
- Impl Vol, Impl CS, CS Used
- Accrued, Carry, BE Call (with rise%), BE Now
- CV BE (call/now), Δ Dime, Capital%, Inv Value

## Key-Value Grid CSS
```css
.kv { display:grid; grid-template-columns:1fr 1fr; gap:1px 10px; }
.kv.three { grid-template-columns:1fr 1fr 1fr; }
.kv-label { font:500 9px sans-serif; color:#64748b; uppercase; }
.kv-val { font:600 12px monospace; color:#e2e8f0; }
.kv-val.lg { font-size:13px; font-weight:700; }
.kv-val.gp { color:#10b981; }
.kv-val.rp { color:#ef4444; }
```
