# Extending the Dashboard

## Adding New Bonds

Add a new dict to `BONDS` list in `sample_data.py`:

```python
{
    "Id": 50099999,
    "Ticker": "NEW.HK",
    "ISIN": "XS9999999999",
    "Issuer": "NewCo",
    "Underlying": "NEW.HK",
    "Ccy": "HKD",
    "Coupon": 0.5,
    "Maturity": "2028-06-15",
    # ... all required fields (see 02_data_model.md)
    "_spot_t1": 50.50,  # default T spot
    "_dy": 0.0, "_ds": 0.0, "_div": 0.0, "_dfx": 0.0,
}
```

Region, rate tenor, and references are auto-derived at the bottom of `sample_data.py`.

## Connecting to Real Data

### Option A: Replace `sample_data.py` with a database query
```python
# In server.py
@app.route("/api/bonds")
def get_bonds():
    bonds = db.query("SELECT * FROM cb_pricing_results WHERE date = ?", today)
    return jsonify(bonds)
```

### Option B: Read from the CB pricing engine JSON output
```python
import json
with open("path/to/CbCalcGraph_output.json") as f:
    raw = json.load(f)
# Map raw fields to the expected schema
```

### Option C: Bloomberg/API feed
```python
# Fetch from Bloomberg via blpapi or REST
# Map to the same field schema
```

## Adding New P&L Components

1. Add the computation in `attribution.py`:
```python
# e.g. Rho P&L
rho_pnl = rho * delta_rate_level
```

2. Add the key to the return dict
3. Add to `PKS` and `PL_LABELS` arrays in `index.html`
4. The grid and charts auto-expand

## Deployment

### Development
```bash
python server.py  # Flask dev server, port 5000
```

### Production
```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:8080 server:app
```

Or with Docker:
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install flask gunicorn
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8080", "server:app"]
```

### Nginx reverse proxy
```nginx
location /cb-attrib/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

## Performance Notes
- Frontend re-renders entire grid HTML on every change — fine for <50 bonds
- For 100+ bonds, consider virtual scrolling or pagination
- API compute is pure Python math — handles 1000 bonds in <100ms
- Canvas waterfall redraws on resize — efficient for any count

## Known Limitations
- No real-time data feed (manual input or replace `sample_data.py`)
- No user authentication
- Single-page (no routing)
- Column widths stored per-browser (localStorage)
- No export to Excel/CSV (add with a `/api/export` route)
