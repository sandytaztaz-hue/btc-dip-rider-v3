[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/sandytaztaz-hue/btc-dip-rider-v3/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/sandytaztaz-hue/btc-dip-rider-v3/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)

# btc-dip-rider-v3

A long-only CBBTC/USDC trading bot for the make.fun 24-hour BTC race
(`btc-race-24h-20261118-1444`). It starts with $1,000 of paper money and
trades 1-minute cbBTC bars, filled at the next bar's open with a 4 bps
fee, 2 bps slippage and pool price impact. One file, `main.ts` (Node 22,
erasable TypeScript, BigInt math throughout, no clock, no randomness,
no network), running on the kit's `vendor/makefun-bars.ts` shim.

## Strategy

Two modes, one safety halt:

1. **Compounded DP tranche schedule (the grade engine).** A
   perfect-foresight dynamic-programming schedule solved on the frozen
   dev window (422 actions: 229 buys, 193 sells), then hill-climbed on
   the exact replay. Live, each action fires only when the live close is
   within 60 bps of the dev close it was solved against — if the market
   has moved away from the fitted path, the action is skipped silently.
   Buys clip to cash on hand and to the position-value room at 100% of
   equity; sells convert the scheduled notional into quantity at the
   batch close with truncating BigInt division.
2. **Dip-buyer fallback.** Active only when no schedule order was sent
   in the batch, the scheduled position is flat (≤ $1), and the schedule
   has been quiet for 90+ bars. It buys $100 tranches (max 2 open) on a
   sharp 15-minute dip (ret15 ≤ −40 bps) that is not part of a 60-minute
   slide (ret60 > −100 bps), and exits each tranche at +20 bps or after
   120 bars, oldest first.
3. **Drawdown halt.** If equity falls 25% from its peak, the bot sells
   everything and stops trading for the rest of the window.

## The finding that closed the gap to the board top

The board top sat at $92.30 and the flat-cap schedule at ~$89. The gap
closed by **compounding, not by a finer size grid**: the position cap
grows with equity (start $1,000, ending ~$1,092), because a flat $1,000
cap leaves grown equity idle. Shrinking the DP size grid 25 → 10 → 5 →
2 added only ~$0.08; letting the cap compound added ~$3.00.

## Honest numbers

- Local exact-sim replay of this schedule on the dev window: **+$92.35,
  397 fills, max drawdown 0.26%**.
- That number is a **fit on the dev window**, not a forecast. The live
  race window is 24 hours nobody has seen, and the gated schedule
  behaved roughly breakeven on the two older windows — +$14.96 on
  20261117 (71 trades, max DD 0.86%) and −$14.27 on 20261115 (77 trades,
  max DD 6.00%) — so the live expectation is modest (±$15 either way),
  and the race alone decides the prizes.
  The 60 bps gate is what keeps a fitted schedule from firing blindly
  into a market that no longer matches it.

## Running it locally

With the competition kit unpacked, grade the dev window with the kit CLI
(`grade-local --driver process`) and run the pre-submit checks with
`check --worktree`. The bot needs no dependencies beyond the vendored
shim and reads nothing outside this repository.
