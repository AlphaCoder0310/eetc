# CB P&L Attribution Dashboard — Overview

## Purpose
Overnight P&L attribution dashboard for a convertible bond trading desk. Decomposes the change in each CB's fair value into contributions from underlying risk factors (equity, rates, credit, volatility, theta, FX).

## Tech Stack
- **Backend**: Python 3 + Flask (serves REST API + static files)
- **Frontend**: Single-file vanilla HTML/JS/CSS (no framework, no build step)
- **Charts**: HTML5 Canvas API (waterfall chart)
- **Compute Engine**: Python (`attribution.py`) — pure math, no external libs
- **Data**: Python dicts in `sample_data.py` (mock data; in production, replace with API/DB)
- **Layout Persistence**: Browser `localStorage`

## File Structure
```
cb_attribution_pnl/
├── server.py           # Flask app — API routes + static file serving
├── attribution.py      # P&L attribution engine (pure Python math)
├── sample_data.py      # Bond universe with mock data + region/rate tenor derivation
├── requirements.txt    # Python deps (flask, pandas, numpy, etc.)
├── static/
│   └── index.html      # Complete frontend (HTML + CSS + JS in one file)
└── docs/
    ├── 01_overview.md          # This file
    ├── 02_data_model.md        # Bond fields, moves state, results schema
    ├── 03_attribution_engine.md # Math formulas and compute logic
    ├── 04_backend_api.md       # Flask routes and request/response formats
    ├── 05_frontend_layout.md   # Three-panel layout, CSS architecture
    ├── 06_grid_component.md    # Unified grid: columns, editing, frozen cols
    ├── 07_detail_panel.md      # Right panel: collapsible sections, mini-bars
    ├── 08_charts.md            # Waterfall chart canvas rendering
    ├── 09_interactivity.md     # Resize handles, column drag, layout persistence
    └── 10_extending.md         # How to add bonds, connect real data, deploy
```

## How to Run
```bash
pip install flask
cd cb_attribution_pnl
python server.py
# Open http://localhost:5000
```
