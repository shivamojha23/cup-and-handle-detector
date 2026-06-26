# Cup and Handle Pattern Detector ☕

A robust Python script for detecting "Cup and Handle" patterns in the Indian Stock Market (NSE). It fetches data using `yfinance` and validates patterns using strict geometrical and temporal rules to eliminate false positives.

## Features

- **Live Scanner Mode:** Scans the NSE watchlist during market hours and alerts you to patterns that are completing *today*.
- **Historical Backtest Mode:** Sweeps through the entire timeline (e.g., 2 years of daily data) to find every valid historical pattern.
- **Strict Validation Rules:**
  - **No Double-Dip:** Ensures the price doesn't drop below the cup bottom before reaching the right rim.
  - **Duration Checks:** Requires minimum candles for both the cup sides and the handle (prevents V-shape and 1-candle noise false positives).
  - **Right Rim Stability:** Filters out sharp, 1-candle wick spikes using a 5-candle moving average.
  - **Geometry Check:** Strictly enforces the rule that the handle pullback cannot exceed 32% of the total cup depth.
- **Dynamic Configuration:** Easily configure the candle interval (e.g., `15m`, `1d`) and lookback period (e.g., `59d`, `2y`) via command line arguments or the interactive menu. Built-in `yfinance` limitation checking is included.
- **Self-Test Suite:** Run synthetic data tests without the internet to prove the geometrical math works flawlessly.

## Requirements

- Python 3.7+
- Requirements listed in `requirements.txt`:
  - `pandas`
  - `numpy`
  - `yfinance`
  - `scipy`

Install dependencies with:
```bash
pip install -r requirements.txt
```

## Usage

You can run the script via the **Interactive Menu** by running it without any arguments:
```bash
python cup_and_handle_detector.py
```
From there, you can choose Live Scanner, Historical Backtest, Single Stock Backtest, or Self-Test. It will interactively prompt you for intervals and lookback periods.

### Command Line Mode
You can bypass the menu by using command-line arguments:

```bash
# 1. Live Scan (e.g., 15-minute candles over the last 59 days)
python cup_and_handle_detector.py live --interval 15m --lookback 59d

# 2. Historical Backtest (e.g., daily candles over the last 2 years)
python cup_and_handle_detector.py historical --interval 1d --lookback 2y

# 3. Backtest specific stocks using custom dates
python cup_and_handle_detector.py historical RELIANCE.NS TCS.NS --start-date 2024-01-01 --end-date 2024-06-01

# 4. Run built-in self-test
python cup_and_handle_detector.py test
```

## How It Works
The script utilizes `scipy.signal.find_peaks` to identify the structural components of the pattern (Left Rim, Cup Bottom, Right Rim, Handle Low). It then applies a series of geometrical thresholds and temporal duration checks before validating and alerting the pattern.

## Output
- **Live Mode:** Prints alerts to the terminal when a pattern completes on the current day. Deduplicates alerts so you only get notified once per stock per day.
- **Historical Mode:** Outputs an aggregated summary to the terminal and writes a detailed breakdown of all detected patterns, geometry, and dates to `backtest_results.txt`.
