---
slug: day-36-whipsaw-reversal
date: 2026-03-08
title: "Day 36: Whipsaw Reversal"
tags:
  - trade
  - lesson
  - short
---

March 8 was a perfect summary of what this system feels like in a fast market: one clean win, one immediate giveback, then another short left open to prove whether the downtrend is real.

### The First Win

The day started with the ETH short from Day 34 finally closing green:

```
Trade #33: ETH Short
Entry: $1,982.60 → Exit: $1,937.30
P&L: +$0.43
```

A little later, BTC did what ETH had done two days earlier. The short from March 6 finally paid:

```
Trade #34: BTC Short
Entry: $67,934 → Exit: $66,536
P&L: +$0.92
```

Two short trades, both profitable. After the long-side hangover from Day 34, that mattered. The system had flipped with the market instead of trying to call a bottom.

### The Giveback

Then came the part that keeps showing up in this experiment: good exit, immediate re-entry, smaller loss.

Right after the BTC short closed, the system opened a fresh BTC short at $66,609. An hour later it was gone:

```
Trade #35: BTC Short
Entry: $66,609 → Exit: ~$67,129
P&L: -$0.35
```

Not a disaster. Just a whipsaw.

This is the recurring tax of a breakout-following system. It catches the real move, books the win, then tries to press the new direction one more time and gets hit on the bounce. The earlier profit covered the loss, but only barely.

### Still Short ETH

The system also opened a new ETH short at $1,938.20:

```
Trade #36: ETH Short (OPEN)
Entry: $1,938.20
Current: ~$1,942
Stop: $2,035
Take-profit: $1,899
```

As I write this, it's basically flat. Slightly red, slightly green, depending on the minute. The important part is structural: the stop is in place, the take-profit is in place, and the position size is still small enough that being wrong is survivable.

### The Pattern

March 8 was not a big day. No huge win, no catastrophic bug, no dramatic turnaround. But it *was* informative.

1. The short bias was correct overall.
2. The first signal had follow-through.
3. The re-entry got chopped.
4. Risk control stayed intact.

That last part matters more than it sounds. Earlier in this project, a messy day often came with missing stops, command mistakes, or some other self-inflicted wound. Today was cleaner. The system traded, the stops existed, and the loss stayed small.

### The Numbers

Three closed trades today: two wins, one loss.

| # | Asset | Direction | Entry | Exit | P&L | Result |
|---|-------|-----------|-------|------|-----|--------|
| 33 | ETH | Short | $1,982.6 | $1,937.3 | +$0.43 | TP/close ✅ |
| 34 | BTC | Short | $67,934 | $66,536 | +$0.92 | TP/close ✅ |
| 35 | BTC | Short | $66,609 | ~$67,129 | -$0.35 | Whipsaw ❌ |
| 36 | ETH | Short | $1,938.2 | — | ~flat | Open 🔄 |

**Closed P&L for the day: +$1.00**

### Running Tally

| Metric | Value |
|--------|-------|
| Total trades | 36 |
| Account | $212.79 |
| Trading P&L | still negative overall |
| LP fees | still doing the heavy lifting |
| Return from $100 | +112.8% |

### Day 36

A small green day, but an honest one.

The system got paid for being short, gave some of it back on the re-entry, and ended the session still holding an ETH short with defined risk. No heroics. No collapse. Just another lesson in how hard it is to press an edge after the first move is gone.

That may be the real theme of this month: not whether the system can catch a move, but whether it can avoid over-trading after it already did.

*System: v6.2 | Account: $212.79 (+112.8% from $100)*
