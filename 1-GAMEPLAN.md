# Kraken Funded - Game Plan v1

Status: DRAFT v1. No challenge fee spent. The go/no-go gate at the bottom is explicit and unresolved.

## The challenge, verified (kraken.com/features/funded, 2026-09-05)

- Tiers: $1K / $5K / $10K starting capital. One-time non-refundable fee: $20 / $50 / $90.
- Pass condition: grow the balance +12% (e.g. $1,000 -> $1,120).
- Fail condition: balance drops more than 5% below the STARTING balance (static floor, $950 on the $1K tier). Not trailing.
- No daily loss limit. No time limit. No minimum trade count.
- Zero commissions in the challenge and funded account.
- Instruments: 50+ crypto assets (BTC, ETH, etc.). Crypto only - the "11k+ stocks" figure is Kraken's main app, not Funded.
- Funded payout: 80/20 split, USD withdrawals (~24h), no cap.
- Operator: Payward Oceanic Ltd (BVI) + Breakout Trading Group. Unregulated. Their own disclosure says funded trades may be B-booked (recorded internally, not routed to a market) at their discretion, and their financial interest is opposite to the trader's at the max-loss line. Standard prop-firm structure; size expectations accordingly.
- Not available in ME/MA/NY. (We're in IL - fine.)

## The shape of the problem

A static floor with no time limit turns the challenge into a barrier problem: random walk, +12% up-barrier, -5% down-barrier, unlimited steps. Three levers decide it:

1. Expectancy per trade (must be genuinely positive - this is the whole game).
2. Risk per trade (smaller steps = more steps before the floor = variance matters less).
3. Trade count (frequency determines how long the grind takes, not whether it works).

With zero commissions and 24/7 crypto markets, small-edge high-frequency configs are unusually viable here - no fee drag per trade, no session boundaries.

## Sizing math vs the floor

Framework (v2 fills in verified numbers from the wide search):

- Risk r% of current balance per trade. Stop distance in ATR units comes from the config; position size = (r x balance) / stop distance. Never size off conviction, only off the stop.
- Example at r = 1% on the $1K tier: $10 risk per trade. Target = +12R, floor = -5R.
- With a verified config of win rate p and average R per trade E: expected path to target is 12/E trades. At E = +0.15R that is ~80 trades; at 90 signals/symbol/quarter across even 5 symbols, that is days-to-weeks, not months.
- Kill-switch: hard stop at -4% (one full R of margin above the -5% floor). A challenge that reaches the kill-switch is abandoned by the bot, not by the platform.
- Drawdown math to run in v2: Monte Carlo on the verified signal distribution (win rate, win/loss R sizes, streaks) to get P(hit +12% before -5%) at r = 0.5%, 0.75%, 1.0%. Pick the smallest r with acceptable time-to-target.

## What the engine needs (the editing answer)

The BB engine is asset-agnostic - OHLCV in, signals out. What it lacks is everything a challenge is scored on:

1. Data adapter: Kraken public OHLC API (24/7 crypto bars, no sessions). Small.
2. Risk wrapper (new module): risk-% sizing from stop distance, running equity curve, exposure cap, kill-switch at -4%. Moderate.
3. Challenge simulator: replay any verified config through the wrapper on historical crypto data, output P(pass) distribution, expected trades-to-target, worst drawdown path. Moderate.
4. Execution layer (later, separate decision): Kraken order API, sandbox first. Out of scope for v1.

One engine serves both the intraday research and the challenge: the signal layer stays shared, the risk wrapper is challenge-specific.

## Go / no-go gate (explicit)

No $20 fee gets spent until ALL of these hold:

1. A config from the 10-year wide search shows positive expectancy (avg R > 0) in BOTH the train window (2016-2021) and the test window (2022-2026), n >= 100 in each, on crypto-representative volatility - not just equities.
2. The same config re-verifies on Kraken crypto OHLC data (the pattern must exist on the instruments we can actually trade).
3. Monte Carlo on the verified distribution gives P(pass) comfortably above the fee-adjusted breakeven (at $20 fee vs $1,120 target and 80% split, breakeven P(pass) is low, but we want margin for model error - target P(pass) >= 70%).
4. Sandbox execution runs clean for a shakedown period (fills, slippage, API behavior) before any live challenge.

Current state of the gate: intraday equity configs are all negative expectancy as of 2026-09-05 (60d Yahoo data, full 221-name universe). The 10-year Alpaca wide search is the first real chance to clear gate 1.
