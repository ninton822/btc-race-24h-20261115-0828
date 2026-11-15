# BTC Race Bot — capitulation dip buyer

[![make.fun · btc-race-24h-20261115-0828](https://make.fun/badge/btc-race-24h-20261115-0828.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![rank](https://make.fun/badge/btc-race-24h-20261115-0828/ninton822/btc-race-24h-20261115-0828/rank.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)
[![reward](https://make.fun/badge/btc-race-24h-20261115-0828/ninton822/btc-race-24h-20261115-0828/reward.svg)](https://make.fun/bounty/btc-race-24h-20261115-0828?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261115-0828)

Built for the make.fun bounty: https://make.fun/bounty/btc-race-24h-20261115-0828

A Python 3.13 (stdlib only) bot that trades Coinbase Wrapped BTC against
USDC on 1-minute on-chain bars, with $1,000 of paper money. It waits flat in
cash, buys capitulation dips in legs, banks a slice on the first rebound,
then rides the rest with a trailing stop.

## Strategy

- **Leg A (40%)** — price 2.2% below its 60-minute peak and 1.2% below the
  close 60 bars ago: a plain dip.
- **Leg B (30%)** — 2.9% below the 60-minute peak, or 1.9% below the close
  60 bars ago: a deep dip.
- **Leg C (30%)** — 3.0% below the close 90 bars ago, 3.0% below the
  120-minute peak, or 2.7% below the close 60 bars ago: capitulation.
- **Leg D (20%)** — a sharp 1.2% plunge inside 30 minutes, whatever the
  longer lookbacks say.

Legs fire together on one bar, sized to reach ~100% of equity by the time
the dip is a capitulation. Two more legs were tried on the dev window and
dropped: a shallow-wiggle leg and a recovery-pullback leg both added losing
trades in grinding downtrends.

## Exits

- **Bank** 20% of the position at +1.2% over the blended entry (fees in).
- **Trailing stop** once +1.2% is seen: exit if it gives back 0.7% from the
  peak, 1.2% once the trade is up 3% or more — so a V-shaped rebound can
  pay several percent instead of being cut at the first take-profit.
- **Hard stop** at −1.8% below the blended entry.
- **Time stop** after 5 hours still under +0.5%: idle capital is wasted.

## Measured on the dev window (kit, local grade)

| | |
|---|---|
| Net PnL | **+$23.60** on $1,000 (grade-local, process driver) |
| Trades | 15 (6 wins of 8 round trips) |
| Max drawdown | 1.54% |
| Guardrails | met (≥1 trade, drawdown ≤ 25%) |

The dev window itself falls ~4% top to bottom, so the money comes from
the V-shaped rebounds — the biggest one runs +5.5% from its trough.

## Tuning journey

- **v1** (plain dip legs, flat take-profit): +$0.30 — barely above water.
- **v2** (ride-the-V: bank a slice, trail the rest): +$5.64.
- **v3** (tighter dip thresholds, deeper capitulation leg): +$21.27.
- **Final** (leg sizes rebalanced, trailing giveback tuned): +$25.13 in the
  backtester, +$23.60 under the kit's own grader.

Tried and discarded: averaging down into a grinding downtrend (−$25.52),
a shallow-wiggle leg and a recovery-pullback leg (both added losing
trades). The lesson of the window: only sharp, confirmed capitulations
pay; shallow dips in a grind do not.

## Run it

```sh
python3 -P -s main.py
```

The bot speaks `agent-fun/bars-replay/v1` on stdin/stdout (one JSON line
per minute bar, one reply per batch). No network, no wall clock, no files
outside the repo, fully deterministic.

## Files

- `main.py` — the entry program (packed form of the readable source, made
  with the kit's `makefun compress`)
- `vendor/makefun_bars.py` — the kit's protocol shim, copied byte for byte
- `LICENSE` — MIT
- `.makefun/owner.json` — owner wallet
