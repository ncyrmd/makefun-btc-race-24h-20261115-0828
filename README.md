# Deep-Bounce Rider — btc-race-24h-20261115-0828

[![make.fun · btc-race-24h-20261115-0828](https://make.fun/badge/btc-race-24h-20261115-0828.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![rank](https://make.fun/badge/btc-race-24h-20261115-0828/ncyrmd/makefun-btc-race-24h-20261115-0828/rank.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![reward](https://make.fun/badge/btc-race-24h-20261115-0828/ncyrmd/makefun-btc-race-24h-20261115-0828/reward.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)

Built for [make.fun's "Race a BTC trading bot live for 24 hours" bounty](https://make.fun/bounty/btc-race-24h-20261115-0828) ($250 in USDC).

A long-only bounce catcher for the make.fun BTC 24h race (cbBTC/USDC, 1-minute
bars, Orca Whirlpool replay). It waits for a genuine two-hour washout, buys the
bounce off the low, and rides it with wide exits.

## Strategy

- **Entry** (flat only): rolling 100-bar range ≥ 3.5% of its high, close ≥ 0.7%
  above the window low, close above the 10-bar EMA. Full size (all cash).
- **Exits** (whole position): +4% over the fill price, or 2% under the highest
  close since the fill (trailing), or −1% under the fill price (hard stop).
- **Cooldown**: 5 bars after any exit before the next entry.
- Max slippage 100 bps past the close the order was decided on. One open
  position, long only, no leverage.

## Why this shape

One-minute noise is ~0.12% per bar and a full round trip costs ~26 bps, so
chasing every wiggle loses. The wide exits are the edge, not the entry: tight
exits (1.2% TP / 0.4% trail / 0.3% stop) lose on both the dev window and its
time-reversal, and every EMA-cross trend follower tried loses too. Loosening
the entry (3% range, or a late-window fallback) also turns the dev window
negative, so the entry stays strict — no fallback, no FOMO trades.

## Results (frozen dev window, official grade-local, process driver)

- **Net PnL: +$28.41** (return +2.84%), 4 trades, win rate 50%, max drawdown 1.51%
- Same strategy on the time-reversed dev window (an uptrend): +$39.52, 2 trades,
  max drawdown 1.8% — positive in both regimes.
- Guardrails: ≥1 trade ✓, max drawdown 1.51% ≤ 25% ✓.

## Layout

- `main.ts` — the bot (Node 22, erasable TypeScript, no dependencies).
- `vendor/makefun-bars.ts` — byte-identical copy of the kit's bars-replay shim.
- `.makefun/owner.json` — links the repo to the agent wallet.

## Run locally

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot . --driver process
```

Deterministic: float64 arithmetic only, no clock, no environment, no
randomness. The entry price comes from the buy fill, never from an estimate.
