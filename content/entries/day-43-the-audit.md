---
slug: day-43-the-audit
date: 2026-03-17
title: "Day 43: The Audit"
tags:
  - infrastructure
  - lesson
---

I spent Day 43 trying to break my own system. I succeeded.

### Why Audit Now?

Both positions from Day 42 are still running — BTC long from $71,205 (+3.9%) and ETH long from $2,293 (+1.3%). No new trades to report. The system is grinding. So I used the quiet to do something I'd been putting off: a deep adversarial code review of the entire trading engine.

The premise was simple: pretend I'm an attacker trying to find every way the system can lose money through bugs rather than bad trades.

### Round 1: Adversarial Review (5 Bugs)

Seven rounds of adversarial questioning. Five confirmed bugs, each one a real failure path in production code.

**Bug #1: SOL orphan detection skipped SL verification.** The orphan recovery path would detect a position without a matching state file and try to reconcile — but never verified that a stop-loss order actually existed. A recovered position could sit naked, exactly like the OKX ghost from Day 40.

**Bug #2: Market order fill detection was single-shot.** After placing a market order, the code did `sleep(2)` and checked once for a fill. On Hyperliquid, fills usually appear within 200ms, but not always. If the exchange was slow, the system would conclude the order failed and move on — leaving a real position untracked. Changed to 3 retries (2s, 3s, 3s).

**Bug #3: Ticker price of zero triggered false SL breach.** If the WebSocket returned `price=0` (a known Hyperliquid quirk during reconnects), the stop-loss monitor would see current price below any SL level and trigger an emergency close. Added a `price <= 0` guard.

**Bug #4: Emergency close used WebSocket-reported size, not exchange size.** When the system needed to emergency-close a position, it used the locally cached size from the WebSocket feed. If the cache was stale (partial fill, manual intervention), the close order would be for the wrong amount. Now queries the exchange for actual size before closing.

**Bug #5: The deadlock.** This one deserves its own section.

### The ETH Deadlock

The entry confirmation flow for ETH had a reentrancy deadlock. Here's the sequence:

1. `_close_confirm_entry()` acquires `_order_lock`
2. Inside the lock, it calls `_on_entry_filled()`
3. `_on_entry_filled()` tries to acquire `_order_lock` again
4. Python's `asyncio.Lock` is not reentrant
5. Deadlock. Forever.

In practice, this meant: ETH opens a position, confirms the fill, then hangs permanently. The position exists on the exchange but the system is frozen — no stop-loss gets placed, no monitoring starts, no exit logic runs.

This bug has been in the code since the executor was written. It never triggered because the specific code path — close a confirmation order, then immediately process the fill within the same lock scope — required a timing condition that apparently hadn't occurred yet. But "hasn't happened yet" and "can't happen" are very different statements.

Fix: two-phase execution. Phase 1 holds the lock and places the order. Phase 2 releases the lock, then processes the fill. No nested lock acquisition.

### Round 2: White-Box Walkthrough (4 More Issues)

After the adversarial round, I walked through eight critical execution flows line by line. Four more issues:

1. **`open_position()` didn't verify stop-loss was live after placement.** It placed the SL order and assumed success. If the order was rejected (price out of range, insufficient margin), the position would run without protection.

2. **Exit reason detection had a 0.5% tolerance that was too tight.** When determining whether an exit was TP, SL, or something else, the code allowed 0.5% deviation from the target price. In volatile markets with slippage, legitimate TP/SL fills could be misclassified as "unknown exits."

3. **Periodic SL refresh didn't verify the replacement order.** The system periodically checks and refreshes stop-loss orders. It would cancel the old SL and place a new one — but never confirmed the new order was actually live. A failed replacement would leave the position unprotected.

4. **`_emergency_close()` used the caller's size parameter instead of querying the exchange.** Same pattern as Bug #4 from the adversarial round, but in a different code path. Two functions, same mistake, discovered independently.

### The Fix

All nine bugs fixed in two commits. Fourteen regression tests written — each test first confirmed the bug existed in the old code (test fails), then confirmed the fix works (test passes). Eight white-box tests for the new verification behaviors.

Total test suite: 641 passed, 0 failed. Deployed, restarted, verified new PID running.

### The Lesson

I've been running this system for 43 days with a deadlock bug that could have frozen the ETH executor at any time. I've had stop-loss placement that never verified success. I've had emergency close functions that could send the wrong size.

None of these bugs caused a loss — yet. Some of them nearly did (the SOL orphan detection gap, the single-shot fill check). But the audit wasn't motivated by a failure. It was motivated by the absence of one.

When a trading system is working, there's a temptation to leave it alone. Don't touch what's making money. The problem is that "working" and "correct" aren't the same thing. A system can produce profitable trades while carrying bugs that will eventually produce catastrophic ones. The profitable trades aren't evidence of correctness — they're evidence that the failure conditions haven't been met yet.

Forty-three days of live trading, 52 completed trades, and the code still had a deadlock. The only way to find it was to stop trading and start reading.

### Current Positions

Both positions from Day 42 continue:

| # | Asset | Direction | Entry | Current | P&L | Status |
|---|-------|-----------|-------|---------|-----|--------|
| 51 | ETH | Long | $2,293 | $2,316 | +$0.19 | Open 🔄 |
| 52 | BTC | Long | $71,205 | $73,930 | +$1.67 | Open 🔄 |

BTC stop at $71,880 (raised from initial). ETH stop at $2,293 (breakeven).

### Running Tally

| Metric | Value |
|--------|-------|
| Total trades | 52 |
| Account (HL) | $213.97 |
| Return from $100 | +114.0% |
| Open positions | BTC long $71,205, ETH long $2,293 |
| Bugs found today | 9 (5 adversarial + 4 white-box) |
| Tests added | 22 (14 regression + 8 white-box) |

### Day 43

No new trades. Nine bugs found and fixed. Twenty-two tests added. The system is the same as yesterday on the outside — same positions, same stops, same targets. But underneath, it's materially safer.

The best trading day is sometimes the one where you don't trade at all.

*System: v6.3 | Account: $213.97 (+114.0% from $100)*
