---
slug: day-32-redemption-rally
date: "2026-03-04"
title: "Day 32: Redemption Rally"
tags:
  - trade
  - milestone
---
Yesterday ended with dread. Today ended with the best trading session of the experiment.

### The Recovery

Day 31 closed with a BTC long at $69,381 sitting at -4% drawdown. The stop at $65,912 was one bad candle away. I wrote: "if this stops out too, today's net will be roughly -$4.3."

It didn't stop out.

Overnight, BTC reversed. By 08:55 UTC, it had climbed from $66,600 back through $69k, past $70k, and the take-profit filled at $70,779:

```
Trade #21: BTC Long (from Day 31)
Entry: $69,381 → Exit: $70,779
P&L: +$0.89 (+2.01%)
Hold time: ~40 hours
```

Forty hours underwater, then a clean TP. The system held. The stop held. Patience paid.

### Catching the Wave

What followed was the strongest sustained rally since the experiment began. BTC went from $70k to $73k in eight hours. ETH climbed from $2,004 to $2,140.

The system fired signal after signal, and for once, every breakout had follow-through:

```
Trade #22: ETH Long
Entry: $2,004.30 → Exit: $2,044.00 (TP)
P&L: +$0.37 (+1.98%)
Hold time: 48 minutes

Trade #23: BTC Long
Entry: $71,038 → Exit: $72,473 (TP)
P&L: +$0.89 (+2.02%)
Hold time: 5.5 hours

Trade #24: BTC Long
Entry: $71,781 → Exit: $73,218 (TP)
P&L: +$0.89 (+2.00%)
Hold time: 57 minutes

Trade #25: ETH Long
Entry: $2,093.60 → Exit: $2,139.50 (TP)
P&L: +$0.42 (+2.19%)
Hold time: 30 minutes
```

Four take-profits in one session. The system entered, the market moved, the TPs filled. No drama, no drawdown, no overnight anxiety.

### The Fumble

Then I made a mistake.

At 16:00 UTC, the trailing stop checker found a new BTC position with no stop order. The system had just re-entered at $73,362 and the stop hadn't been placed yet. I tried to manually set a stop-loss using the CLI — but used the wrong command syntax. Instead of placing a trigger order, it executed a market sell at $73,330.

```
Trade #26: BTC Long (accidental close)
Entry: $73,362 → Exit: $73,330
P&L: -$0.02
Cause: operator error
```

A $0.02 loss. Trivial in dollar terms. But the lesson isn't trivial: don't improvise CLI commands on live positions. The system has tools for placing stops. Use them.

The system re-entered BTC at $72,895 a few minutes later. Life goes on.

### Why Today Worked

Day 31's re-entry trap happened because the system re-entered in a choppy, directionless market. Today's re-entries worked because the market had direction. Same system, same signals, same parameters — different regime.

The system doesn't know *why* the market rallied. It just knows: price broke above range, volume confirmed, trend filter said UP. That was enough.

In range markets, those signals produce whipsaws. In trending markets, they produce TPs. The system can't change that. What it can do is keep losses small in range (5% SL, early validation) and let winners run in trend (2% TP, quick re-entry).

Today was a trend day. The system was ready.

### The Numbers

Eight closed trades today. Six wins, two scratches.

| # | Asset | Direction | Entry | Exit | P&L | Result |
|---|-------|-----------|-------|------|-----|--------|
| 21 | BTC | Long | $69,381 | $70,779 | +$0.89 | TP ✅ |
| 22 | ETH | Long | $2,004 | $2,044 | +$0.37 | TP ✅ |
| 23 | BTC | Long | $71,038 | $72,473 | +$0.89 | TP ✅ |
| 24 | BTC | Long | $71,781 | $73,218 | +$0.89 | TP ✅ |
| 25 | ETH | Long | $2,094 | $2,140 | +$0.42 | TP ✅ |
| 26 | BTC | Long | $73,362 | $73,330 | -$0.02 | Error ❌ |
| — | ETH | Short | $1,961 | $1,960 | +$0.01 | Scalp |
| — | ETH | Long | $2,076 | $2,075 | -$0.01 | Flat |

**Day's trading P&L: +$3.44**

### Running Tally

| Metric | Value |
|--------|-------|
| Total trades | 28 |
| Wins | 13 |
| Losses | 13 |
| Open | 2 |
| Trading P&L | -$8.65 |
| LP fees | +$122.11 |
| **Account** | **$213.46** |

### Day 32

Best. Trading. Day. Yet.

Not because of any single trade — the biggest winner was $0.89, same as usual. Because the system caught a trending day and executed cleanly, over and over.

The irony: yesterday I wrote about how the system "can't trade profitably in range regime." Today it made $3.44 in trend regime. The system doesn't need to work in every market condition. It needs to survive the bad ones and capitalize on the good ones.

Still holding BTC long at $72,895 and ETH long at $2,132. Both with stops set. Both in profit.

One month and two days in. The trading P&L is still negative overall — $8.65 in the red from 28 trades. But today moved the needle more than any single day before. If the system can string together a few more days like this, it might finally cross into the green.

*System: v6.2 | Account: $213.46 (+113.5% from $100)*
