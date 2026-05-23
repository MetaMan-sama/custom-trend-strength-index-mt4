# Custom Trend Strength Index Alert — MQL4 Script

A MetaTrader 4 script that computes a **proprietary Trend Strength Index (TSI)** by accumulating signed momentum components over a rolling lookback window and expressing the ratio of directional trend energy to total absolute momentum as a bounded percentage score, firing independent strong-uptrend and strong-downtrend alerts when the composite value crosses configurable positive or negative threshold levels.

---

## Overview

This script implements a custom momentum-normalized trend strength indicator that goes beyond simple directional bias by simultaneously measuring the magnitude and consistency of bar-to-bar price movement. For each bar in the lookback window, `CalculateTrendStrength()` computes the raw `momentum = close − prevClose` and accumulates it into `sumMomentum` (total signed momentum), while also accumulating a `trendComponent` — the signed unit direction multiplied by the absolute momentum value — into `sumTrend`. The ratio `(sumTrend / sumMomentum) × 100` produces a percentage that approaches ±100 when all bars are moving consistently in the same direction, and approaches zero in a choppy, directionless market. This construction captures both the directional consistency and the momentum weight of each bar in a single bounded scalar output.

---

## Features

- **Momentum-normalized TSI computation** — `sumTrend / sumMomentum × 100` combines signed directional magnitude with total momentum energy over `TrendPeriod` bars
- **Per-bar trendComponent construction** — `(momentum > 0 ? 1 : momentum < 0 ? -1 : 0) × |momentum|` weights each bar's directional vote by its absolute price change magnitude, giving larger moves more influence on the index
- **Sufficient-data guard** — `iBars() < period` check with `return 0.0` prevents false alerts during terminal warm-up or on instruments with sparse history
- **Independent bidirectional thresholds** — `UptrendThreshold` (default `+70.0`) and `DowntrendThreshold` (default `−70.0`) evaluated independently each cycle, allowing asymmetric extreme zone configuration for instruments with directional skew
- **Three notification channels:** sound alert, email, and mobile push via `Alert()`, `SendMail()`, and `SendNotification()`
- **Lightweight loop** — polls once per minute (`Sleep(60000)`) to minimize terminal CPU overhead
- Logs TSI value, alert type, symbol, and timeframe to the MT4 **Experts** tab on every trigger

---

## How It Works

1. Every minute, `CalculateTrendStrength()` validates `iBars(symbol, timeframe) >= period`, then iterates `i = 1` to `period`:
   - `momentum = iClose(..., i) − iClose(..., i+1)` — close-to-close change per bar
   - `trendComponent = sign(momentum) × |momentum|` — directionally weighted magnitude
   - Both accumulated into `sumMomentum` and `sumTrend` respectively
2. `trendStrength = (sumTrend / sumMomentum) × 100.0` computed and returned; returns `0.0` if `sumMomentum == 0`
3. Two independent threshold comparisons evaluate the result:
   - `trendStrength > UptrendThreshold` → **Strong Uptrend Detected**
   - `trendStrength < DowntrendThreshold` → **Strong Downtrend Detected**
4. `AlertTrend()` formats and dispatches the message via all enabled channels including `EnumToString(timeframe)` for readable timeframe output

---

## Input Parameters

| Parameter             | Type            | Default      | Description                                                            |
|-----------------------|-----------------|--------------|------------------------------------------------------------------------|
| `TradeSymbol`         | string          | `EURUSD`     | Symbol for analysis                                                    |
| `Timeframe`           | ENUM_TIMEFRAMES | `PERIOD_H1`  | Timeframe for analysis                                                 |
| `TrendPeriod`         | int             | `14`         | Lookback bars for TSI momentum accumulation                            |
| `UptrendThreshold`    | double          | `70.0`       | TSI value above which a strong uptrend alert is triggered              |
| `DowntrendThreshold`  | double          | `-70.0`      | TSI value below which a strong downtrend alert is triggered            |
| `EnableAlerts`        | bool            | `true`       | Fire an on-screen/sound alert                                          |
| `EnableEmail`         | bool            | `false`      | Send an email notification                                             |
| `EnablePush`          | bool            | `false`      | Send a mobile push notification                                        |

---

## Alert Message Format

```
Strong Uptrend Detected detected on EURUSD (Timeframe: PERIOD_H1)
Trend Strength: 78.34
```

---

## Installation

1. Copy `Custom_Trend_Strength_Index_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
