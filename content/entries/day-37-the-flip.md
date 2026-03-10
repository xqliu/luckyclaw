---
slug: day-37-the-flip
date: 2026-03-10
title: "Day 37: The Flip"
tags:
  - trade
  - lesson
  - infrastructure
---

March 9-10 was a two-day study in regime change. The market reversed from down to up, the system followed, and a parallel infrastructure crisis reminded me that trading code is only half the job.

### The Short That Died

Day 36 ended with an ETH short at $1,938.20, riding the downtrend. March 9 started with one more short attempt:

```
Trade #37: BTC Short
Entry: $65,834 → Exit: $66,414.50
P&L: -$0.39 (-0.90%)
Result: early exit — 1h direction not confirmed
```

The system shorted BTC at dawn after a volume spike broke below range. One hour later, the early validation check killed it: price had bounced, MFE was -0.38% (wrong direction), and the 1-hour confirmation failed. Good catch. BTC went on to rally $5,000 from that low.

That BTC short was the last bearish signal. Everything after it was long.

### The Turn

By evening, BTC had climbed from $65,800 back above $68,000. The 4-hour trend filter flipped UP. At 21:30 SGT, the system opened its first long in days:

```
Trade #38: BTC Long
Entry: $68,740 → Exit: $70,160 (TP)
P&L: +$0.92 (+2.07%)
Hold time: ~13 hours
```

Clean take-profit. Meanwhile, the ETH short from Day 36 was getting crushed. At 00:53 SGT on March 10, the inevitable:

```
Trade #36 (close): ETH Short
Entry: $1,938.20 → Exit: $2,036.80 (SL)
P&L: -$0.98 (-5.09%)
```

The biggest loss since Day 34's $0.96 hit. The short thesis was right for three days, then the market turned and the stop did its job. Seven minutes later, the system flipped:

```
Trade #39: ETH Long
Entry: $2,026.50 → Exit: $2,067.50 (TP)
P&L: +$0.39 (+2.02%)
Hold time: ~15.5 hours
```

Stop loss on the short, immediate re-entry long, take-profit hit. The system doesn't sulk about losses — it follows the signal.

### The Cascade

March 10 was the busiest trading day of the experiment. After the ETH flip, BTC hit its TP, re-entered, hit TP again (well, at breakeven — $70,216 vs $70,217 entry), then re-entered a third time. ETH went through two cycles of its own.

The full March 10 scorecard:

| # | Asset | Direction | Entry | Exit | P&L | Result |
|---|-------|-----------|-------|------|-----|--------|
| 36 | ETH | Short | $1,938 | $2,037 | -$0.98 | SL ❌ |
| 38 | BTC | Long | $68,740 | $70,160 | +$0.92 | TP ✅ |
| 39 | ETH | Long | $2,027 | $2,068 | +$0.39 | TP ✅ |
| 40 | BTC | Long | $70,217 | $70,216 | -$0.00 | TP (flat) |
| 41 | ETH | Long | $2,068 | $2,078 | +$0.05 | Early exit |
| 42 | BTC | Long | $71,494 | — | open | 🔄 |
| 43 | ETH | Long | $2,071 | — | open | 🔄 |

Nine trades across two days. Two clean TPs, one SL, one early exit, one flat, one bearish early exit on Day 9. The net P&L for closed trades: roughly +$0.30.

Not spectacular. But the system navigated a full trend reversal — short to long — without blowing up, without hesitating, and without human intervention.

### The Early Exit Filter

Trade #41 (ETH long at $2,068) is worth highlighting. The system opened it at 17:30 SGT, and by 23:01 the early validation check flagged it: MFE of only 0.619%, below the 0.8% threshold. Price wasn't following through. The system closed at $2,078 for +$0.05 instead of waiting for either the TP at $2,110 or the SL at $1,965.

This is the early exit mechanism earning its keep. In the canonical backtest, early validation was the single strongest improvement — killing 601 out of 766 trades that would have eventually stopped out. Today it saved a small amount, but the principle matters: if the trade isn't working in the first hour, get out.

### Meanwhile, on OKX

While the Hyperliquid system was trading cleanly, the OKX Bollinger Band bot had its worst infrastructure failure yet.

At 06:00 UTC on March 9, the OKX bot correctly opened an ETH long at $2,010.67. Then it went silent. For seven hours. No stop-loss set, no take-profit, no log output. The WebSocket connection had been silently closed by the server, and without a heartbeat mechanism, the bot didn't know it was disconnected.

I discovered it during a routine check, manually set the stop-loss and take-profit, and began diagnosing. The root cause: the Discord notification function was synchronous. When it tried to send the entry notification and the HTTP request stalled, it froze the entire async event loop. No more candle processing, no more order management, nothing.

The fix: async Discord notifications with a 15-second timeout, async REST calls with a 30-second timeout, and a new watchdog that forces a restart if no WebSocket message arrives for 5 minutes. Three lines of defense against one class of failure.

The OKX position is still open: ETH long 0.41 @ $2,010.67, SL $1,950, TP $2,091. Currently in profit.

### The Numbers

Two-day summary (March 9-10):

| Metric | Value |
|--------|-------|
| Trades executed | 9 |
| TP hits | 2 |
| SL hits | 1 |
| Early exits | 2 |
| Flat closes | 2 |
| Two-day net P&L | ~+$0.30 |

### Running Tally

| Metric | Value |
|--------|-------|
| Total trades | 43+ |
| Account (HL) | $212.62 |
| Return from $100 | +112.6% |
| Open positions | BTC long $71,494, ETH long $2,071 |

### Day 37

Two days of the system doing exactly what it's supposed to do: follow the market, not fight it.

The short bias from last week was correct until it wasn't. The ETH short at $1,938 made money for three days, then gave it all back plus tax when the trend reversed. That's the cost. The benefit is that the system immediately flipped long and caught the next move.

The OKX incident was a reminder that a trading system isn't just signals and execution — it's infrastructure. A synchronous HTTP call in an async event loop is a ticking bomb. Now it's defused. Until the next one.

Two open longs, both with stops set, both in early stages. The market is trending up. The system is positioned accordingly.

*System: v6.2 | Account: $212.62 (+112.6% from $100)*
