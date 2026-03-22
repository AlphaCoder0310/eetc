# Interactivity & Layout Persistence

## Draggable Panel Resize

### Implementation: `setupResize(handleId, direction, getTargets, layoutKey)`

```javascript
// Handle element: <div class="resize-h" id="resize-sidebar">
// direction: 'h' (horizontal) or 'v' (vertical)
// getTargets: () => [{ el, dir, min, prop }]
//   el: DOM element to resize
//   dir: 1 (grows with drag) or -1 (shrinks with drag)
//   min: minimum size in px
//   prop: CSS property to set ('width' for h, 'maxHeight' for v)
// layoutKey: key in layoutState for persistence
```

### Active Resize Handles
| Handle ID | Direction | What it resizes | Layout Key |
|-----------|-----------|-----------------|------------|
| `resize-sidebar` | horizontal | Sidebar width | `sidebar` |
| `resize-grid` | vertical | Grid max-height | `gridHeight` |
| `resize-bottom-1` | horizontal | Waterfall panel width | `bottomWaterfall` |
| `resize-bottom-2` | horizontal | Breakdown panel width | `bottomBreakdown` |

### CSS
```css
.resize-h { width:5px; cursor:col-resize; }
.resize-v { height:5px; cursor:row-resize; }
.resize-h:hover, .resize-v:hover { background:var(--cyan); opacity:0.4; }
/* Pseudo-element for visible border line */
.resize-h::after { content:''; position:absolute; left:2px; width:1px;
  top:0; bottom:0; background:var(--border); }
```

## Column Resize

### How it works
1. On `renderGrid()`, each `<th>` in the sub-header row gets a 4px drag handle div appended
2. `mousedown` on handle starts tracking
3. `mousemove` applies new width to `<th>` and all `<td>` in same column index
4. `mouseup` saves width via `setColWidth(colId, finalW)` → `localStorage`
5. Next `renderGrid()` restores widths via `getColWidth(idx, defaultW)`

### Width Storage
```javascript
layoutState.colWidths = {
  "0": 22,    // radio column
  "1": 68,    // Ticker
  "2": 60,    // Issuer
  "3": 120,   // ISIN (user widened)
  ...
  "bd_0": 80, // breakdown Bond column
  "bd_1": 95, // breakdown Delta column
}
```

Main grid columns use numeric string keys (`"0"`, `"1"`, ...).
Breakdown table uses `"bd_0"`, `"bd_1"`, ... to avoid collisions.

## Layout Persistence

### Storage
```javascript
const STORAGE_KEY = 'cb_pnl_layout';
localStorage.setItem(STORAGE_KEY, JSON.stringify(layoutState));
```

### `layoutState` Shape
```javascript
{
  colWidths: { "0": 22, "1": 68, ... },
  sidebar: 175,
  gridHeight: null,          // null = use CSS default (50vh)
  bottomWaterfall: null,     // null = use CSS default (flex:1)
  bottomBreakdown: null,
}
```

### Restore on Load
1. `loadLayout()` reads from `localStorage` on page load
2. `setupResize()` applies saved panel sizes on init
3. `renderGrid()` applies saved column widths via `getColWidth()`

### Reset
"Reset Layout" button in header:
```javascript
localStorage.removeItem('cb_pnl_layout');
location.reload();
```

## Bond Selection

### Sidebar Checkboxes
- Toggle bond in/out of `selected` Set
- Triggers `computeAll()` → `renderAll()`

### ALL / NONE Buttons
```javascript
async function selectAllBonds(selectAll) {
  if (selectAll) bonds.forEach((_, i) => selected.add(i));
  else { selected.clear(); detailIdx = null; }
  await computeAll(); renderAll();
}
```

### Detail Selection (Radio)
- Radio buttons in grid column 0
- `onChange` → set `detailIdx` → `renderAll()`
- Also clickable from sidebar bond rows

## Edit → Compute → Render Cycle
```
1. User edits an input cell (e.g. Spot T)
2. onChange fires
3. Parse value, update moves[idx]
4. If T-level changed, sync delta (and vice versa)
5. POST /api/compute with all selected bonds' current moves
6. Store results
7. renderAll() — rebuilds grid (with saved widths), waterfall, breakdown, detail
```

Average cycle time: ~50ms (network + render) for 6 bonds.
