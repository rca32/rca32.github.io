---
layout: post
title: "Pine Script 需要市场状态，而不只是信号"
date: 2026-06-10 10:00:00 +0900
lang: zh
ref: pine-script-market-regime-2026-06-10
permalink: /zh/2026/06/10/pine-script-market-regime/
categories: [pine-script, ai, trading, market]
---

交易信号从来不是孤立存在的。

它总是活在某种市场状态之中。流动性平稳时的均线交叉，和 CPI 周的均线交叉，不是同一件事。低利率牛市里的突破，和通胀黏性较强、利率较高、波动率会被新闻标题瞬间推高的市场里的突破，也不是同一件事。

因此，Pine Script 不应该只描述入场。它还应该描述这些入场条件在哪种环境里才被允许有意义。

截至 2026 年 6 月上旬，市场不是一个简单故事。美联储在 4 月会议上将联邦基金利率目标区间维持在 3.50% 至 3.75%，并表示经济前景的不确定性仍处于高位。4 月 PCE 价格指数同比上涨 3.8%，核心 PCE 同比上涨 3.3%。5 月就业报告仍显示非农就业新增 17.2 万人，失业率为 4.3%。与此同时，VIX 从 6 月 4 日的 15.40 升至 6 月 5 日的 21.51，随后在 6 月 8 日回落到 18.92。

这个组合对策略设计很重要。

它说明市场仍可能奖励风险承担，但也可能惩罚懒散的假设。它说明趋势跟随想法仍可能有效，但错误的代价可能迅速变化。它说明忽略波动率、滑点、仓位规模和事件风险的回测，可能看起来比真实市场更干净。

为这种市场编写的 Pine Script 策略，应该提出不同的问题。

不只是：

> 我应该什么时候买入？

还应该问：

> 这个买入信号被允许存在于哪种市场状态之中？

这个差别看起来很小。在代码里，它很大。

一个简单的突破规则可能会说：

> 当价格收盘高于过去 20 根 K 线的最高价时买入。

这是信号。但一个有市场状态意识的版本会问更多问题。

- 波动率是否扩张得太快？
- 指数位于长期趋势之上还是之下？
- 这个标的是跟随大盘，还是逆着大盘？
- 止损距离相对于 ATR 是否合理？
- 波动率高时，策略是否应该降低仓位？
- 重大宏观事件窗口中是否应该禁止入场？
- 警报是否应该等待 K 线收盘？

这些问题不是装饰。它们是策略的一部分。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/regime-filter.png" alt="展示交易信号在执行前穿过市场状态过滤器的研究图片" loading="lazy" decoding="async">
  <figcaption>信号在成为交易之前经过市场状态过滤器，会变得更有用。</figcaption>
</figure>

理解 Pine Script 的一个有用方式是：

> 入场逻辑解释机会。市场状态逻辑解释许可。

一个策略可能发现许多机会。但它不需要全部参与。

在高波动环境里，突破可能意味着强势。它也可能意味着耗尽。阻力位上方的收盘可能是真实需求，也可能是临时流动性制造的扫止损。代码无法知道未来，但它可以避免把每一次突破都当作同一种对象。

例如，交易者可以加入一个波动率过滤器。

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

这不是一个完整交易系统。它只是某种设计态度的草图。

重要的不是具体的 VIX 水平，也不是具体的 ATR 百分位。重要的是策略拥有“市场天气”的概念。在平静市场中，脚本可以接受更紧的突破。在压力市场中，它可能要求更大的缓冲、降低仓位，或者完全停止交易。

许多交易者会手动做这种调整。他们看着图表说：“今天感觉太危险。”但如果这条规则只留在交易者脑中，就无法被诚实地回测。Pine Script 让这个模糊句子变得可见。

另一个细微问题是，市场状态不只是方向问题。

市场可以同时看涨又危险。一只股票可以站在 200 日均线之上，同时点差却在扩大。指数可以接近高位，同时领涨范围却在变窄。一个策略总体上可能盈利，但在三类特定周里很脆弱。

因此，严肃的 Pine Script 工作至少应该分开四个层次。

第一层是方向。资产是在上涨、下跌，还是横盘？

第二层是波动率。当前区间是正常、压缩，还是扩张？

第三层是流动性和成本。止损单和市价单是否可能遭遇比回测假设更大的滑点？

第四层是执行时点。策略是在 K 线收盘、下一根开盘、K 线内部，还是在低周期确认之后行动？

大多数初学者脚本会把这四层压缩成一个信号。它们说“RSI 上穿 30”或“价格上穿均线”。这对指标可能足够。对策略而言，通常不够。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/cost-assumptions.png" alt="展示滑点、佣金、价差和波动率作为回测隐藏成本的研究图片" loading="lazy" decoding="async">
  <figcaption>在波动市场中，成本假设不是小细节。它们可能决定一个结果是策略，还是幻觉。</figcaption>
</figure>

当前市场让成本假设尤其重要。

TradingView 策略通过 broker emulator 运行。这个模拟器很有用，但它仍然是模拟。默认情况下，它使用图表数据，以及关于价格如何在一根 K 线内部移动的假设。TradingView 的 Bar Magnifier 可以使用低周期数据，让历史成交更精细，但它仍然不会把回测变成真实订单簿。

这意味着，一个在零佣金、零滑点条件下看起来极好的策略，可能主要是成本幻觉。

Pine Script 提供了让这种幻觉变小的工具。在 `strategy()` 声明中，我们可以设置佣金、滑点、保证金、订单规模、加仓和成交行为。这些不是无聊设置。它们是论点的一部分。

例如：

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

这比把所有假设都藏在 Strategy Tester 面板里的策略更诚实。

AI 生成 Pine Script 的危险之处在于，它可以跳过这些假设，却生成看起来很厉害的代码。AI 生成 Pine Script 的有用之处在于，代理可以被训练成每次都询问这些假设。

在生成策略之前，代理应该问：

> 应该假设多少佣金和滑点？

> 是否允许加仓？

> 仓位规模应该固定、按波动率调整，还是按权益比例计算？

> 策略是否应在高波动状态中禁用入场？

> 入场是否应等待 K 线收盘确认？

> 高周期数据是否需要避免重绘？

> 这个策略是仅用于回测、用于警报，还是用于通过外部系统实盘执行？

这些问题会拖慢第一稿。但它们会让第二稿更有用。

在由新闻标题驱动的市场里，重绘也变得更重要。当价格在一根 K 线内部快速移动时，人很容易想要盘中反应。但历史 K 线和实时 K 线并不总是在每个脚本里表现相同。TradingView 文档反复提醒，如果 `request.security()` 和高周期数据处理不当，就可能产生重绘问题。

这不只是技术警告。它也是策略警告。

一个在历史数据上看起来抓住了每次宏观驱动反转的脚本，可能使用了当时实际上不可获得的信息。一个看起来正好出现在 K 线最高点或最低点的信号，在 K 线收盘前可能并未确认。高周期趋势过滤器在历史上看起来稳定，但在实时中可能波动。

在这种环境中，更安全的默认值往往很无聊。

    confirmedSignal = barstate.isconfirmed and rawSignal

这一行不会让策略盈利。它只是让测试少一点魔法。

警报也是如此。

在平静市场中，坏警报只是烦人。在波动市场中，坏警报可能很昂贵。如果 Pine Script 警报只发送 "BUY" 或 "SELL"，它就缺少下一层执行所需要的上下文。

更有用的警报消息应包含市场状态、价格、止损、风险规模和交易理由。目标不只是触发行动。目标还包括让之后的审计成为可能。

例如，一个警报 payload 可以包含：

    {
      "symbol": "{{ticker}}",
      "side": "buy",
      "reason": "breakout_with_trend_filter",
      "regime": "normal_volatility",
      "risk_model": "atr_stop",
      "timeframe": "{{interval}}"
    }

这一点重要，是因为 Pine Script 通常不是最终执行层。策略和指标可以创建警报，但实盘交易系统通常还会继续经过 webhook、外部服务器、券商 API 和风险控制。如果警报消息含糊，整条自动化链路都会变得脆弱。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/alert-context.png" alt="展示 Pine Script 警报把市场状态、风险和执行上下文传递给外部交易系统的研究图片" loading="lazy" decoding="async">
  <figcaption>警报不应只说买或卖。它应该携带足够的上下文，以便风险控制和事后复盘。</figcaption>
</figure>

当前市场还有另一个安静的问题：策略半衰期。

当市场被单一主题主导，例如 AI 基础设施、能源冲击、利率预期或地缘政治新闻时，一些策略可能在短窗口里显得强大。动量规则之所以有效，可能不是因为它结构性很强，而是因为一个主题暂时压倒了其他一切。

这正是 Pine Script 应该帮助交易者怀疑近期表现的地方。

一个好脚本不应该只显示净利润。它应该鼓励提出这些问题：

- 策略在当前市场主题出现之前是否有效？
- 它是否经历过低波动和高波动时期？
- 最好的交易是否来自一个很短的集群？
- 业绩是否依赖单一资产、单一板块或单一月份？
- 在现实滑点和佣金之后，策略是否仍然赚钱？
- 参数稍微变化后，它是否仍然稳定？

AI 可以在这里提供帮助，但前提是代理被正确设计。代理不应该过快庆祝高回测收益。它应该要求稳健性检查。

例如，在生成 Pine Script 之后，代理可以建议：

> 用 ATR 长度 10、14、21 测试同一逻辑。

> 用突破长度 20、30、50 测试。

> 将滑点加倍后运行策略。

> 当 VIX 高于选定阈值时禁用多头入场。

> 比较当前宏观状态之前和之后的结果。

> 检查入场等待 K 线收盘时策略是否仍然有效。

这就是代码生成和策略研究的区别。

代码生成说：

> 这是你的 Pine Script。

策略研究说：

> 这是你的 Pine Script，以及可能让它失效的假设。

第二个版本更有价值。

当前金融环境提醒我们，市场不是一台机器。它是一组机器的堆叠：利率、通胀、流动性、板块领导力、仓位、波动率和新闻。Pine Script 策略无法完全理解它们。但它可以停止假装它们不存在。

这就是实际目标。

一个好的 Pine Script 策略不需要预测每个宏观事件。它应该知道自己的信号什么时候变弱，成本假设什么时候变脆弱，回测什么时候可能在说谎。

在安静市场中，简单脚本也会显得聪明。

在困难市场中，假设会变得可见。

因此，下一代 AI 交易代理不应该只是更快地编写 Pine Script。它们应该明确隐藏假设：市场状态、成本、成交、重绘、警报和风险。

交易想法应该变成代码。但在 2026 年，代码也应该知道自己站在什么样的市场里。

## 参考资料

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
