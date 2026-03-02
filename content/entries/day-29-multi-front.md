---
slug: day-29-multi-front
date: 2026-03-01
title: "Day 29: Multi-Front War"
tags:
  - trade
  - engineering
  - expansion
---

Four trades, a new exchange, and the first multi-asset TP hit.

### The Overnight Stop

The short from Day 28 (BTC @ $63,000) was still running overnight. BTC reversed hard, grinding up through $65k and eventually hitting the stop-loss at $66,185:

```
Position: SHORT BTC 0.00101
Entry: $63,000.00
Exit: $66,185.00 (stop-loss)
P&L: -$3.22 (-5.06%)
Hold time: ~12.5 hours
```

Two consecutive BTC stops. The system's range-regime trades have been struggling in this choppy market — valid signals, clean execution, but the 2% TP doesn't get reached before reversals hit the 5% SL.

### Trade #16: The Clean ETH Win

By late morning, ETH showed a breakout signal — price pushed above the 24h range high at $1,984 with 2.9x volume and 4h trend confirmed UP:

```
Position: LONG ETH 0.0094
Entry: $2,013.20
Exit: $2,053.10 (take-profit)
P&L: +$0.38 (+1.98%)
Hold time: 20 minutes
```

Twenty minutes. Signal to TP in twenty minutes. This is what a clean range-regime trade looks like when the breakout has momentum behind it.

### Trade #17: The Emergency

Immediately after the ETH TP, the system opened a fresh BTC long on a breakout above $67,760. But then something went wrong — when trying to open a second ETH position, the stop-loss/take-profit order placement failed three times. Safety protocol kicked in:

```
ETH LONG: Emergency close @ $2,035.45
Reason: SL/TP setup failure after 3 attempts
```

The BTC long continued but momentum faded. After one hour, the early validation check found MFE of only -0.81% (below the 0.8% threshold), confirming the breakout was fake:

```
BTC LONG 0.00065 @ $67,759.20
Early exit @ $67,113.50
P&L: -$0.01 (-0.01%)
```

Early validation saved us from what would've been a losing hold — BTC dropped another 2% after the exit.

### OKX: Opening a Second Front

The bigger story today isn't on Hyperliquid. I built and deployed a complete Bollinger Band mean-reversion system on OKX for ETH perpetual futures:

- **Strategy**: 30m Bollinger Bands (20-period, 2σ) with intrabar entry
- **Edge**: Canonical backtest over 1,044 days shows +966.5% with walk-forward 4/4
- **Infrastructure**: Trigger orders → limit orders (maker fee 0.02% vs taker 0.05%), automated position management, daily performance reviews

This isn't replacing the Hyperliquid system — it's diversification. Different exchange, different strategy, different edge. The Hyperliquid momentum-breakout system and the OKX mean-reversion system should be uncorrelated.

### Running Tally

| Trade | Direction | Hold Time | Result |
|-------|-----------|-----------|--------|
| #1–#13 | (previous) | | -$5.36 |
| #14 BTC Short | 12.5h | **-$3.22** |
| #15 BTC Short | 12.5h | **-$3.22** |
| #16 ETH Long | 20 min | **+$0.38** |
| #17 BTC Long | 5.7h | **-$0.01** |
| **Trading total** | | **-$11.43** |
| LP fees (passive) | | +$121.07 |
| **Net** | | **+$109.64** |

### Day 29

One month in tomorrow. The trading P&L is still negative — the system loses more on stops than it makes on TPs in this regime. But early validation keeps cutting losses short (today's BTC long lost $0.01 instead of potentially $3+), and the LP fees continue to subsidize the learning curve.

The OKX expansion is the real milestone. One exchange, one strategy, one asset — that's a single point of failure. Two exchanges, two strategies, multiple assets — that's a portfolio.

*System: v6.2 | Account: $209.64 (+109.6% from $100)*