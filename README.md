# Minty Triangle

A MetaTrader 5 Expert Advisor that looks for **triangular arbitrage** — short moments when the
exchange rates between three currencies disagree with each other — and trades the gap.

> ⚠️ **Many brokers prohibit arbitrage, retail latency usually erases the edge, and backtests
> can't simulate the thing that matters.** Run [Shadow Mode](#shadow-mode) before risking money.
> Full explanation in [details.md](details.md).

## Features

- **Automatic triangle discovery** — give it a currency list (e.g. `XAU,USD,EUR`) and it finds
  your broker's symbols (including suffixes like `.raw` or `m`) and builds every unique triangle
  in both directions.
- **Realistic cost model** — commission (open + close) plus slippage measured from your own fills
  at a configurable percentile.
- **Stale-quote gate** — rejects signals built on prices older than `MaxQuoteAgeMs`.
- **Revalidate before firing** — recalculates on fresh ticks and cancels if the opportunity is gone.
- **Worst-leg-first execution** with leg timeouts and exit on a missing leg.
- **Decaying profit target** — closes at break-even after `BreakEvenAge`.
- **Filters** — high-impact news, volatility spikes, market open/close windows, cooldown after bad exits.
- **Shadow Mode** — measures opportunities after a simulated latency without ever placing a trade.

## Files

| File | Description |
|---|---|
| `MintyTriangle.mq5` | Source code |
| `MintyTriangle.ex5` | Compiled Expert Advisor |
| `details.md` | In-depth explanation of the strategy, risks and settings |

## Installation

1. Copy `MintyTriangle.ex5` (or `.mq5`) to `MQL5\Experts\` in your MT5 data folder
   (*File → Open Data Folder*).
2. If using the source, open it in MetaEditor and compile (F7).
3. Restart MT5 or refresh the Navigator, then drag the EA onto any chart.
4. Enable **Algo Trading**. For the news filter, make sure the economic calendar is available.

## Shadow Mode

Set `ShadowMode = true`. The EA runs normally but never trades. Every detected opportunity is
re-evaluated after `ShadowLatencyMs` and logged to `MQL5\Files\_MintyTriangleShadowLog.csv`
with `sig_net`, `real_net`, `edge_decay` and `survived`.

If `edge_decay` is consistently larger than `sig_net`, the strategy won't work on your setup.
Otherwise, use the data to set `ProfitThreshold`. Run it for at least a week.

## Key settings

| Setting | Default | Purpose |
|---|---|---|
| `CurrencySet` | `XAU,USD,EUR` | Currencies to build triangles from |
| `RiskFactor` | 10 | % of free margin to use |
| `ProfitThreshold` | 100 | Required profit as % of costs |
| `ProfitTrgger` | 50 | % of theoretical profit to take |
| `BreakEvenAge` | M5 | When to start closing at break-even |
| `EnableStoploss` | false | Per-leg stop-loss |
| `EnableVolatility` | false | Skip volatility spikes |
| `EnableNewsFilter` | false | Avoid high-impact news |
| `SymbolSuffix` | *(empty)* | Broker symbol suffix |
| `MaxQuoteAgeMs` | 250 | Reject stale prices (0 = off) |
| `SlippagePercentile` | 90 | Pessimism of the slippage estimate |
| `ShadowMode` | false | Measure only, never trade |
| `ShadowLatencyMs` | 150 | Simulated execution latency |
| `MagicNumber` | 8172934 | Trade ownership tag |

See [details.md](details.md#settings) for the complete list.

## Suggested approach

1. Confirm in writing that your broker permits arbitrage.
2. Backtest only to check it runs and builds the expected triangles.
3. Run Shadow Mode live for a week or more and check `edge_decay`.
4. Set `ProfitThreshold` from Shadow data, trade minimum size, and scale slowly.

## License

Copyright © 2025 Christopher Benjamin Hemmens. Distributed under the BSD-style license in the
header of `MintyTriangle.mq5`. Provided without warranty. This is not financial advice, nor a
claim that the strategy is profitable or permitted on your account.
