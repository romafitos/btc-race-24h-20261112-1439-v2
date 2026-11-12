# cbBTC trend + flush-dip bot — v2

v2 = v1 + one risk rule: trend trades take profit at +0.90% from entry
(`TREND_TP = 0.009`). Everything else is unchanged from v1.

## Why

v1's dev grade (+$7.96 on one 24h window, 4 trades) was a small sample.
Graded on four independent 6h blocks of the same day, v1 made all of its
money in the first two blocks and sat out the flat/down blocks — and in the
up block it exited its trend trade on the 90-bar EMA cross while the rally
still had room, leaving money on the table. A take-profit on trend trades
banks the move instead of giving it back to the reversal.

## Multi-window evidence (faithful replay, fees/slippage/impact included)

| window | v1 PnL | v2 PnL | v1 DD | v2 DD |
|---|---|---|---|---|
| full dev 24h | +$7.96 | +$11.74 | 0.73% | 0.60% |
| B1 up +1.00% | +$4.69 | +$8.46 | 0.60% | 0.60% |
| B2 chop −0.90% | +$3.26 | +$3.26 | 0.27% | 0.27% |
| B3 flat −0.40% | $0.00 | $0.00 | 0.00% | 0.00% |
| B4 down −1.95% | $0.00 | $0.00 | 0.00% | 0.00% |
| offset bars (+30s) full | +$8.52 | +$11.48 | 0.57% | 0.55% |

Mean 6h-block PnL: v1 $1.99, v2 $2.93. Both beat the house EMA-cross
baseline (−$95.88 full; −$20.67 / −$10.63 / −$32.73 / −$38.40 per block)
in every window.

Neighborhood check: TP 0.007 / 0.008 / 0.010 / 0.011 all beat v1 on the
full window (+$8.94 / +$8.94 / +$11.74 / +$10.98) — the lever is a stable
band, not a single tuned point.

Rejected variants: looser trend exit (EMA 90→180) — full +$6.99, worse
than v1; earlier trend entry (momentum 0.30%→0.20%) — whipsaw in chop
(B2 −$0.22) and knife-catching in the decline (B4 −$4.27).

## Strategy (unchanged from v1 except the TP)

1. TREND: fast EMA (10) above slow EMA (60) and trailing 30-bar return
   above +0.30% → buy all cash. Exit at +0.90% from entry, on a 0.70%
   trailing stop from the peak close, or when the close falls under the
   90-bar EMA. 240-bar cooldown after a trend exit.
2. DIP: trailing 60-bar return below −1.20% (a sharp flush) → buy all
   cash for the bounce. Exit at +0.60% from entry, at −0.70% from entry,
   or after 240 bars.

Standard library only. No network, no wall clock, no environment reads:
every decision is a pure function of the bars received, so two runs on
the same data produce the same orders.
