# Cup and Handle Pattern Detector v2.0 ☕

A production-quality Python script for detecting "Cup and Handle" chart patterns in the Indian Stock Market (NSE). Uses `yfinance` for data, `scipy` for peak detection, price smoothing for noise reduction, volume confirmation for conviction, and a composite quality score for ranking — all with strict geometric validation to eliminate false positives.

## What is a Cup and Handle Pattern?

Imagine the cross-section of a coffee cup:

```
Left Rim ────╮                  ╭──── Right Rim
              ╲                ╱
               ╲              ╱         ╮── Handle
                ╲            ╱          │   (small dip)
                 ╰──────────╯           ╯
                   Cup Bottom
```

1. Price rises to a **high** (Left Rim)
2. Price **drops** 10–35% (the "cup")
3. Price **recovers** back near the original high (Right Rim, within 3%)
4. Price dips **slightly** again (the "handle" — must stay within 32% of cup depth)
5. A **breakout** above the rim signals a potential upward move

## Features

### Three Modes
- **🧪 Self-Test:** Synthetic data tests (instant, no internet needed) to prove the detection math works
- **📊 Historical Backtest:** Sweeps past data (up to 2 years) for already-completed patterns
- **🚀 Live Scanner:** Continuous scanning during NSE market hours (Mon-Fri, 9:15 AM – 3:30 PM IST)

### New in v2.0
- **Price Smoothing:** SMA(5) applied to `find_peaks()` for noise-free peak/trough detection — all reported prices remain raw
- **Volume Confirmation:** Three checks (Cup Decline, Recovery, Breakout) validate institutional interest
- **Quality Score:** Composite ranking so you can prioritize the strongest patterns

### Strict Validation Rules (5 Bug Fixes Built-In)
| Rule | What it prevents |
|---|---|
| No Double-Dip | Price can't drop below cup bottom during recovery |
| Bottom Roundedness | Rejects V-shaped bottoms (requires ≥20% of candles in base zone) |
| Corrected Handle Low | Tracks true low until breakout confirms (not first dip) |
| Pause Before Breakout | Handle must be genuine consolidation (≥5 candles, negative slope) |
| Structural Ceiling | No internal price can exceed the Left Rim (rejects broken structure) |
| Discontinuity Filter | Rejects jagged cups with >8% single-day price gaps |
| Right Rim Stability | Filters out single-candle spike false positives |
| Handle Geometry | Handle pullback ≤ 32% of cup depth |

## Requirements

- Python 3.7+
- Dependencies:

```bash
pip install yfinance pandas numpy scipy
```

Or use the requirements file:
```bash
pip install -r requirements.txt
```

## Usage

### Interactive Menu
```bash
python cup_and_handle_detector.py
```
Choose from Self-Test, Historical Backtest, Live Scanner, or Single Stock scan. It will prompt for candle interval and lookback period.

### Command Line
```bash
# Self-test (instant, no internet)
python cup_and_handle_detector.py test

# Historical backtest (daily candles, 2 years)
python cup_and_handle_detector.py historical --interval 1d --lookback 2y

# Backtest specific stocks
python cup_and_handle_detector.py historical RELIANCE.NS TCS.NS --interval 1d --lookback 2y

# Custom date range
python cup_and_handle_detector.py historical --start-date 2024-01-01 --end-date 2024-06-01

# Live scanner (15-min candles)
python cup_and_handle_detector.py live --interval 15m --lookback 59d
```

### Supported Intervals
| Interval | Max Lookback | Use Case |
|---|---|---|
| `15m`, `30m`, `1h` | ~59 days | Intraday/swing patterns |
| `1d` | Multi-year | Standard daily chart patterns |
| `1wk` | Multi-year | Long-term macro patterns |

> **Note:** yfinance limits intraday data to ~60 days. The script automatically caps lookback if you choose an incompatible combo.

## How to Interpret the Output

### Quality Score
Higher is better (typical range: 5–80+). The formula:
```
Score = (Cup Drop % × 0.4) + log(Recovery Vol Ratio + 1) × 30 + log(Breakout Vol Ratio + 1) × 30
```
- **Cup Drop %**: Deeper cups = stronger patterns (a 20% drop scores more than 10%)
- **Recovery Volume Ratio**: If buying volume during recovery exceeds selling volume, it adds confidence
- **Breakout Volume**: Higher breakout volume = more institutional interest

### Volume Checks (Soft — don't reject patterns)
- **Cup Decline ✅**: Average volume during the decline < 50-period SMA = "light selling" (good)
- **Recovery ✅**: Up-day volume / Down-day volume > 1.0 = "more buying" (good)
- **Breakout ✅**: Breakout volume > 1.2× 50-period SMA = "strong interest" (good)
- **⚠ WARN**: Doesn't reject the pattern but lowers confidence

### Debug Output
Every candidate (pass or fail) shows the full diagnostic:
- Geometry, Roundedness, Pause-Before-Breakout checks
- Handle Low comparison (old first-dip vs new corrected)
- All 3 volume checks with exact numbers
- Smoothing method used
- Final Verdict with score or rejection reason

## Output Files
- **Terminal:** Clean summary table + full debug output
- **`backtest_results.txt`:** Detailed breakdown saved automatically after historical backtests

## Migrating to a Real-Time Broker API (Fyers)

If you later open a trading account and want real-time data instead of Yahoo Finance's delayed feed:

1. **Sign up** for a Fyers developer account at [myapi.fyers.in](https://myapi.fyers.in/)
2. **Replace `fetch_batch_data()`** with Fyers API calls using the `fyers-apiv3` Python package
3. **Symbol format**: Change `.NS` suffix to Fyers format (e.g., `NSE:RELIANCE-EQ`)
4. **Benefits**: True real-time data, faster updates, no Yahoo throttling, order placement capability
5. The detection logic (`detect_cup_and_handle()`) stays **100% the same** — only the data source changes

```bash
pip install fyers-apiv3
```

## Project Structure
```
cup_and_handle_detector.py   # Main script (all-in-one)
requirements.txt             # Python dependencies
README.md                    # This file
backtest_results.txt         # Generated after historical backtests
```

## License
MIT
