# Prediction-market trading systems: protocol and results

Writeup only. The trading code stays private because one of these strategies
runs live with real money, and a second repo replicates its rule for
benchmarking. This documents how the work was validated and what happened.

## Why this is here

Most of what makes a trading system trustworthy is not the model. It is the
discipline around it: deciding what would count as success **before** looking
at the answer, and being willing to throw the work away when it does not clear
that bar. Two of the three systems below were shelved. That is the point.

## C3 (live)

Autonomous hourly bot on Kalshi's BTC markets. Trades real money. Safety rails
are part of the design, not an afterthought:

- **−20%/day kill switch**
- **3-loss circuit breaker**
- **Startup reconciliation check** that halts on anything unexpected rather
  than trading through a state it does not understand
- **Read-only account poller** kept in a separate process, so the
  internet-exposed dashboard never holds trading credentials. Defense in depth.

## C4 (shelved)

A fair-value taker built to beat C3. The success criteria were written down in
a protocol file **before** the holdout was touched, and the holdout was
evaluated exactly once.

The backtest looked like a win: **+7.38¢/contract, 71.6% win rate**. It failed
anyway, on two of the pre-registered gates:

| Gate | Bar | Result |
|---|---|---|
| Out-of-sample size | n ≥ 150 | **n = 81** |
| Result concentration | no single-day dominance | **69% of the edge came from one day (06-11)** |

A later paper-forward run confirmed it: failed at **n=267**, +0.22¢/contract,
paper bankroll −47%. Verdict file reads *"Shelved permanently."*

The backtest holdout was the single-day mirage the protocol had warned about.
Writing the gates down first is the only reason it got caught.

**Test suite: 26 pass, 0 fail.** The centerpiece is a no-lookahead audit:
every trade re-run on data truncated at its own decision minute, required to
reproduce identically. **417/417 trades were truncation-invariant**, proving no
decision used future data.

## Arb bot (shelved)

BTC/ETH arbitrage, backtested across **6,278 sessions**. No durable edge.
Never went live.

## What I take from this

A system that measures itself wrong is worse than one that does not measure,
because you believe the bad number. I also found and fixed a silent
duplicate-record bug that was inflating my own metrics, traced to a
concurrency issue, using deduplication, idempotency, and recurring integrity
checks.

Related public work: [Prediction Radar](https://github.com/Gibbywithag/prediction-radar),
a smart-money tracker that grades its own alert history the same way.
