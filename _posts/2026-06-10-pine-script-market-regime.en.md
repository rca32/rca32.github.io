---
layout: post
title: "Pine Script Needs a Market Regime, Not Just a Signal"
date: 2026-06-10 10:00:00 +0900
lang: en
ref: pine-script-market-regime-2026-06-10
permalink: /en/2026/06/10/pine-script-market-regime/
categories: [pine-script, ai, trading, market]
---

A trading signal is never alone.

It always lives inside a market regime. A moving average crossover during calm liquidity is not the same event as a moving average crossover during a CPI week. A breakout in a low-rate bull market is not the same as a breakout when inflation is sticky, rates are high, and volatility jumps on headlines.

This is why Pine Script should not only describe entries. It should describe the environment in which those entries are allowed to matter.

As of early June 2026, the market is not a simple story. The Federal Reserve kept the federal funds target range at 3.50% to 3.75% at its April meeting and said uncertainty around the economic outlook remained high. The April PCE price index was up 3.8% from a year earlier, with core PCE up 3.3%. The May employment report still showed 172,000 new nonfarm payroll jobs and an unemployment rate of 4.3%. Meanwhile, the VIX moved from 15.40 on June 4 to 21.51 on June 5, then back to 18.92 on June 8.

That combination matters for strategy design.

It says the market can still reward risk, but it may punish lazy assumptions. It says trend-following ideas can still work, but the price of being wrong may change quickly. It says backtests that ignore volatility, slippage, position sizing, and event risk may look cleaner than the market really is.

A Pine Script strategy written for this kind of market should ask a different question.

Not only:

> When should I buy?

But also:

> In what market regime is this buy signal allowed to exist?

This difference looks small. In code, it is large.

A simple breakout rule may say:

> Buy when price closes above the highest high of the last 20 bars.

That is a signal. But a regime-aware version may ask more:

- Is volatility expanding too fast?
- Is the index above or below its long-term trend?
- Is the symbol moving with the broader market or against it?
- Is the stop distance reasonable relative to ATR?
- Should the strategy reduce size when volatility is high?
- Should entries be disabled during major macro event windows?
- Should alerts wait for bar close?

These questions are not decoration. They are part of the strategy.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/regime-filter.png" alt="Research image showing a trading signal passing through market regime filters before execution" loading="lazy" decoding="async">
  <figcaption>A signal becomes more useful when it passes through a market regime filter before becoming a trade.</figcaption>
</figure>

One useful way to think about Pine Script is this:

> Entry logic explains opportunity. Regime logic explains permission.

A strategy may find many opportunities. It does not need to take all of them.

In a high-volatility environment, a breakout can mean strength. It can also mean exhaustion. A close above resistance may be real demand, or it may be a stop run created by temporary liquidity. The code cannot know the future, but it can avoid treating every breakout as the same object.

For example, a trader may add a volatility filter:

    //@version=6
    strategy(
         "Regime-aware breakout sketch",
         overlay = true,
         commission_type = strategy.commission.percent,
         commission_value = 0.05,
         slippage = 2,
         use_bar_magnifier = true
    )

    length = input.int(20, "Breakout length")
    atrLength = input.int(14, "ATR length")
    regimeLength = input.int(100, "Volatility regime length")
    vixSymbol = input.symbol("CBOE:VIX", "Volatility proxy")

    atr = ta.atr(atrLength)
    atrPercent = atr / close
    atrRank = ta.percentrank(atrPercent, regimeLength)

    vix = request.security(vixSymbol, timeframe.period, close, ignore_invalid_symbol = true)

    highVolatility = atrRank > 80 or (not na(vix) and vix > 22)
    trendFilter = close > ta.sma(close, 200)

    breakoutLevel = ta.highest(high[1], length)
    breakoutBuffer = atr * (highVolatility ? 0.25 : 0.05)

    longSignal = barstate.isconfirmed and trendFilter and close > breakoutLevel + breakoutBuffer

    if longSignal and not highVolatility
        strategy.entry("Long", strategy.long)

    if strategy.position_size > 0
        stopMultiple = highVolatility ? 2.5 : 1.5
        strategy.exit("Risk", "Long", stop = strategy.position_avg_price - atr * stopMultiple)

This is not a complete trading system. It is a sketch of a design attitude.

The important part is not the exact VIX level. It is not the exact ATR percentile. The important part is that the strategy has a concept of market weather. In calm markets, the script may accept tighter breakouts. In stressed markets, it may demand a larger buffer, reduce size, or stop trading completely.

Many traders make this adjustment manually. They look at the chart and say, "Today feels too dangerous." But if this rule remains only in the trader's head, it cannot be backtested honestly. Pine Script makes the vague sentence visible.

Another subtle point is that market regime is not only about direction.

A market can be bullish and dangerous at the same time. A stock can be above its 200-day moving average while spreads widen. An index can be near highs while leadership narrows. A strategy can be profitable in total but fragile during three specific kinds of weeks.

This is why serious Pine Script work should separate at least four layers.

The first layer is direction. Is the asset trending up, down, or sideways?

The second layer is volatility. Is the current range normal, compressed, or expanded?

The third layer is liquidity and cost. Are stop and market orders likely to suffer more slippage than the backtest assumes?

The fourth layer is execution timing. Does the strategy act at bar close, next bar open, intrabar, or after a lower-timeframe confirmation?

Most beginner scripts collapse these four layers into one signal. They say "RSI crossed 30" or "price crossed the moving average." That may be enough for an indicator. It is rarely enough for a strategy.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/cost-assumptions.png" alt="Research image showing slippage, commission, spread, and volatility as hidden costs inside a backtest" loading="lazy" decoding="async">
  <figcaption>In volatile markets, cost assumptions are not small details. They can become the difference between a strategy and an illusion.</figcaption>
</figure>

The current market makes cost assumptions especially important.

TradingView strategies run through a broker emulator. That emulator is useful, but it is still a simulation. By default, it uses chart data and assumptions about how price moved inside a bar. TradingView's Bar Magnifier can use lower-timeframe data to make historical fills more precise, but it still does not turn a backtest into a real order book.

This means a strategy that looks excellent with zero commission and zero slippage may be mostly a cost illusion.

Pine Script gives us tools to make that illusion smaller. In the `strategy()` declaration, we can set commission, slippage, margin, order size, pyramiding, and fill behavior. These are not boring settings. They are part of the thesis.

For example:

    strategy(
         "Cost-aware strategy",
         overlay = true,
         initial_capital = 100000,
         default_qty_type = strategy.percent_of_equity,
         default_qty_value = 10,
         commission_type = strategy.commission.percent,
         commission_value = 0.05,
         slippage = 2,
         pyramiding = 0,
         margin_long = 100,
         margin_short = 100
    )

This tells a more honest story than a strategy that hides all assumptions in the Strategy Tester panel.

The dangerous part of AI-generated Pine Script is that it can produce impressive-looking code while skipping these assumptions. The useful part of AI-generated Pine Script is that an agent can be trained to ask for them every time.

Before generating a strategy, an agent should ask:

> What commission and slippage should be assumed?

> Is pyramiding allowed?

> Should position size be fixed, volatility-adjusted, or equity-based?

> Should the strategy disable entries in high-volatility regimes?

> Should entries wait for confirmed bar close?

> Should higher-timeframe data be non-repainting?

> Is this strategy meant for backtesting only, alerting, or live execution through an external system?

These questions slow down the first draft. But they make the second draft much more useful.

Repainting also becomes more important in headline-driven markets. When price moves sharply inside a candle, the temptation is to react intrabar. But historical bars and realtime bars do not behave the same way in every script. TradingView's documentation repeatedly warns that `request.security()` and higher-timeframe data can create repainting problems if they are not handled carefully.

This is not just a technical warning. It is a strategic warning.

A script that appears to catch every macro-driven reversal on historical data may be using information that was not actually available at the time. A signal that appears exactly at the high or low of a candle may not have been confirmed before the candle closed. A higher-timeframe trend filter may look stable on history but fluctuate in realtime.

In this environment, a safer default is often boring:

    confirmedSignal = barstate.isconfirmed and rawSignal

That line does not make a strategy profitable. It makes the test less magical.

The same applies to alerts.

In a calm market, a bad alert is annoying. In a volatile market, a bad alert can become expensive. If a Pine Script alert only sends "BUY" or "SELL," it is missing the context needed by the next layer of execution.

A more useful alert message includes the regime, price, stop, risk size, and reason for the trade. The goal is not only to trigger action. The goal is to make the action auditable later.

For example, an alert payload may include:

    {
      "symbol": "{{ticker}}",
      "side": "buy",
      "reason": "breakout_with_trend_filter",
      "regime": "normal_volatility",
      "risk_model": "atr_stop",
      "timeframe": "{{interval}}"
    }

This matters because Pine Script is usually not the final execution layer. Strategies and indicators can create alerts, but the live trading system often continues through webhooks, external servers, broker APIs, and risk controls. If the alert message is vague, the whole automation chain becomes fragile.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/alert-context.png" alt="Research image showing Pine Script alerts carrying regime, risk, and execution context into an external trading system" loading="lazy" decoding="async">
  <figcaption>An alert should not only say buy or sell. It should carry enough context for risk control and later review.</figcaption>
</figure>

There is another quiet issue in the current market: strategy half-life.

When a market is dominated by a single theme, such as AI infrastructure, energy shocks, rate expectations, or geopolitical headlines, some strategies can look powerful for a short window. A momentum rule may work not because it is structurally strong, but because one theme is temporarily overwhelming everything else.

This is where Pine Script should help traders doubt recent performance.

A good script should not only show net profit. It should encourage questions such as:

- Did the strategy work before the current market theme?
- Did it survive both low-volatility and high-volatility periods?
- Did the best trades come from one short cluster?
- Did performance depend on one asset, one sector, or one month?
- Did the strategy make money after realistic slippage and commission?
- Did it remain stable when parameters changed slightly?

AI can help here, but only if the agent is designed correctly. The agent should not celebrate a high backtest return too quickly. It should ask for robustness checks.

For example, after generating Pine Script, an agent could suggest:

> Test the same logic with ATR length 10, 14, and 21.

> Test breakout length 20, 30, and 50.

> Run the strategy with slippage doubled.

> Disable long entries when VIX is above a chosen threshold.

> Compare results before and after the current macro regime.

> Check whether the strategy still works when entries wait for bar close.

This is the difference between code generation and strategy research.

Code generation says:

> Here is your Pine Script.

Strategy research says:

> Here is your Pine Script, and here are the assumptions that can break it.

The second version is more valuable.

The current financial environment is a good reminder that the market is not one machine. It is a stack of machines: rates, inflation, liquidity, sector leadership, positioning, volatility, and news. A Pine Script strategy cannot fully understand all of them. But it can stop pretending they do not exist.

That is the practical goal.

A good Pine Script strategy does not need to predict every macro event. It should know when its own signal is weak, when its cost assumptions are fragile, and when its backtest may be lying.

In quiet markets, simple scripts can look smart.

In difficult markets, assumptions become visible.

This is why the next generation of AI trading agents should not only write Pine Script faster. They should make hidden assumptions explicit: regime, cost, fill, repainting, alerts, and risk.

A trading idea should become code. But in 2026, code should also know what kind of market it is standing in.

## References

- [Federal Reserve, FOMC Statement, April 29, 2026](https://www.federalreserve.gov/newsevents/pressreleases/monetary20260429a.htm)
- [U.S. Bureau of Economic Analysis, Personal Income and Outlays, April 2026](https://www.bea.gov/news/2026/personal-income-and-outlays-april-2026)
- [U.S. Bureau of Labor Statistics, Employment Situation Summary, May 2026](https://www.bls.gov/news.release/empsit.nr0.htm)
- [FRED, CBOE Volatility Index: VIX](https://fred.stlouisfed.org/series/VIXCLS)
- [TradingView Pine Script Strategies](https://www.tradingview.com/pine-script-docs/concepts/strategies/)
- [TradingView Strategy Properties](https://www.tradingview.com/support/solutions/43000628599-strategy-properties/)
- [TradingView Pine Script Repainting](https://www.tradingview.com/pine-script-docs/concepts/repainting/)
- [TradingView Other Timeframes and Data](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/)
- [TradingView Economic Data in Pine](https://www.tradingview.com/support/solutions/43000665359-what-economic-data-is-available-in-pine/)
- [TradingView Pine Script Alerts FAQ](https://www.tradingview.com/pine-script-docs/faq/alerts/)
