# GMMA Price Action MTF Engine (TradingView)

Pine Script v6 overlay that combines **GMMA (Guppy Multiple Moving Average)**, **multi-timeframe bias**, **momentum filters**, and **price-action rules** into a **0–100 score** that gates **LONG** and **SHORT** signals with optional **entry, stop-loss, and take-profit** lines.

**Source file:** `gmma_indicator.pine.txt` (copy its contents into TradingView; the `.txt` suffix is for easy editing outside TradingView).

---

## How to add the indicator in TradingView

1. Open [TradingView](https://www.tradingview.com/) and open a chart.
2. Click **Pine Editor** at the bottom (or **Indicators → Create** in some layouts).
3. Open **New** → **Blank indicator** (or clear the default template).
4. Open `gmma_indicator.pine.txt` in this repo, **select all**, **copy**, and **paste** into the Pine Editor.
5. Click **Save** (give it a name, e.g. `GMMA MTF Engine v6`), then **Add to chart**.

The script uses `//@version=6`. Your account must support **Pine Script v6** (current TradingView plans do).

---

## What you see on the chart

| Element | Purpose |
|--------|---------|
| **Fast / slow EMA ribbons** | Classic GMMA: stacked fast group vs slow group; color reflects bull/bear/neutral read. |
| **EMA 200** | Long-term trend filter and context. |
| **Supertrend** | Trend filter aligned with the score engine. |
| **Pivot high / low lines** | Last confirmed pivot levels used for BOS and structure. |
| **LONG / SHORT labels** | Fire when **entry logic**, **filters**, **minimum score**, and **bar confirmation** (if enabled) all agree—only on the **first bar** of the setup. |
| **Entry / SL / TP lines + label** | Shown when a signal fires (if enabled); SL mode and R:R come from settings. |
| **Trend background** | Optional green/red tint from GMMA state. |
| **Score table** (top right) | Live read of long/short scores, MTF bias, momentum, price action, and quality labels. |

---

## Main settings (quick reference)

### Trading behavior

- **Trading Mode**
  - **Aggressive:** More triggers (e.g. GMMA cross, BOS, pullback, inside break).
  - **Balanced:** BOS, retest, or pullback-style entries.
  - **Conservative:** Stricter combinations (e.g. retest-focused, or BOS + engulf, pullback + pin).
- **Only Confirm On Candle Close:** When on, signals evaluate on the **closed** bar (fewer repaints on the forming candle).
- **Min Long Score / Min Short Score:** Signals require the score to be at least this value (default **75**).

### Display toggles

Turn ribbons, EMA200, Supertrend, signals, trade levels, score table, and background on or off from the indicator inputs.

### Multi-timeframe (MTF)

- **Use MTF Engine:** Pulls bias from two higher timeframes (default **5** and **15** minutes; change to fit your chart, e.g. on a daily chart use **W** and **M**).
- **MTF Weight TF1 / TF2:** Extra points when that timeframe is **fully aligned** (GMMA + EMA200 + Supertrend + RSI midpoint + ADX floor).
- **Conflict:** If higher timeframes align **against** your direction, the score is **penalized** (reduces chasing counter-trend).

### Risk

- **Stop Loss Mode:** **ATR** (multiple of ATR), **Swing** (recent low/high over lookback), or **Slow Band** (uses slow GMMA line `emaS1`).
- **Risk Reward (`rr`):** Take-profit distance as a multiple of risk from entry to stop.

### Filters (all optional via toggles)

Core: GMMA, Supertrend, EMA200, RSI, MACD, ADX, optional volume and ATR floor.

Price action: **BOS** (break of last pivot), optional **retest**, candle confirmation (engulf / pin), optional inside-bar break.

When a filter is **on**, it must pass (along with the BOS/retest logic as coded) for a signal to print.

---

## Understanding the score (not machine learning)

Long and short scores are **rule-based**: points are added or subtracted for trend alignment, momentum, price action events, ribbon width, volatility regime, stretch vs EMA200, CHoCH-style context, and MTF weights/penalties. Scores are capped between **0** and **100**. The table’s **Signal Grade** and **Entry Quality** summarize that snapshot for quick scanning.

---

## Alerts

In the chart’s **Alerts** panel, create an alert on this indicator and choose:

- **Long Signal** / **Short Signal**
- **Bull BOS** / **Bear BOS**

Use your preferred notification channel (app, email, webhook, etc.) in TradingView’s alert dialog.

---

## Repository layout

```
Indicator-tdv/
├── gmma_indicator.pine.txt   # Full Pine v6 source (paste into TradingView)
└── README.md                 # This guide
```

---

## Disclaimer

This indicator is for **education and research** only. It is not financial advice. Past performance of any script does not guarantee future results. Always manage risk and comply with your jurisdiction’s rules.
