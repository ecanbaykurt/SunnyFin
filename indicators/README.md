# Python Trading Indicators – Notebook Playground

This repository is a small, self‑contained playground for implementing and testing several popular TradingView‑style indicators in Python on a bundle of large‑cap tech stocks.[conversation_history:1]

The focus is on:

- Re‑implementing well‑known Pine Script indicators in Python using pandas.[conversation_history:1]
- Cleaning and reshaping multi‑ticker OHLCV data into an analysis‑friendly long format.[conversation_history:1]
- Applying the indicators to real stock data (e.g., Tesla) and visualizing them with Matplotlib.[conversation_history:1]

Interactive Plotly charts were intentionally removed from the notebook to keep the repo light and emphasize the core indicator logic.[conversation_history:1]

---

## Repository structure

- **`indicators.ipynb`**  
  Main Jupyter Notebook that:

  - Loads and cleans `tech_stocks.csv`.[conversation_history:1]
  - Reshapes the original “wide” table (`Close`, `Close.1`, …) into a long format with columns: `Date`, `Company`, `Open`, `High`, `Low`, `Close`, `Volume`.[conversation_history:1]
  - Maps tickers (AAPL, AMZN, GOOG, META, MSFT, NVDA, TSLA) to their full company names to make plots more readable.[conversation_history:1]
  - Implements and tests several indicators on a chosen stock (e.g., Tesla), including plotting with Matplotlib.[conversation_history:1]

- **`tech_stocks.csv`**  
  Sample OHLCV data for several large‑cap tech companies from 2023 onwards, originally exported in a wide layout with per‑ticker column suffixes (e.g., `Close.1`, `Close.2`, …).[conversation_history:1]  
  The notebook shows how to reshape this into a tidy, long dataset.

---

## Implemented indicators

All indicator functions operate on a **single‑instrument** pandas `DataFrame` with:

- Index: `Date` (datetime)
- Columns: `open`, `high`, `low`, `close`, `volume` (lower‑case)[conversation_history:1]

These frames are obtained by slicing the long multi‑company dataset.

### 1. Squeeze Momentum Indicator (LazyBear variant)

- Reproduces the well‑known Squeeze Momentum Indicator using:

  - Bollinger Bands vs. Keltner Channels to detect volatility “squeezes”.  
  - A linear regression of a transformed price series to build a momentum histogram.[conversation_history:1]

- Outputs include:

  - `val`: momentum value (histogram).
  - `sqz_on`, `sqz_off`, `no_sqz`: boolean squeeze state flags.[conversation_history:1]

This mirrors the popular LazyBear implementation used on TradingView.[web:22][web:21]

### 2. WaveTrend Oscillator (LazyBear)

- Implements the WaveTrend Oscillator based on:

  - Average price \( \text{HLC3} \).
  - Double‑smoothed channel index to produce two lines, WT1 and WT2.[web:71][web:68]

- Outputs:

  - `wt1`: main oscillator line.
  - `wt2`: signal line (SMA of `wt1`).
  - `wt_diff`: `wt1 - wt2` (useful for area/histogram style plots).[conversation_history:1]

### 3. Volume Flow Indicator (VFI, LazyBear port)

- Python port of LazyBear’s Volume Flow Indicator, itself based on Markos Katsanos’ VFI.[web:103][web:107]

- Core steps:

  - Compute typical price and log returns.
  - Use volatility‑scaled cutoff to classify positive/negative money flow.
  - Cap extreme volumes and accumulate signed volume over a lookback.
  - Smooth with an EMA to obtain a signal line.[conversation_history:1]

- Outputs:

  - `vfi`: main Volume Flow Indicator.
  - `vfima`: EMA of VFI (signal line).
  - `d`: `vfi - vfima` (for optional histogram).[conversation_history:1]

### 4. Ultimate MACD MTF (CM_Ult_MacD_MTF‑style core)

- Flexible MACD implementation inspired by the “Ultimate MACD MTF” indicator:[web:84][web:83]

  - Configurable fast/slow lengths and signal length.
  - Choice of SMA or EMA for both MACD base and signal line.
  - Optional higher‑timeframe close series to emulate multi‑timeframe MACD.[conversation_history:1]

- Outputs:

  - `macd`, `signal`, `hist`.
  - `trend_up`, `trend_dn`: MACD vs. signal trend flags.
  - `cross_up`, `cross_dn`: MACD crossing the signal.
  - `cross_up_strict`, `cross_dn_strict`: crosses where MACD is above/below zero at the event.[conversation_history:1]

These states are suitable for coloring histograms, highlighting trend phases, and driving alerts.[web:92][web:90]

### 5. EMA‑ATR Buy/Sell Signal System

- Simple but complete position‑management helper built around:

  - Trend detection via fast and slow EMAs on the close.
  - ATR‑based stop‑loss (SL) and multi‑R take‑profit (TP) levels.
  - Optional “candle confirmation” for entries (e.g., bullish candle on long entry).[conversation_history:1]

- For each bar, the function tracks:

  - `buy_signal`, `sell_signal`: entry triggers.
  - `in_position`, `position_type`: whether a position is open and if it is `LONG` or `SHORT`.
  - `entry_price`, `stop_loss`, `take_profit_final`.
  - `tp_final_hit`, `sl_hit`: exit events.
  - `invalidated`, `invalidated_type`: when an opposite signal kills the prior trade.
  - Intermediate TP levels and per‑level hit flags.[conversation_history:1]

This structure can be plugged into backtests or a charting front end to draw risk lines and labels.

---

## Data preparation

The `tech_stocks.csv` file starts in a **wide** format with:

- Header rows storing ticker names and labels.
- Per‑ticker columns like `Close`, `Close.1`, …, `Volume.6`.[conversation_history:1]

The notebook converts this into a **long** format in three main steps.

### 1. Clean header rows and dates

- Use the first rows to recover ticker symbols (AAPL, AMZN, GOOG, META, MSFT, NVDA, TSLA).  
- Rename the `Price` column to `Date` and parse it to `datetime`.[conversation_history:1]

### 2. Reshape to “Date–Ticker–OHLCV”

- For each suffix group (`''`, `.1`, …. `.6`) corresponding to a specific ticker:

  - Collect `Open`, `High`, `Low`, `Close`, `Volume` columns.
  - Attach the correct ticker name.
  - Concatenate all groups into one long table.[conversation_history:1]

The result:

Date | Ticker | Open | High | Low | Close | Volume


This long structure is standard for multi‑asset analysis and works well with pandas groupby operations.[web:158][web:165]

### 3. Map ticker → company name

- Map tickers to human‑readable names, for example:

  - `AAPL` → `Apple`
  - `AMZN` → `Amazon`
  - `GOOG` → `Alphabet`
  - `META` → `Meta`
  - `MSFT` → `Microsoft`
  - `NVDA` → `NVIDIA`
  - `TSLA` → `Tesla`[conversation_history:1]

- Replace `Ticker` with `Company` so the final long table is:

Date | Company | Open | High | Low | Close | Volume


This makes plots and downstream analysis easier to interpret.

---

## Usage

### 1. Install dependencies

From a fresh Python environment:

pip install pandas numpy matplotlib


Plotly is not required in the current version of the notebook; all visualizations use Matplotlib.[conversation_history:1]

### 2. Open the notebook

Launch Jupyter (or VS Code) and open:

jupyter notebook indicators.ipynb


Execute the cells in order:

1. **Load and reshape data**  
   - Produces the long `df_long` DataFrame and company‑specific slices such as `df_tesla`.[conversation_history:1]

2. **Prepare OHLCV for a single company**  
   Example for Tesla:

df_tsla = (
df_long[df_long["Company"] == "Tesla"]
.sort_values("Date")
.set_index("Date")
)

ohlcv_tsla = df_tsla[["Open", "High", "Low", "Close", "Volume"]].rename(columns=str.lower)


3. **Compute indicators**

wt_tsla = wavetrend_lazybear(ohlcv_tsla)
vfi_tsla = vfi_lazybear(ohlcv_tsla)
sqz_tsla = squeeze_momentum_indicator(ohlcv_tsla)
macd_tsla = macd_ultimate_mtf(ohlcv_tsla)
bs_tsla = buy_sell_signal_system(ohlcv_tsla)


4. **Visualize (Matplotlib)**  
The notebook creates multi‑subplot figures with:

- TSLA price and Buy/Sell markers in the top pane.
- WaveTrend, VFI, Squeeze Momentum, and MACD in separate panels below.[conversation_history:1]

---

## Extending the project

- **Interactive charts**  
Although the current notebook stops at Matplotlib plots, the indicator outputs are designed to be dropped into an interactive front end (e.g., Plotly, or a custom TradingView‑like UI).[web:177][web:187]

- **More indicators**  
Additional indicators can be added by following the same pattern:

- A pure function that takes a single‑instrument OHLCV DataFrame.
- A short “example usage” section showing how to call it and what outputs mean.[web:62][web:188]

- **Backtesting and scanning**  
The Squeeze Momentum, MACD, VFI, and Buy/Sell system all expose boolean signal columns, which can drive scanners or be wired into a simple backtesting engine.[web:39][web:36]

This repository is intended as a clear, educational starting point for building a reusable technical‑indicator library in Python on top of real multi‑company stock data.[conversation_history:1]
