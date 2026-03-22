# Data Model

## Bond Object (`sample_data.py`)

Each bond dict contains these fields from the CB pricing engine:

### Identity
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| Id | int | 50023748 | Internal bond ID |
| Description | str | "6886.HK 0.000 02/08/2027" | Full description |
| Ticker | str | "6886.HK" | Trading ticker |
| ISIN | str | "XS2435678901" | ISIN code |
| Issuer | str | "Meituan" | Issuer name |
| Underlying | str | "3690.HK" | Underlying equity ticker |
| Ccy | str | "HKD" | Currency |
| Coupon | float | 0.0 | Coupon rate (%) |
| Maturity | str | "2027-02-08" | Maturity date |

### Prices
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| FairValue | float | 99.805 | Model fair value (per 100) |
| CBMarketPrice | float | 99.189 | Market price |
| BondFloor | float | 97.481 | Bond floor (straight bond value) |
| InvestmentValue | float | 97.481 | Investment value |
| AccruedInterest | float | 0.0 | Accrued interest |

### Underlying & Conversion
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| SpotPrice | float | 15.860 | Underlying equity spot (T-1 close) |
| Forward | float | 16.249 | Forward price |
| ConversionRatio | float | 5.0761 | Shares per bond |
| Parity | float | 80.508 | Conversion value (Spot × ConvR) |
| SpotFx | float | 1.0 | FX rate (1.0 if same ccy) |
| Moneyness | float | 1.242 | Bond price / Parity |
| CalendarLife | float | 0.904 | Years to maturity |

### Greeks / Sensitivities
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| Delta | float | 0.23666 | dCB/dSpot — first-order equity |
| GammaStk | float | 21.147 | d²CB/dSpot² × 100 — second-order equity |
| DV01 | float | -0.00725 | Price change per 1bp yield move |
| CR01 | float | -0.00722 | Price change per 1bp credit spread move |
| Vega | float | 0.23683 | Price change per 1 vol point |
| Theta | float | -0.00131 | Daily time decay |

### Volatility & Credit
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| ImpliedVolatility | float | 0.2129 | CB model-implied vol (decimal) |
| ImpliedCreditSpread | float | 0.01725 | Implied credit spread (decimal) |
| CreditSpreadUsed | float | 0.0075 | Credit spread used in pricing |

### CB-Specific
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| CmBeCall | float | 21.286 | Breakeven stock price (to call) |
| CmBeNow | float | 20.682 | Breakeven stock price (now) |
| CmRiseBeCall | float | 0.338 | % rise to breakeven |
| CvBeCall | float | 108.052 | CB price at breakeven |
| ChangeDime | float | 0.1218 | Delta for 10% stock move |
| Capital | float | 29.714 | Capital (premium over parity %) |
| Carry | float | -0.2157 | Daily carry |

### Derived Fields (computed at load time in `sample_data.py`)
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| Region | str | "HK" | Derived from Ticker (.HK → HK, else US) |
| RateTenor | str | "1Y" | Matched to CalendarLife (see tenor map) |
| RateRef | str | "US 1Y Tsy (USGG1YR Index)" | Rate benchmark reference |
| SpotRef | str | "3690.HK Equity" | Spot price reference |
| CreditRef | str | "Meituan 5Y CDS / iTraxx Asia IG" | Credit reference |
| VolRef | str | "CB model-implied vol" | Vol reference |
| FxRef | str | "USDHKD Curncy" | FX reference |

### Rate Tenor Mapping
```python
CalendarLife ≤ 0.75y  → 6M  (USGG6M Index)
CalendarLife ≤ 1.5y   → 1Y  (USGG1YR Index)
CalendarLife ≤ 2.5y   → 2Y  (USGG2YR Index)
CalendarLife ≤ 3.5y   → 3Y  (USGG3YR Index)
CalendarLife ≤ 6.0y   → 5Y  (USGG5YR Index)
CalendarLife ≤ 8.5y   → 7Y  (USGG7YR Index)
CalendarLife > 8.5y   → 10Y (USGG10YR Index)
```

### Default Overnight Moves (prefixed with `_`)
| Field | Type | Description |
|-------|------|-------------|
| _spot_t1 | float | Default Spot(T) = SpotPrice × 1.01 |
| _dy | float | Default Δyield = 0 bps |
| _ds | float | Default Δspread = 0 bps |
| _div | float | Default Δvol = 0 |

## Frontend State

### `moves` Object (per bond index)
```javascript
moves[idx] = {
  spot_t1: 16.02,    // Spot price at T (editable)
  dy: 0.0,           // Δ yield in bps (editable, syncs with yld_t)
  ds: 0.0,           // Δ credit spread in bps (editable, syncs with cs_t)
  dv: 0.0,           // Δ implied vol absolute (editable, syncs with vol_t)
  yld_t: 172.5,      // Yield at T in bps (editable, syncs with dy)
  cs_t: 172.5,       // Credit spread at T in bps (editable, syncs with ds)
  vol_t: 0.2129,     // Vol at T absolute (editable, syncs with dv)
}
```

**Sync logic**: Editing `yld_t` sets `dy = yld_t - yld0`. Editing `dy` sets `yld_t = yld0 + dy`. Same for cs/vol pairs.

### `results` Object (per bond index, from API)
```javascript
results[idx] = {
  idx: 0,
  ticker: "6886.HK",
  delta_spot: 0.159,       // Spot T - Spot T-1
  delta_spot_pct: 1.003,   // % change
  pnl: {
    "Delta P&L (Equity)": 0.037629,
    "Gamma P&L (Equity)": 0.002673,
    "Rate P&L (DV01)": 0.0,
    "Credit P&L (CR01)": 0.0,
    "Vega P&L (Vol)": 0.0,
    "Theta P&L (Time)": -0.00131,
    "FX P&L": 0.0,
    "Total Explained": 0.038992,
    "Residual (Unexplained)": 0.0,
    "Actual P&L": 0.038992,
  }
}
```

### `layoutState` (persisted to localStorage)
```javascript
{
  colWidths: { "0": 22, "1": 68, "3": 120, ... },  // column index → width px
  sidebar: 175,        // sidebar panel width
  gridHeight: null,    // grid max-height (null = default 50vh)
  bottomWaterfall: null, // waterfall panel width
  bottomBreakdown: null, // breakdown panel width
}
```
