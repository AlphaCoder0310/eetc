# Charts — Waterfall (Canvas API)

## Overview
Portfolio-level P&L waterfall chart drawn on HTML5 `<canvas>`. No chart library — hand-drawn with Canvas 2D API in `renderWaterfall()`.

## Data
Aggregates P&L across all selected bonds:
```javascript
const vals = PKS.map(k => sel.reduce((s,i) => s + results[i].pnl[k], 0));
vals.push(residual_sum);
const labels = ["Delta","Gamma","Rates","Credit","Vega","Theta","FX","Resid","TOTAL"];
```

## Waterfall Logic
```
cumulative = 0
for each component (except TOTAL):
  bar.start = cumulative
  bar.end = cumulative + value
  cumulative += value

TOTAL bar:
  bar.start = 0
  bar.end = total
```

## Rendering
1. **Canvas sizing**: Width = parent container width, Height = 220px
2. **Padding**: left 40px, right 10px, top 20px, bottom 30px
3. **Y-axis scale**: Auto from min/max of all bar endpoints × 1.15
4. **Grid lines**: 4 horizontal lines, subtle (rgba 0.06)
5. **Zero line**: Slightly brighter (rgba 0.12)

### Bars
- **Positive**: `#10b981` (green)
- **Negative**: `#ef4444` (red)
- **Total**: `#06b6d4` (cyan)
- Width: 60% of column width
- Connectors: Lines from bar top to next bar's starting level

### Labels
- **Value labels**: Above/below bars, 9px monospace, `+0.0000` format
- **Category labels**: Below x-axis, 8px sans-serif, muted color

## Auto-Resize
```javascript
window.addEventListener('resize', () => renderWaterfall());
// Also called after panel resize drag
```

## Breakdown Table
Rendered by `renderBreakdown()` as a standard `.mg` table:
- Columns: Bond | Delta | Gamma | Rates | Credit | Vega | Theta | FX | Total
- Column widths saved with `bd_` prefix keys (e.g. `bd_0`, `bd_1`)
- Has its own column resize handles
- Portfolio total row at bottom
- Selected bond row highlighted
