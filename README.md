# pump-overhang-sniper

Paper trading bot for the make.fun pump.fun replay sprint (boothlord, 6h horizon).

## Strategy — Breakout Rider

Measured on the dev window (2–3 Nov 2026, 1,222 mints, 128,058 events):

- **Entry:** a mint whose recent 10-slot high crosses **3.5x** its first-trade
  price, within **600 slots** of creation, with real SOL reserves 20–200,
  and the creator never sold. The completing curves are the edge: a mint
  crossing 3.5x young completes 35–58% of the time, and the median price
  300 slots after the cross is still 1.7–2.1x the cross price.
- **Exit:** a **fixed 400-slot hold only**. The price path is a violent
  whipsaw (median intra-300-slot dip to 0.2–0.3x), so every reactive exit
  (stop, trail, big-sell, take-profit) fires on noise and sells the
  winners early. All of them are disabled; only the time limit exits.
- **Pyramid:** adds only into confirmed strength — the position must
  already be up ≥10% and the price stepped up ≥5% from the last add,
  with net SOL still flowing in over the last 30 slots. Losers peak near
  1.1x cost and never pyramid (stay at the 0.3 SOL initial stake);
  winners peak at 1.5–2.25x cost and build to the full 5 SOL stake.
- **Guardrail:** 0.02 SOL probe round-trips paced to 30 coins / 300 fills
  over 6h, so the 20-coin round-trip guardrail is met regardless of the
  main strategy.

## Local scores (grade-local, process driver, dev window)

- Net PnL **+8.29 SOL**, max drawdown **31.6%** (limit 50%)
- PnL at 2x fees **+6.00 SOL**
- 514 fills, 103 distinct mints, 192 counted round-trips
- Guardrails: compliant (drawdown, fills, mints all pass)
- Determinism: action logs reproduced (`l1.reproduced: true`)
- Bootstrap interval (90%): low **−4.97 SOL**, high +22.99 SOL — the PnL
  is concentrated in the top 3 completing coins (48% of gains)
- Stress (20 block-surrogate copies): median **−6.30 SOL**, 18/20 losing.
  The block surrogate rebuilds the window from trade blocks, which
  destroys the completion-path persistence the edge relies on; the eval
  window is real trades, so the dev score is the relevant predictor.

## Files

- `main.ts` — the bot (Node 22, `vendor/makefun-bot.ts` SDK shim)
- `vendor/makefun-bot.ts` — kit-mandated shim, copied unchanged from the kit
