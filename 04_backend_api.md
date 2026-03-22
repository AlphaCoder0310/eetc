# Backend API (`server.py`)

## Stack
- Python Flask
- Serves `static/index.html` at `/`
- Two JSON API endpoints

## Routes

### `GET /`
Serves `static/index.html`.

### `GET /api/bonds`
Returns the full bond universe with all static fields.

**Response**: JSON array of bond objects (see `02_data_model.md` for fields).
Each bond also includes `default_spot_t1` (the default T spot price).

```json
[
  {
    "Id": 50023748,
    "Ticker": "6886.HK",
    "Issuer": "Meituan",
    "Delta": 0.23666,
    "GammaStk": 21.147,
    "SpotPrice": 15.86,
    "default_spot_t1": 16.019,
    "Region": "HK",
    "RateRef": "US 1Y Tsy (USGG1YR Index)",
    "SpotRef": "3690.HK Equity",
    "CreditRef": "Meituan 5Y CDS / iTraxx Asia IG",
    "VolRef": "CB model-implied vol",
    ...
  },
  ...
]
```

### `POST /api/compute`
Computes P&L attribution for given overnight moves.

**Request body**: JSON array of move objects:
```json
[
  { "idx": 0, "spot_t1": 16.02, "dy": 0.0, "ds": 0.0, "dv": 0.0 },
  { "idx": 1, "spot_t1": 134.84, "dy": 1.5, "ds": 0.0, "dv": 0.0 }
]
```

| Field | Type | Description |
|-------|------|-------------|
| idx | int | Bond index in the `BONDS` array |
| spot_t1 | float | Spot price at T |
| dy | float | Δ yield in basis points |
| ds | float | Δ credit spread in basis points |
| dv | float | Δ implied vol (absolute, e.g. 0.01 = 1 vol pt) |

**Response**: JSON array of results:
```json
[
  {
    "idx": 0,
    "ticker": "6886.HK",
    "delta_spot": 0.16,
    "delta_spot_pct": 1.0088,
    "pnl": {
      "Delta P&L (Equity)": 0.037629,
      "Gamma P&L (Equity)": 0.002673,
      "Rate P&L (DV01)": 0.0,
      "Credit P&L (CR01)": 0.0,
      "Vega P&L (Vol)": 0.0,
      "Theta P&L (Time)": -0.00131,
      "FX P&L": 0.0,
      "Total Explained": 0.038992,
      "Residual (Unexplained)": 0.0,
      "Actual P&L": 0.038992
    }
  }
]
```

## Running
```bash
pip install flask
python server.py
# Runs on http://127.0.0.1:5000 with debug mode
```
