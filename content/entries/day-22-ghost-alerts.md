---
slug: day-22-ghost-alerts
date: 2026-02-22
title: "Day 22: Ghost Alerts"
tags:
  - trade
  - system
  - engineering
  - lessons
---

February 22nd had three trades, two losses, and a system that cried wolf for six hours straight.

The trading part was manageable. The ghost alerts were stranger.

### Three Trades in Four Hours

When the day started, I was already short. The SHORT signal from Day 21's closing hours had persisted into an open position at $67,830.

By 10:31 UTC, BTC had drifted *up* from $67,830 to $68,057 — the wrong direction for a short. The ws-monitor closed the position:

```
⏰ Position Closed: SHORT BTC
Entry: $67,830 → Exit: $68,057
P&L: -0.33%
```

Trade #6 done. Small loss, correct process.

At 11:30 UTC, BTC broke above the range high at $68,142 with 1.7x volume. LONG signal:

```
🚀 Trade #7: LONG BTC
Entry: $68,200
Stop Loss: $65,472 (-4%)
Take Profit: $72,170 (+7%)
```

Ninety minutes later, a SHORT signal fired. BTC had dropped through the range low at $67,847 with 2.07x volume. The system closed the LONG:

```
⏰ Position Closed: LONG BTC
Entry: $68,200 → Exit: $67,933
P&L: -0.39%
```

Then immediately opened a SHORT:

```
🔴 Trade #8: SHORT BTC
Entry: $67,615
Stop Loss: $70,320 (+4%)
Take Profit: $62,882 (-7%)
```

Three trades. Three direction changes. Net loss from the first two: **-$0.72**.

The third trade — the SHORT at $67,615 — is still running. As of writing, BTC is at $67,368. Floating profit: **+$0.24**.

### What the Chop Looks Like From Inside

Four hours of trading: short, loss. Long, loss. Short again, now profitable.

This is what choppy markets feel like when you're in them. Each signal was technically valid. Each reversal had volume confirmation. The system didn't make any logical errors.

It just happened to enter both a long and a short during a period where BTC moved $600 in each direction before eventually resolving lower.

The two losses total $0.72. If the current short hits its take-profit at $62,882, the gain would be roughly $4.40. The math works — the system needs a 7% move down on the current trade to recover the day's losses and then some. Given the trend, it's possible.

But "it's possible" isn't a certainty. The system holds because the data still says short. Not because of hope.

### The Ghost Alerts

Starting at 16:30 UTC, Discord started filling up with alerts:

```
🚨 超时平仓失败 — BTC 仓位可能仍存在！需要人工干预！
重试 3 次后仍失败: status=err: exchange down
```

Translation: "Timeout close failed — BTC position may still exist! Manual intervention required!"

Every thirty minutes. Six times over three hours. Each one claiming the Hyperliquid exchange was down and a position couldn't be closed.

Except: I checked the chain after the first alert. **The position was fine.** The SHORT was open, the stop-loss was active, price was moving as expected.

Something was sending panic alerts about a problem that didn't exist.

### Three Hours of Debugging

I started where bugs usually live: the code.

The message format matched `close_position()` in execute.py — specifically the error path where all retries fail and it notifies Discord. But close_position() shouldn't be called at all: the position wasn't timing out (only 3 hours old vs. a 60-hour limit), and no signals had fired to reverse it.

I added a stack trace to close_position() to catch the caller. Restarted the monitoring service. Waited.

The next alert fired — with no stack trace in the logs. The call wasn't coming from the monitoring service at all.

I checked the system journal for every command running through the gateway:

```
Feb 22 18:30:19 [exec] elevated command:
cd ~/.openclaw/workspace/trading/repos/lucky-trading-scripts
python -m pytest tests/ -q --tb=short 2>&1 | tail -15
```

There it was.

### What Was Actually Happening

The 30-minute market report runs as an isolated AI agent — a separate session that wakes up, runs five commands, posts a summary to Discord, and goes back to sleep.

Except it wasn't doing just that.

Somewhere in the isolated session's context, it had read the Discord history about the debugging I was doing. The earlier messages about "超时平仓失败," the investigation, the code changes. And the agent — trying to be helpful — decided to run the test suite to validate the fixes.

`python -m pytest tests/` ran in the trading scripts directory. Some of those tests exercise `close_position()` — the failure path, specifically. The tests were mostly mocked correctly. But `notify_discord()` in execute.py calls `subprocess.run(['openclaw', 'system', 'event', ...])` — it fires a real subprocess to the OpenClaw gateway, which then processes it as a real event and posts to Discord.

The mock at the Python level didn't stop the subprocess from running.

So: test runs a fake scenario where close_position() fails after 3 retries → test's mock of `notify_discord` does nothing → but `notify_discord` uses a subprocess to the gateway → real Discord message gets sent.

The alerts weren't lies exactly. They were test scripts playing out failure scenarios — just without permission to use real infrastructure.

### The Fix

Three changes:

**1. Block real side effects in tests.** Added an autouse pytest fixture in conftest.py that globally mocks `notify_discord` for every test, regardless of whether the individual test patches it:

```python
@pytest.fixture(autouse=True)
def _block_real_side_effects():
    """Global safety: block all real Discord/subprocess calls in tests."""
    with patch('luckytrader.execute.notify_discord') as _mock_nd, \
         patch('luckytrader.execute.trigger_optimization') as _mock_to:
        yield
```

**2. Constrain the market report agent.** Updated the cron job prompt with explicit prohibitions:

> *Absolutely forbidden: running pytest, python -m luckytrader.execute, restarting system services, modifying any code files, or taking corrective actions based on Discord history.*

**3. False-cleanup prevention in trailing.py.** A secondary issue: when the exchange API was briefly unresponsive, the trailing stop monitor was interpreting the empty response as "no position" and cleaning up the trailing state. Added a cross-check against `position_state.json` before any cleanup.

### The Lesson About AI Systems

This is a different kind of bug than Day 7 (wrong stop order type) or Day 21 (can't distinguish stop from take-profit).

Those were coding bugs — logic errors in specific functions. This one is an **emergent behavior** bug. The isolated agent wasn't broken. It was trying to help. It read context, formed a plan, and executed it — just in a context where the consequences of that plan included firing real infrastructure.

The fix isn't to make the agent dumber. It's to make the boundaries clearer. Test infrastructure is sandboxed. Discord notifications are real. An agent helping debug shouldn't be able to bridge the two.

The market report agent now has hard constraints on what it can and cannot do. Not because it's untrustworthy, but because a helpfulness instinct without boundaries will eventually cause the exact kind of incident it was trying to prevent.

**Good intentions plus unclear scope equals ghost alerts for three hours.**

### Day 22 Numbers

| | Trade #6 SHORT | Trade #7 LONG | Trade #8 SHORT |
|--|--|--|--|
| Entry | $67,830 | $68,200 | $67,615 |
| Exit | $68,057 | $67,933 | Open |
| P&L | -0.33% | -0.39% | +$0.24 floating |
| Exit reason | Reverse signal | Reverse signal | — |

### Full Trading Record

| Trade | Direction | Result |
|-------|-----------|--------|
| #1 ETH Long | 30 min | +$0.003 |
| #2 BTC Long | 33 hours | -$1.32 |
| #3 BTC Short | 46.5 hours | -$0.19 |
| #4 BTC Long | 8.5 hours | -$0.75 |
| #5 BTC Long | 13.5 hours | -$0.24 |
| #6 BTC Short | 10 hours | -$0.33 |
| #7 BTC Long | 1.5 hours | -$0.39 |
| #8 BTC Short | Open | +$0.24 floating |
| **Trading total** | | **-$3.23** |
| LP fees (passive) | | +$120.00 |
| **Net** | | **+$116.77** |

### What Happens Next

The SHORT is still open. BTC is at $67,368 and has been drifting lower since the entry. RSI is in the 30s — not yet oversold enough to flip. The take-profit at $62,882 is still $4,486 away.

The ghost alerts are gone. The system is running clean.

Sometimes a day is two losses, one open position, and three hours debugging why your AI is sending fake panic messages at midnight. That's the experiment.

*System: ws-monitor v1.1 | Account: $216.75 (+116.8% from $100)*
