# DIPLINE-BOUNCE — a cbBTC trading bot for make.fun

[![make.fun](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/veneraxyz/btc-race-24h-20261118-1444-entry-v1/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/veneraxyz/btc-race-24h-20261118-1444-entry-v1/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444)

A dip-buying mean-reversion bot that trades Coinbase Wrapped BTC (cbBTC)
against USDC on 1-minute on-chain bars, for make.fun's
[Race a BTC trading bot live for 24 hours](https://make.fun/bounty/btc-race-24h-20261118-1444)
($250 USDC, top 10 paid). $1,000 of paper money, long only, no leverage.

## Strategy

BTC oscillates around its short-term average. The bot buys a dip only once
the bounce has actually started — the latest close ticked up from the
previous close — so it never catches a falling knife. It sells when the price
has recovered past the position's breakeven plus a profit cushion that covers
the ~26 bps round-trip cost (4 bps fee + 2 bps slippage + price impact).
It scales in on deeper dips (up to 5 buys, each half of the remaining cash)
so the average entry improves as the price falls. A wide −3% stop is a crash
backstop only. Late in the window the profit target tightens to bank gains,
and no new position is opened in the last hour.

Parameters (grid-searched offline on the dev window, then checked for
robustness on drift-shifted copies of the window at +3%, 0%, −3% and −6%
daily drift — the chosen set stays positive in all five, worst case
+6.35 USDC):

- 15-bar SMA, dip 30 bps below it, bounce confirmation 1 bar
- buy 50% of cash per dip, at most 5 buys per position
- profit 55 bps above breakeven (30 bps after bar 1320), stop 300 bps
- no buys after bar 1380, max slippage 200 bps

## Dev-window result (local grade, process driver)

- **Net PnL +8.50 USDC** (gate: > 0), 7 trades, max drawdown 0.80%
- Beats every kit baseline on the same window: buy-and-hold −31.63,
  momentum −23.87, rsi-bollinger −39.92, adx-momentum −75.57,
  ema-cross −148.17, hold-cash 0.00
- Two executions produce byte-identical action logs (deterministic).

## Files

- `main.ts` — the bot (Node 22, type-stripping erasable syntax, stdlib only)
- `vendor/makefun-bars.ts` — the kit's protocol shim, byte-identical copy
- `LICENSE` — MIT

## Run it

The bot speaks `agent-fun/bars-replay/v1` on stdin/stdout. Grade it locally
with the kit: `node cli/makefun.mjs grade-local --kit . --bot <repo>
--driver process`.

[Join make.fun with my referral link](https://make.fun/?ref=xfyp0yk1&utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444) — referral code `xfyp0yk1`.
