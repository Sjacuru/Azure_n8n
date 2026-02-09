# Money Scanner Extension — PRD v1

## Mission

As an advanced trader, I want extra insight into everything I care about when deciding on an entry position. Right now I only have trend, time since flip, and performance % since flip. I want all info in one place at one glance.

## Problem

The Money Scanner doesn't show everything needed for quick trading decisions. Extending it creates clutter. The business owner already relaunched a stripped-down version because newcomers get overwhelmed. We cannot add complexity to the main UI without friction.

## Solution

A **browser extension** (Chrome, Manifest V3) that enhances the existing Money Scanner page at bullmania.com. Advanced metrics are **hidden by default** behind a collapsible row per asset. The extension must strictly follow the existing UI language: no new interaction patterns!

## Target Users

Advanced traders in a paid community who need quick, multi-metric trading decisions.

## Features (Priority Order)

### 1. Trading Volume / Market Cap Ratio

**Why**: Validates price movements. Identifies untradeable coins (large cap, tiny volume) and anomalies (single large order moving price).

**What**:

- Ratio: `(24h Volume / Market Cap)` displayed in %
- Traffic light visualization (green / yellow / red)
- Thresholds: TBD — suggested starting point: Green >5%, Yellow 1–5%, Red <1%

**Data**: CoinGecko API

### 2. Resistance & Support Levels

**Why**: Distance to R/S levels determines potential gain vs risk and helps compare assets for trade selection.

**What**:

- Show **resistance** in bull trend, **support** in bear trend
- PoC: Top 1 level only
- Two analyses per asset:
  - Weekly timeframe (full lifetime history)
  - Daily timeframe (last 365 days)
- Displayed as % distance from current price

**Detection method**: Chart screenshot → LLM analysis using a standardized prompt that codifies rules for identifying local tops/bottoms.

**Data**:

- Charts: TradingView screenshots (weekly full history + daily 365d)
- Analysis: ChatGPT API
- **User must provide their own ChatGPT API key** (stored in extension settings)

### 3. ATR (Average True Range)

**Why**: Sets correct SL/TP levels and filters out low-quality trades (low ATR = unattractive).

**What**:

- Metric: `ATR(14) / Mark Price` displayed as ±%
- This is directly usable as SL/TP input on exchanges
- Timeframe: Reads the active timeframe from the Money Scanner page (4h / Daily / Weekly / Monthly)
- Default: 1× ATR(14)
- Clickable toggle **per asset**: 1× → 1.5× → 2× → loop back (this is a v2 feature, in the PoC/v1 we will just show the 1x ATR in %)

**Data**: CoinGecko OHLC data, calculated client-side

### 4. Perp-Dex Jump-Off Button

**Why**: Quick access to the best perpetual futures market for an asset.

**What**:

- Button **inside the expanded row** (not in the main scanner row)
- Links to the perp DEX listing the asset with the highest 24h volume
- Logic:

  1. Query CoinGecko for asset's markets
  2. Filter: DEX + Perpetuals
  3. Sort by 24h volume descending
  4. Link directly to the #1 result's trading page

**Data**: CoinGecko markets API

## UI Behavior

### Expand / Collapse

- Small **chevron icon** per row (collapsed by default)
- Click chevron → expanded row appears below the asset showing all metrics
- Click again → row collapses
- State resets on page refresh

### Design Rules

- Must match Money Scanner's existing visual style, colors, spacing, typography
- No new interaction patterns
- Non-advanced users should not notice the extension unless they look for the chevron

## Asset Mapping

The extension must reliably map each scanner row's token to the correct CoinGecko coin ID. This mapping is required to fetch volume, market cap, OHLC, and perp-dex market data from the CoinGecko API. Charts for R/S analysis are sourced separately from TradingView.

## Data Fetching Behavior

- Fetch data **only on expand** (not on page load)
- Show loading spinner while fetching
- Keep data until manually collapsed; no auto-refresh
- Cache R/S analysis results in localStorage until browser session closed (cost optimization for LLM calls)

## Security

The perp-dex jump-off button links users directly to external trading platforms. A malicious or compromised link could send users to a fraudulent exchange, resulting in loss of funds. This is the **top security concern** of the entire extension. Mitigations:

- Code must be open-source and peer-reviewable
- Link generation logic must be transparent and auditable
- Links must only come from verified CoinGecko market data

## Community Positioning

The extension may be perceived as intrusive inside a paid community. Mitigations:

- Position as a **community enhancement tool**, not a competing product
- Opt-in only (user installs voluntarily, Ivan can cancel us anytime and we will not show any resistance or grieving)
- Open-source for transparency

## Open Decisions

| # | Question | Status |
|---|----------|--------|
| 1 | Traffic light thresholds for Vol/MCap | TBD — needs real data validation |
| 2 | R/S prompt engineering | Needs iterative testing with actual charts |
| 3 | Fallback behavior on API failure/timeout | TBD |
| 4 | Distribution method | Chrome Web Store vs manual .crx install |
| 5 | TradingView screenshot feasibility | TradingView may block automated capture — fallback needed |
