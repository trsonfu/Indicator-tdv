# GMMA Price Action AI-Style MTF Engine v6 — Tutorial

This README is a **hands-on tutorial** for the TradingView indicator in this repo. It matches how the **Pine Script v6** source actually behaves (`gmma_indicator.pine.txt`).

**Source file:** `gmma_indicator.pine.txt` — copy its entire contents into TradingView’s Pine Editor (the `.txt` suffix is only for editing outside TradingView).

<img width="2949" height="1325" alt="Chart example" src="https://github.com/user-attachments/assets/23a8f6ff-69e6-41f8-ab02-d9c6832645d8" />

---

## 1. What you are installing

You are not installing a single “buy when line crosses” script. The engine is a **multi-framework decision stack** that tries to answer, in order:

1. **Trend** — Is bias bullish, bearish, or mixed (GMMA, Supertrend, EMA200)?
2. **Momentum** — Is the move backed by force (RSI, MACD, ADX/DMI)?
3. **Structure & price action** — Breaks, retests, pullbacks, candle shapes?
4. **Regime** — Expanding vs compressing volatility, ribbon width, stretch vs mean?
5. **Higher timeframes** — Do two bias timeframes support or fight the idea?
6. **Only then** — Should a **LONG/SHORT** label appear, with **entry / SL / TP** hints?

**“AI-style”** here means: many features are **combined, weighted, and penalized** into **long** and **short** scores (0–100), plus **grades** and **entry quality** labels. There is **no machine learning** inside Pine; it is a transparent rule engine.

---

## 2. Install on TradingView

1. Open [TradingView](https://www.tradingview.com/) and a chart.
2. Open **Pine Editor** (bottom) → **New** blank indicator.
3. Paste the full contents of `gmma_indicator.pine.txt`.
4. **Save**, then **Add to chart**.

Requires **Pine Script v6**. The script is an **overlay** (`overlay=true`).

---

## 3. How to use it (recommended workflow)

Treat the **Score Table** (top right) as the main interface. **Read context before chasing a label.**

Suggested order:

1. **Bias** — Long vs short edge (score gap).
2. **Trend** — Bull / Bear / Mixed (GMMA + Supertrend + EMA200 agreement).
3. **MTF Bias** — Full HTF align, partial, or neutral.
4. **Structure** — HH/HL vs LH/LL, transition, reversal attempts.
5. **Price Action** — BOS, retest, engulf, pin, inside break, CHoCH hints.
6. **Momentum** — RSI + MACD + ADX story.
7. **Condition** — Expansion, compression, tight/choppy.
8. **Stretch** — Overextension vs normal (chase risk).
9. **Entry Quality** — Good / Early/Good / Late/Chase / Weak.
10. **Signal** — Only if **mode + filters + min score + bar rule** all pass.

**Decision support, not autopilot:** Prefer trades when the table tells a **coherent story** (aligned trend, momentum, PA, and HTF), and skip marginal rows (mixed trend, neutral MTF, compression, late quality, low grade).

---

## 4. Core building blocks (what the script actually computes)

### 4.1 GMMA (trend core)

- **Fast EMAs:** 3, 5, 8, 10, 12, 15 (configurable).
- **Slow EMAs:** 30, 35, 40, 45, 50, 60 (configurable).

**Bullish GMMA:** average of fast group **>** average of slow group **and** each group is **perfectly stacked** in ascending order (fast ribbon) / descending for bearish.

**Cross events:** `fastAvg` vs `slowAvg` crossover / crossunder (used in **Aggressive** mode).

**Ribbon separation** (as % of price):

- **Strong ribbon:** separation ≥ **0.15%** → adds score (clearer trend space).
- **Tight ribbon:** separation **< 0.08%** → subtracts score (chop / weak separation).

### 4.2 EMA200

- Price **above** EMA200 → long-side environment; **below** → short-side.
- Used in filters, trend text, MTF alignment, and **stretch**: if price is far from EMA200 **relative to ATR%**, the engine marks **overextended** and **penalizes** score (anti-chase).

### 4.3 Supertrend

Confirms trend direction with the built-in Supertrend. Bull/bear states feed **score**, **filters**, **trend text**, and **MTF alignment**.

### 4.4 RSI, MACD, ADX/DMI

- **RSI:** long-friendly if RSI ≥ **55** (default); short-friendly if RSI ≤ **45** (default).
- **MACD:** bull if MACD line **>** signal **and** histogram **≥ 0**; bear mirrored with histogram **≤ 0**.
- **ADX:** trend force filter; **+DI > -DI** or **-DI > +DI** with ADX ≥ **min ADX** (default 20) for directional pressure.

### 4.5 Volume & ATR floor (optional filters)

- **Volume:** bar volume **>** SMA(volume) × multiplier (default 1.0) when the volume filter is on.  
  *Note:* In the score engine, **high volume adds the same bonus to both long and short** when `volBull` is true (participation, not direction).
- **ATR floor:** current ATR vs a baseline SMA × multiplier — filters **dead** volatility when enabled.

### 4.6 Price action layer

- **Pivots:** pivot high/low over `pivotLen` (default 3) — last pivot levels are drawn and drive **BOS** and retest logic.
- **BOS (break of structure):** close **breaks** the last pivot high (bull) or low (bear) with the prior close on the other side.
- **Retest:** prior bar broke the level; current bar pulls back within an **ATR × buffer** zone and closes back on the trend side.
- **Pullback:** in GMMA bull/bear context, price vs slow ribbon + **cross** of close vs **fast EMA #3** (`emaF3`).
- **Candles:** engulfing, pin/rejection, inside-bar breakout.
- **Dow-style:** simplified HH/HL vs LH/LL style read from swings + structure flags.
- **CHoCH-style hints:** pivot break **against** current GMMA state — small score nudge and structure text (“reversal try”).

### 4.7 Compression / expansion

Uses **ATR%** vs its own 20-bar SMA:

- **Compression** if ATR% **<** SMA(ATR%, 20) × **0.85** → score penalty.
- **Expansion** if ATR% **>** SMA(ATR%, 20) × **1.15** → score bonus.

Table **Condition** combines this with tight ribbon for **“Tight / Choppy”** when relevant.

---

## 5. Multi-timeframe (MTF) engine

With **Use MTF Engine** on, the script calls `request.security` on **Bias TF 1** and **Bias TF 2** (defaults **5** and **15** — **always set these higher than or appropriate to your chart timeframe**).

**Aligned long** on an HTF requires **all** of:

- HTF **GMMA bull**
- HTF **close > EMA200**
- HTF **Supertrend bull**
- HTF **RSI ≥ 50**
- HTF **ADX ≥ min ADX**

**Aligned short** is the mirror (GMMA bear, close < EMA200, Supertrend bear, RSI ≤ 50, ADX ≥ min).

**Scoring:**

- Adds **MTF Weight TF1** / **TF2** (defaults 10 and 15, max 30 each in inputs) when that TF is aligned **with** that side.
- Subtracts **10** when MTF is **conflicting** (e.g. for a long idea, either HTF is **fully aligned short**).

**Table text:** full bull/bear HTF align, **partial** if only one TF aligns, else **MTF Neutral**.

---

## 6. Scoring engine (exact weights in code)

Scores start at 0. **Additions and penalties** below are per **longScore** / **shortScore** from the same logical event on each side (mirror for bearish). MTF weights use your input values.

| Category | Condition | Points (typical side) |
|----------|-----------|------------------------|
| Trend | GMMA bull / bear | **+16** |
| Trend | Supertrend bull / bear | **+8** |
| Trend | EMA200 bull / bear | **+8** |
| Structure | Dow-style bull / bear | **+6** |
| Momentum | RSI bull / bear | **+6** |
| Momentum | MACD bull / bear | **+6** |
| Momentum | ADX/DMI bull / bear | **+8** |
| Participation | High volume (`volBull`) | **+4** (both sides) |
| PA | BOS bull / bear | **+9** |
| PA | Retest bull / bear | **+8** |
| PA | Pullback long / short | **+7** |
| PA | Engulfing bull / bear | **+4** |
| PA | Pin bull / bear | **+3** |
| PA | Inside break bull / bear | **+3** |
| Context | Strong ribbon | **+5** |
| Context | Tight ribbon | **−6** |
| Context | Expansion | **+4** |
| Context | Compression | **−5** |
| Context | Overextended long / short | **−6** |
| Structure | CHoCH-style bull / bear hint | **+3** |
| MTF | Aligned TF1 / TF2 | **+weight** (inputs) |
| MTF | Conflict | **−10** |

Then each score is **clamped to 0–100**.

**Signal grade** (`Signal Grade` row) uses **max(longScore, shortScore)**:

- **A+** ≥ 90  
- **A** ≥ 82  
- **B** ≥ 74  
- **C** ≥ 64  
- else **D**

**Entry quality** (separate for long/short) uses score, HTF alignment, retest/BOS, overextension — labels include **Good**, **Early/Good**, **Late/Chase**, **Weak**.

---

## 7. When LONG / SHORT actually print

A signal is **not** “score > threshold” alone. **All** must be true:

1. **Entry pattern** for the selected **Trading Mode** (see below).
2. **Every enabled filter** passes (`useGMMAFilter`, `useSupertrendFilter`, …).
3. **Long/short score** ≥ **Min Long/Short Score** (default 75).
4. **Bar confirmation** if **Only Confirm On Candle Close** is on (`barstate.isconfirmed`).
5. **First bar** of the raw condition: `signalRaw and not signalRaw[1]` (no repeat every bar).

### 7.1 Trading modes (entry pattern)

| Mode | Long triggers (any) | Short triggers (any) |
|------|------------------------|----------------------|
| **Aggressive** | GMMA cross up, bull BOS, pullback long, inside break up | GMMA cross down, bear BOS, pullback short, inside break down |
| **Balanced** | Bull BOS, bull retest, pullback long | Bear BOS, bear retest, pullback short |
| **Conservative** | Bull retest, or (**bull** BOS + bull engulf), or (pullback long + bull pin) | Bear retest, or (**bear** BOS + bear engulf), or (pullback short + bear pin) |

### 7.2 Filters (when toggled on)

Examples:

- **BOS filter:** allows long if **bull BOS OR bull retest OR pullback long** (so you are not forced into only a raw BOS).
- **Retest filter:** if on, **requires** the retest condition for that side.
- **Candle filter:** long requires **bull engulf OR bull pin**; short requires **bear engulf OR bear pin**.
- **Inside bar filter:** requires inside-bar breakout in that direction.

Disable filters gradually if you want more signals; enable more for **stricter** discretionary trading.

---

## 8. Risk: entry, stop, target

On a signal bar (for display):

- **Entry** = **close** of the signal bar.
- **Stop** by mode:
  - **ATR:** entry ± ATR × multiplier.
  - **Swing:** lowest low / highest high over **Swing Lookback**.
  - **Slow band:** **EMA slow group first line (`emaS1`)** as stop reference.
- **Take profit:** entry ± (**risk** × **Risk Reward**), where risk = distance to stop (floored to `syminfo.mintick`).

Lines extend a fixed number of bars ahead for visualization; **manage real trades** in your platform with your own rules.

---

## 9. Score Table cheat sheet

| Row | Meaning |
|-----|--------|
| Long / Short Score | Raw 0–100 after clamp; compare to your min threshold. |
| Bias | Long vs short edge if one score leads by **> 10**. |
| Trend | Bull / Bear / Mixed from GMMA + ST + EMA200. |
| MTF Bias | Combined HTF story. |
| TF1 / TF2 | Bull / Bear / Neutral for each bias timeframe. |
| Structure | Dow-style + CHoCH-style hints. |
| Price Action | Highest-priority PA label for that bar. |
| Momentum | ADX + MACD + RSI combined text. |
| Condition | Expansion / compression / tight-choppy / normal. |
| Stretch | EMA200 distance vs ATR-style overextension. |
| RSI / MACD / ADX / Volume / ATR | Raw context for the engine. |
| Pivot Levels | Last pivot high/low used in logic. |
| Long/Short Entry Q | Timeliness / chase risk for that direction. |
| Signal Grade | Letter grade from max score. |

---

## 10. Suggested starting presets (from project docs)

Adjust to your symbol and session; these are **starting points**, not guarantees.

**Crypto scalping (chart 1m–3m)**  
Bias TF1 **5m**, TF2 **15m** · Mode **Balanced** · Min score **~75** · Stop **ATR** mult **~1.4–1.8** · BOS + candle filters on.

**XAUUSD scalping**  
Bias TF1 **15m**, TF2 **60m** · Mode **Conservative** · Min score **~80** · Stop **Swing** · BOS + **retest** + candle + **ATR floor** on.

**Forex intraday (5m–15m chart)**  
Mode **Balanced** or **Conservative** · Stop **Swing** · EMA200 + MTF on · **ATR floor** often useful.

---

## 11. Alerts

Create alerts from the indicator’s conditions:

- **Long Signal** / **Short Signal**
- **Bull BOS** / **Bear BOS**

---

## 12. What this project is not

- Not a guaranteed profitable system.  
- Not a single-indicator holy grail.  
- Not training ML inside Pine (future external ML would be a separate pipeline).  
- Not a substitute for your own risk, sizing, and discretion.

---

## 13. Repository layout

```
Indicator-tdv/
├── gmma_indicator.pine.txt   # Pine v6 source — paste into TradingView
└── README.md                 # This tutorial
```

---

## Disclaimer

This indicator is for **education and research** only. It is not financial advice. Past performance does not guarantee future results. Trade only with capital you can afford to lose and follow applicable laws and broker rules.
