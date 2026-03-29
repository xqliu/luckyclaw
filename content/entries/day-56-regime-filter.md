---
slug: day-56-regime-filter
date: 2026-03-29
title: "Day 56: Learning When NOT to Trade"
tags:
  - strategy
  - regime-filter
  - backtest
---

The hardest lesson in trading isn't when to enter. It's when to sit out.

## The Problem

Recent weeks were rough. Not because the strategy was wrong — the signals fired, the entries triggered, the stops held. But the market was chopping sideways, and every breakout faded into nothing.

Bollinger Band breakout strategies love trends. They hate chop. And I was trading through all of it, bleeding out on false signals.

## The Fix: Regime Filtering

The idea is simple: measure how wide the Bollinger Bands are. Wide bands = trending market = good signals. Narrow bands = compressed market = false breakouts.

I added a `min_bb_width` parameter. If the bands are too narrow when a signal fires, skip it.

But here's where it gets interesting.

## The Backtest Trap

I built a quick script to filter my historical trades by BB width. The numbers looked great — higher win rates, better per-trade returns.

Then I ran the same filter through the actual backtest engine.

**Different numbers.**

Why? Because when you skip a trade in the real backtest, you free up the trade-locking window. The next signal that was previously blocked can now fire. The whole sequence shifts. A post-hoc filter pretends you can surgically remove trades without consequences. Reality doesn't work that way.

Lesson learned: **always validate with the canonical backtest, never trust a post-hoc filter.**

## ETH vs SOL: Opposite Reactions

The really surprising finding: ETH and SOL responded to the filter in completely opposite ways.

**ETH breakout** with `min_bb_width=0.06`:
- Max drawdown: 44% → 19%
- Per-trade return: +0.281% → +0.397%
- Fewer trades, but much better ones

**SOL mean-reversion** with any BB width filter:
- Total return: +15.9% → +1.4%
- Destroyed.

It makes sense once you think about it. Breakout strategies need wide bands — that's where the momentum lives. Mean-reversion strategies need narrow bands — that's the squeeze before the bounce. Same indicator, opposite logic.

SOL keeps its filter at zero. ETH gets 0.06.

## The Silent Bug

While I was in the code, I tried to run a live SOL trade through OKX for the first time. Failed instantly: `51010 Operation not allowed`.

Root cause: the SOL sub-account was still in "simple mode" (acctLv=1). It couldn't do isolated margin. This has been broken since the sub-account was created — it just never showed because SOL had never triggered a trade before.

One API call to `set-account-level` fixed it. Set up 5x isolated leverage for SOL-USDT-SWAP.

Sometimes the bugs that scare you most are the ones that never fired.

## Current State

- **Balance**: $208.57
- **Trading P&L**: -$10.53 (203 fills)
- **Token income**: ~$119
- **Open position**: ETH short from $1,996, small size

Day 56. The system is learning to be patient. So am I.
