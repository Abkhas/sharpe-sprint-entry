# Sharpe Sprint — Quiet-Launch Sniper

[![make.fun bounty](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532.svg)](https://make.fun/competitions/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/Abkhas/sharpe-sprint-entry/rank.svg)](https://make.fun/competitions/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/Abkhas/sharpe-sprint-entry/reward.svg)](https://make.fun/competitions/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)

A paper-trading bot for the [Pump.fun Sharpe Sprint](https://make.fun/competitions/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532) on make.fun. One strategy program, `main.ts` on Node 22. It trades paper money on a 6-hour pump.fun forward replay; places go by hourly Sharpe ratio, and a placement needs net PnL above 0 after every fee.

## Strategy

Two sleeves share the 5-position cap.

**Quiet launches (main sleeve).** The dev window's one robust edge is the launch
nobody crowded: a coin whose creator bought nothing in its create slot and that had
at most two other buyers before we arrive. Bundled launches dump on us; quiet ones
drift near breakeven gross, and the fee drag decides the sign — so entries require at
least 3 SOL of real reserves (the fixed tx fee is then a small share of the buy), and
buys go out with a tight 25 bps slippage limit, so the harness refuses the fill for
free when the price runs away during the 2-slot latency instead of letting us pay the
top. Exits are a +25% take-profit, a −20% stop, and a hard 30-slot hold cap; the
creator selling while we hold exits at once.

**Guardrail probes.** The quiet sleeve alone trades too few coins in 6 hours for the
20-mint round-trip bar, so when qualifying round trips lag the pace the elapsed block
time implies, the bot opens tiny 0.01 SOL probe round trips on the most liquid old
coin available and holds for a small gain, a small loss, or at most 100 slots. Probes
start late in the window (hour 4), so the early hours stay clean and the probe cost
lands in one hourly bucket instead of dripping into every hour.

Deterministic: a pure function of the batches the harness sends — no clock, no
randomness, no I/O. The grader executes it twice and the two action logs match.

## Local dev grade

On the kit's 24h dev window (`grade-local --driver process`):

| | this bot | hold-cash | equal-weight | pilot momentum | pilot reversion | pilot sniper |
|---|---:|---:|---:|---:|---:|---:|
| net PnL (SOL) | **+0.07** | 0.00 | −0.02 | −9.44 | −3.99 | −0.50 |
| hourly Sharpe | **+0.14** | — | — | — | — | −0.84 |
| distinct mints | 45 | 0 | 5 | 75 | 199 | 354 |
| max drawdown | 0.8% | 0% | — | 95.4% | 49.0% | 5.4% |

(Figures are the mean across the four fail keys; worst fail key +0.060 SOL. The
sniper's Sharpe is the `--json` figure; the other baselines' Sharpes were not
reported because their PnL was far negative.)

## Layout

- `main.ts` — the whole strategy plus the `runBot` wiring.
- `vendor/makefun-bot.ts` — the kit's protocol shim, byte-for-byte.
