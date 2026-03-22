# Attribution Engine (`attribution.py`)

## Formula

```
ΔCB = δ·ΔS + ½·Γ·(ΔS)² + DV01·Δy + CR01·Δcs + ν·Δσ + θ + CB₀·(ΔFX/FX₀) + ε
```

## Components

| # | Component | Formula | Inputs |
|---|-----------|---------|--------|
| 1 | Delta P&L | `delta × delta_spot` | Delta, ΔSpot |
| 2 | Gamma P&L | `0.5 × (gamma_stk / 100) × delta_spot²` | GammaStk, ΔSpot |
| 3 | Rate P&L | `dv01 × delta_yield_bps` | DV01, Δy (bps) |
| 4 | Credit P&L | `cr01 × delta_spread_bps` | CR01, Δcs (bps) |
| 5 | Vega P&L | `vega × (delta_ivol / 0.01)` | Vega, Δσ (absolute) |
| 6 | Theta P&L | `theta` (1 day) | Theta |
| 7 | FX P&L | `fair_value × (delta_fx / spot_fx)` | FairValue, ΔFX, SpotFX |
| 8 | Residual | `actual_pnl - total_explained` (if actual given) | |

## Function Signature

```python
def compute_attribution(
    delta, gamma_stk, dv01, cr01, vega, theta,  # Greeks from T-1
    fair_value_t0, spot_t0, spot_fx_t0,          # T-1 levels
    delta_spot,          # ΔSpot (absolute)
    delta_yield_bps,     # Δy in basis points
    delta_spread_bps,    # Δcs in basis points
    delta_ivol,          # Δσ absolute (e.g. 0.01 = 1 vol pt)
    delta_fx,            # ΔFX absolute
    actual_pnl=None,     # Optional actual CB price change
) -> dict
```

## Return Value

```python
{
    "Delta P&L (Equity)": float,
    "Gamma P&L (Equity)": float,
    "Rate P&L (DV01)": float,
    "Credit P&L (CR01)": float,
    "Vega P&L (Vol)": float,
    "Theta P&L (Time)": float,
    "FX P&L": float,
    "Total Explained": float,      # sum of above 7
    "Residual (Unexplained)": float, # actual - explained (0 if no actual)
    "Actual P&L": float,            # actual or explained if no actual
}
```

## Key Notes

- All P&L in **price terms per 100 notional**
- `GammaStk` from the pricing engine is pre-scaled ×100, so we divide by 100
- `Vega` is per 1 vol point (0.01 absolute), so Δσ is divided by 0.01
- `DV01` and `CR01` are already signed (negative = price drops when factor rises)
- FX P&L only computed when `spot_fx != 0` and `delta_fx != 0`

## Rate Reference Selection

The rate benchmark is matched to the bond's remaining life:

| Bond Life | Tenor | Bloomberg |
|-----------|-------|-----------|
| < 0.75y | 6M | USGG6M Index |
| 0.75-1.5y | 1Y | USGG1YR Index |
| 1.5-2.5y | 2Y | USGG2YR Index |
| 2.5-3.5y | 3Y | USGG3YR Index |
| 3.5-6.0y | 5Y | USGG5YR Index |
| 6.0-8.5y | 7Y | USGG7YR Index |
| > 8.5y | 10Y | USGG10YR Index |

Logic: the DV01 reflects sensitivity at the bond's cashflow dates, so the benchmark yield should match the bond's duration, not a fixed tenor.
