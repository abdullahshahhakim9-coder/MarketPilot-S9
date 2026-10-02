# MarketPilot — Samsung Galaxy S9+ edition (v2)

Android decision-support app for forex, gold and silver: **EUR/USD, GBP/USD, USD/JPY, XAU/USD, XAG/USD** on **M15, H1, H4, D1**.
It never places trades and a signal is a *confluence count*, not a probability or a promise.

## What it does
- **Live data** from Twelve Data (1,000 candles per request, UTC timestamps) or clearly-labelled **simulated data** when no API key is set.
- **Candlestick chart** with EMA 20/50/200, Bollinger(20,2), entry / stop / TP1 / TP2 lines.
- **Confluence engine** (score −8…+8, a signal needs ±5 *and* passes safety gates):

  | Component | Range | Rule |
  |---|---|---|
  | Trend | −3…+3 | price vs EMA20, EMA20 vs EMA50, EMA50 vs EMA200 |
  | Momentum | −2…+2 | RSI(14) above 55 / below 45, MACD(12,26,9) histogram sign |
  | Volatility | −1…+1 | Bollinger stretch (ignored when ADX ≥ 25, i.e. a strong trend) |
  | Structure | −2…+2 | higher-highs/higher-lows vs lower-highs/lower-lows, break of last swing |

  Gates: trend must agree (|trend| ≥ 2), ADX ≥ 18 (not ranging), RSI not extreme (≥ 80 / ≤ 20).
  When a gate blocks a signal the app says *which one and why*.
- **Trade plan**: stop is placed beyond the nearest swing (clamped to 1–3 × ATR), TP1 = 1.5R, TP2 = 2.5R. WAIT shows key support/resistance and trigger levels instead of fake trade levels.
- **Position sizing** from your balance and risk % (USD account, standard contracts — your broker may differ).
- **Rule check on loaded history**: replays the same rules over the loaded candles (TP1 vs stop, pessimistic when both are touched, no lookahead, no spread/slippage). It is a sanity check, not a forecast.
- **Closed-candle mode** (default) so signals don't repaint while a candle is forming.
- Indicators use standard charting conventions (Wilder RSI/ATR/ADX, SMA-seeded EMA) and are verified against an independent reference implementation.

## Data & privacy
- Create a free key at twelvedata.com and enter it under **API KEY**. It is stored **AES-256-GCM encrypted in the Android Keystore**, excluded from backups, and sent only to `api.twelvedata.com` over HTTPS.
- Free plans allow 8 requests/min and 800/day. The app caches results for 45 s, skips requests for a symbol you've already moved away from, and pauses for a minute after a rate-limit reply.
- On any error the app keeps the last good data (marked STALE) or shows NO DATA. **It never substitutes simulated data for a failed live request.**

## Building the APK
**GitHub Actions** — push to GitHub, run *Build MarketPilot APK*; it runs the unit tests and uploads `MarketPilot-debug-apk`.

**AndroidIDE (Galaxy S9+)** — open the folder, let Gradle sync, then in the terminal:
```
./gradlew assembleDebug
```
The first run downloads Gradle 8.9 (~130 MB, requires `curl` or `wget`, and `unzip`). The APK is at `app/build/outputs/apk/debug/app-debug.apk`.

**Unit tests** — `./gradlew testDebugUnitTest` (indicators, engine properties, API parsing, sizing).

## Project layout
`Indicators`, `Series`, `SignalEngine`, `Describer`, `Pipeline`, `RiskCalculator`, `DemoData`, `MarketDataClient` are plain Java with no Android imports (unit-tested on the JVM).
`MainActivity`, `ChartView`, `SecureStore` are the Android layer.

## Known limits
- Signals use one timeframe at a time (no multi-timeframe confirmation) and no news/session filter.
- Simulated data is random and exists only to try the interface.
