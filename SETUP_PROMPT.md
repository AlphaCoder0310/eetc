# CB Gamma Trading Monitor — Full Build Prompt

Use this prompt with Claude Sonnet 4.5 to recreate the entire project from scratch. Copy everything below the line and paste it as your message.

---

Build me a convertible bond gamma trading monitor — a single-screen Bloomberg-terminal-style dashboard using React (Vite) frontend with D3.js charts. No backend needed — all analytics run client-side with sample data. Create every file I specify below, exactly as shown.

## Project Structure

```
gamma-monitor/
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── index.css
│       ├── App.jsx
│       ├── sampleData.js
│       ├── hooks/
│       │   └── useMarketSimulator.js
│       └── components/
│           ├── charts.jsx
│           └── panels/
│               ├── TopBar.jsx
│               ├── BondListPanel.jsx
│               ├── CenterPanel.jsx
│               └── RightPanel.jsx
```

## Step 1: Create `frontend/package.json`

```json
{
  "name": "cb-gamma-monitor",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "d3": "^7.9.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "recharts": "^2.12.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^5.4.0"
  }
}
```

## Step 2: Create `frontend/vite.config.js`

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': 'http://localhost:5000'
    }
  }
})
```

## Step 3: Create `frontend/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CB Gamma Trading Monitor</title>
    <link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600;700&family=DM+Sans:wght@400;500;600;700&display=swap" rel="stylesheet" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

## Step 4: Create `frontend/src/main.jsx`

```javascript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

## Step 5: Create `frontend/src/App.jsx`

```javascript
import React, { useState } from 'react'
import { useMarketSimulator } from './hooks/useMarketSimulator'
import TopBar from './components/panels/TopBar'
import BondListPanel from './components/panels/BondListPanel'
import CenterPanel from './components/panels/CenterPanel'
import RightPanel from './components/panels/RightPanel'

export default function App() {
  const [selectedIdx, setSelectedIdx] = useState(2) // Meituan (balanced zone)
  const { portfolio, tick, alerts } = useMarketSimulator({ intervalMs: 2000 })

  return (
    <div className="terminal-grid">
      <TopBar portfolio={portfolio} alerts={alerts} />
      <BondListPanel portfolio={portfolio} selectedIdx={selectedIdx} onSelect={setSelectedIdx} />
      <CenterPanel bond={portfolio[selectedIdx]} tick={tick} />
      <RightPanel portfolio={portfolio} selectedIdx={selectedIdx} alerts={alerts} />
    </div>
  )
}
```

## Step 6: Create `frontend/src/sampleData.js`

This file contains 5 sample convertible bonds (HUATAI, JD.COM, MEITUAN, TENCENT, XIAOMI) with full pricing data, and an `analyzeBond()` function that computes all derived analytics client-side.

Each bond object has these fields:
- Identifiers: id, description, issuer, currency, coupon, maturity, scenario, timestamp
- Price: cbMarketPrice, fairValue, economicFairValue, fairPrice, accruedInterest, accruedPrice
- Parity: parity, conversionRatio, moneyness, spotPrice, spotFx
- Credit: bondFloor, economicBondFloor, investmentValue, creditSpreadUsed, impliedCreditSpread
- Greeks: delta, gamma, gammaStk, eqCashDelta, eqCashGamma, theta, vega, rho, omega
- Volatility: impliedVolatility (as percentage, e.g. 21.29 means 21.29%)
- Yield: currentYield, cmYield, ytm, ytc, carry, yieldAdvantage, investmentYield
- Premium: pointPremium, premiumToParity, premiumToInvestment, be, cmBeCall, cmBeNow, cvBeCall, cvBeNow
- Option: optionValue
- Duration: cr01, dv01, fugit
- Time: yearsToCall, calendarLife, forward
- Hedge: sharesShort, capital
- Realized vol: realizedVol10d, realizedVol20d, realizedVol30d, realizedVol60d, listedOptionsIv
- Position: notional, currentHedgeDelta, borrowCost

Sample bond data (use these exact values):

**Bond 1 - HUATAI SECURITIES**: id=50023748, desc="6886.HK 0.000 02/08/2027", HKD, coupon=0, spot=15.86, cbMkt=99.11, fv=99.805, parity=80.508, cr=5.076, moneyness=1.242, bondFloor=97.481, delta=0.2367, gamma=1.3291, gammaStk=21.147, eqCashDelta=19.053, theta=-0.00131, vega=0.2368, iv=21.29, rv10d=24.5, rv20d=26.1, rv30d=23.8, rv60d=25.3, listedIv=28.5, fugit=0.897, ytc=0.901, notional=5M, hedgeDelta=0.228, borrow=0.003

**Bond 2 - JD.COM**: id=50034892, desc="9618.HK 0.000 06/15/2028", spot=148.30, cbMkt=103.25, fv=105.12, parity=92.45, cr=6.234, moneyness=1.082, bondFloor=94.22, delta=0.4812, gamma=2.1534, gammaStk=34.28, eqCashDelta=44.51, theta=-0.00287, vega=0.4521, iv=31.45, rv20d=35.6, listedIv=34.2, fugit=1.85, ytc=2.25, notional=8M, hedgeDelta=0.465, borrow=0.005

**Bond 3 - MEITUAN**: id=50041567, desc="3690.HK 0.500 09/20/2028", coupon=0.5, spot=245.40, cbMkt=108.45, fv=110.82, parity=101.23, cr=4.125, moneyness=0.988, bondFloor=92.15, delta=0.5534, gamma=1.8721, gammaStk=29.78, eqCashDelta=56.02, theta=-0.00342, vega=0.5234, iv=28.72, rv20d=32.5, listedIv=30.1, fugit=2.12, ytc=2.51, notional=10M, hedgeDelta=0.541, borrow=0.004

**Bond 4 - TENCENT**: id=50052341, desc="700.HK 0.000 12/01/2027", spot=456.10, cbMkt=106.78, fv=107.95, parity=97.85, cr=2.145, moneyness=1.022, bondFloor=95.67, delta=0.6245, gamma=1.5432, gammaStk=24.55, eqCashDelta=62.45, theta=-0.00298, vega=0.4123, iv=25.34, rv20d=27.4, listedIv=26.8, fugit=1.45, ytc=1.71, notional=12M, hedgeDelta=0.610, borrow=0.002

**Bond 5 - XIAOMI**: id=50063218, desc="1810.HK 0.000 03/28/2029", spot=10.13, cbMkt=101.55, fv=103.89, parity=85.67, cr=8.456, moneyness=1.167, bondFloor=93.45, delta=0.3876, gamma=2.4532, gammaStk=39.04, eqCashDelta=31.25, theta=-0.00198, vega=0.5678, iv=35.67, rv20d=39.8, listedIv=38.5, fugit=2.65, ytc=3.03, notional=6M, hedgeDelta=0.372, borrow=0.006

The `analyzeBond(bond)` function computes:
1. **Zone classification**: BUSTED (<0.20 delta), BOND-LIKE (0.20-0.35), BALANCED (0.35-0.65), EQUITY-LIKE (0.65-0.85), IN-THE-MONEY (>0.85)
2. **Breakeven move**: sqrt(2 * |theta| / gamma)
3. **Gamma/Theta ratio**: |gamma / theta|
4. **P&L decomposition**: gammaPnl = 0.5 * gamma * 1^2 * notional/100; thetaCost = theta * notional/100; carry components using 4.5% funding, 4.0% short rebate, bond-specific borrow cost
5. **Cheapness**: cheapRatio = (fairValue - mktPrice) / mktPrice * 100; signals: >5%=COMPELLING, >2%=INTERESTING, >0=SLIGHTLY CHEAP, >-2%=FAIR, else=RICH
6. **IV comparison**: ivDiscount = listedOptionsIv - impliedVolatility; rvIvSpread = realizedVol20d - impliedVolatility
7. **Gamma score (0-100)**: cheapness(0-25) + volSpread(0-25) + gammaQuality(0-20 by zone) + carry(0-15) + structural(0-15 by fugit+floorDistance)
8. **Rebalance signal**: Whalley-Wilmott optimal band = (3*0.0005*spot / (2*gamma*notional/100))^(1/3), clamped to [0.01, 0.10]; urgency based on drift vs band multiples
9. **Vol scenarios**: P&L at realized vols [15,20,25,30,35,40,50]% over 30 days
10. **Gamma profile**: delta and gammaPnl across ±20% spot range (41 points)

Returns all original bond fields plus all computed fields. Also export `getAnalyzedPortfolio()` that maps all SAMPLE_BONDS through analyzeBond.

## Step 7: Create `frontend/src/hooks/useMarketSimulator.js`

Custom hook that simulates live market data:
- Every 2 seconds (configurable via intervalMs), perturbs each bond's spotPrice with a vol-scaled random walk (Box-Muller normal distribution, scaled to tick interval using annual vol / sqrt(252 * ticks_per_day))
- CB market price moves correlated to spot via delta with noise
- Realized vols drift slightly each tick
- Parity updates with spot
- Fair value drifts slightly
- Re-runs analyzeBond() on each perturbed bond
- Detects urgency escalations (to HIGH) and zone changes, pushes alerts
- Audio beep (880Hz, 150ms via Web Audio API) on HIGH alerts
- Returns { portfolio, tick, alerts }
- Alerts: array of { ts (HH:MM:SS), severity, message, key }, max 50

## Step 8: Create `frontend/src/components/charts.jsx`

D3-based chart components:

### Sparkline
- Props: data (array of numbers), width=60, height=20, color, showArea=true
- D3 line + area with CatmullRom curve, SVG with gradient fill

### MiniBar
- Props: value, max, color, width=60
- Simple div bar showing proportional width

### ScoreRadar
- Props: data (object with cheapness, volSpread, gammaQuality, carry, structural), size=160
- 5-axis spider chart (CHP/VOL/GQ/CRY/STR) with max values [25,25,20,15,15]
- Grid rings at 25/50/75/100%, axis lines, polygon fill rgba(6,182,212,0.12), dots at vertices

### IVSurface
- Props: impliedVol (base value)
- 5x5 heatmap grid: moneyness [90-110%] × tenors [1M-2Y]
- Synthetic surface centered on bond's IV with smile and term structure
- Green (low) → amber → red (high) color scale

### ProfileChart
- Props: profile (from analyzeBond), height=200
- D3 SVG with dual Y-axis: left=Delta (0-1, cyan line + area fill), right=Gamma P&L (amber dashed line)
- X-axis: spot move % (-20 to +20), grid lines, zero line dashed
- Current spot marker dots, legend

## Step 9: Create `frontend/src/components/panels/TopBar.jsx`

- Portfolio-level aggregates: Notional, Gamma P&L(1%), Theta/D, Carry/D, Net/D, Vega
- Live clock (HH:MM:SS HKT format, updates every second)
- HIGH urgency count with pulsing red badge, REBAL count with amber badge
- Brand: "CB GAMMA MONITOR v2.1"

## Step 10: Create `frontend/src/components/panels/BondListPanel.jsx`

- Scrollable list of bonds with colored left border when selected
- Each row: issuer name (colored by bond index), ticker, delta/gamma with Greek symbols, gamma score badge, sparkline
- Bottom row: shares short, urgency badge
- Below list: IV Surface heatmap for selected bond
- Bond accent colors: ['#10b981', '#f59e0b', '#06b6d4', '#a78bfa', '#64748b']

## Step 11: Create `frontend/src/components/panels/CenterPanel.jsx`

The main detail view:
1. **Security header**: issuer name (large), description, currency, coupon, maturity | spot/mkt/FV prices | zone badge
2. **Greeks row**: 5 cards with colored top-border (Delta=cyan 20px bold, Gamma=amber, Theta=red, Vega=purple, ImplVol=white), each with dollar sensitivity sub-label
3. **Three-column content**:
   - P&L Decomposition with MiniBar next to each value (Gamma PnL, Short Rebate, Theta Cost, Funding, Borrow, Net Carry, TOTAL)
   - Key Metrics (Breakeven Move, Gamma/Theta, Moneyness, Parity, Bond Floor, Floor Distance, Prem/Parity, Fugit, YTC)
   - Cheapness & Carry (Signal badge, Cheap Ratio, $ Cheapness, IV Discount, RV-IV spread, RV Signal badge, Funding, Short Rebate, Net Carry annual)
4. **ProfileChart** (Delta/Gamma profile, ±20%)

## Step 12: Create `frontend/src/components/panels/RightPanel.jsx`

Stacked sections:
1. **Rebalance Signals**: sorted by urgency (HIGH first), color-coded rows (green=BUY, red=SELL, grey=OK), pulsing animation on HIGH, shows shares to trade and delta drift
2. **Gamma Score Ranking**: sorted descending, rank numbers (cyan for #1, amber for #2-3), sparklines
3. **Score Decomposition Radar**: ScoreRadar component for selected bond
4. **Score Breakdown Table**: color-coded values (>=70% green, >=40% cyan, >=25% amber, <25% grey)
5. **Alert Log**: timestamped alerts (max 20 shown), severity badges

## Step 13: Create `frontend/src/index.css`

Dark terminal theme with these design principles:
- Background: #0b0e17
- Typography: IBM Plex Mono for values, DM Sans for labels
- Contrast hierarchy: primary values #e2e8f0, secondary #94a3b8, labels #64748b (never darker)
- Greek colors: Delta=cyan(#06b6d4), Gamma=amber(#f59e0b), Theta=red(#ef4444), Vega=purple(#a78bfa)
- CSS Grid layout: 3 columns (224px | 1fr | 278px), 2 rows (42px top bar | 1fr content), 100vh, no scrolling
- Grid areas: topbar (full width), left, center, right
- Cards: rgba(255,255,255,0.02) bg, rgba(255,255,255,0.06) border, 8px border-radius
- Greek cards: ::after pseudo-element for 2px colored top border
- Rebalance signals: .sig-row.buy (green bg), .sig-row.sell (red bg), .sig-row.ok (grey)
- Animations: urgencyPulse (pulsing red glow, 1.2s infinite), flashGreen/flashRed for value changes
- Zone badges: BALANCED=cyan, BOND-LIKE/EQUITY-LIKE=amber, BUSTED/IN-THE-MONEY=red
- Thin 4px scrollbar with rgba(255,255,255,0.08)
- Responsive: 1200px breakpoint reduces column widths

## How to Run

```bash
cd frontend
npm install
npm run dev
```

Then open http://localhost:3000. The dashboard loads immediately with simulated live data — no backend needed. Spot prices walk randomly every 2 seconds, alerts fire when bonds need rebalancing.
