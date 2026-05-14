---
layout: post
title: "好的交易想法应该变成 Pine Script"
date: 2026-05-14 10:00:00 +0900
lang: zh
ref: trading-idea-to-pine-script-2026-05-14
permalink: /zh/2026/05/14/trading-idea-to-pine-script/
categories: [ai, chart, market]
---

一个好的交易想法，什么时候才算真的好？

不是它在图表上看起来合理的时候。也不是它连续对了几次的时候。一个好的交易想法，只有在它变成规则，变成代码，并且能用同样的方式在历史数据和实时条件下被测试时，才开始接近真正的好。

所以对 TradingView 用户来说，Pine Script 不只是一个脚本语言。它是一个小型实验室，用来把想法变成可以测试的东西。

很多交易者其实已经在半个策略师的状态下工作。当均线以某个角度发散，当 RSI 从某个区域反转，当成交量放大到超过近期区间时，人会在脑子里写出条件判断。问题在于，如果这些判断只停留在语言里，就很容易失真。记忆会更愿意保留那些看起来有效的画面，也更快忘掉那些失败的画面。

代码做的是相反的事。代码会数你不喜欢的区间，会数那些模糊的入场，也会数止损后马上反弹的情况。这会让人不舒服。但在交易里，这种不舒服很有用。

这里有一个重要区别。图表分析其实混合了两件事。一件事是看见模式。另一件事是确认这个模式能不能重复。人擅长第一件事。代理更适合承担第二件事。

代理在交易里的工作，不是“猜哪个品种会上涨”。这个定义太弱了。更重要的工作，是把用户的假设变成 Pine Script 策略，再把回测、警报条件和风险规则连成一个工作流。

比如有这样一个想法：

> 只有当收盘价保持在20日均线之上，并且成交量高于近期平均值时，才买入突破。

这句话听起来像一个策略。但实际上，它缺了很多东西。“在均线之上”是按收盘价算，还是按盘中价格算？成交量平均值用多少根K线？突破是突破前高，还是突破箱体上沿？止损放在哪里？止盈是固定比例，还是趋势退出？同一根K线里能不能同时入场和出场？

这些问题让人觉得麻烦，是很自然的。但策略的大部分内容就藏在这些麻烦的问题里。想法通常从一句话开始。策略则由几十个小决定组成。

<figure class="article-figure">
  <img src="/assets/images/posts/trading-idea-to-pine-script/strategy-rules.png" alt="图表想法被拆解成策略条件和代码结构的研究图片" loading="lazy" decoding="async">
  <figcaption>好的想法可以从一句话开始，但可验证的策略只有在条件和例外都被写清楚时才会出现。</figcaption>
</figure>

Pine Script 之所以重要，是因为它会把这些小决定暴露出来。你必须决定在哪里调用 `strategy.entry()`，必须决定如何设置止损和止盈，也必须决定警报什么时候触发。用语言说，它可能只是一个“突破策略”。写成代码，它会分裂成许多不同的策略。

现在可以看到代理的位置了。代理不应该把用户的第一句话当成已经完成的策略。相反，它应该找出那句话里面的空白。好的代理与其说是快速写代码的工具，不如说是不断追问缺失条件的工具。

在 TradingView 里，尤其要注意三件事。

第一是重绘。这个问题不能只问“会不会重绘”。TradingView 官方文档也说明，重绘有许多原因和形式。更好的问题应该更具体。脚本在历史K线和实时K线上是否用同样的方式计算？警报是否等待K线收盘后才触发？`request.security()` 是否把未来信息泄漏到了过去数据里？

第二是成交假设。回测不是现实中的订单簿。TradingView 策略通过 broker emulator 计算理论成交。Bar Magnifier 这样的功能可以使用更低时间周期的数据，让成交假设更细，但它仍然不能完全替代真实的滑点和流动性。

第三是执行路径。Pine Script 策略和指标并不是直接向交易所下单的系统。根据 TradingView 官方文档，策略和指标不能通过 TradingView 的券商连接或内置模拟交易账户直接下单。真实交易的自动化通常需要用警报、Webhook 和外部执行系统来设计成单独的一层。

这三点不是为了吓退新手。相反，它们是构建专业交易系统的最低地面。策略的收益率数字，只有站在这块地面上才有意义。

<figure class="article-figure">
  <img src="/assets/images/posts/trading-idea-to-pine-script/backtest-validation.png" alt="用放大镜检查回测收益和回撤区间的研究图片" loading="lazy" decoding="async">
  <figcaption>回测数字不是结论，而是检查对象。尤其要通过回撤和成交假设去看它。</figcaption>
</figure>

AI 代理的发展也和这里相连。最近金融行业关于 AI 的讨论，正在从简单聊天机器人转向更实际的工作流。比起总结报告的工具，更重要的是能在多个系统之间计划并执行任务的代理。在金融领域，风险管理、合规、投资策略优化这类同时包含规则和例外的工作，天然适合代理系统。

个人交易者也很可能经历同样的变化。过去，找到一个好指标就是优势。现在，更重要的优势是你能多快把一个假设变成代码，能多诚实地检验它，又能多快把它改好。

我把这个趋势看作“从读图到策略编译”的移动。编译这个词听起来有点技术化，但意思很简单。它就是把脑子里的交易想法，变成可以执行的规则。就像文字可以被编译成代码，交易想法也应该被编译成 Pine Script。

在这个过程中，代理会反复做四件事。

<figure class="article-figure">
  <img src="/assets/images/posts/trading-idea-to-pine-script/execution-risk.png" alt="图表信号经过警报和外部执行层的研究图片" loading="lazy" decoding="async">
  <figcaption>Pine Script 信号和真实订单执行是不同层。严肃的自动化必须把这条路径单独设计清楚。</figcaption>
</figure>

1. 把用户的交易想法拆成明确条件。
2. 把这些条件写成 Pine Script 指标或策略。
3. 检查重绘、成交假设、警报条件和仓位管理里的空白。
4. 根据回测结果提出下一次实验。

这个循环重要，并不是因为它能立刻保证收益。恰恰相反。很多策略经过这个循环后，会暴露出自己很弱。这是好事。快速丢掉弱策略的能力，本来就是找到强策略能力的一部分。

这个博客之后会研究这个循环。它不会只讲 Pine Script 语法。也不会只罗列图表形态。它会讨论代理如何结构化交易想法，如何生成策略代码，以及我们应该如何怀疑回测数字。

交易里最危险的一句话是“感觉这个有效”。第二危险的一句话是“回测收益率很高”。这两句话都还不够。

好的交易想法应该变成代码。好的代码也应该再次被怀疑。让这个反复过程更快、更严格，是代理在图表之上能做的最现实的事。

## 参考资料

- [TradingView Pine Script User Manual](https://www.tradingview.com/pine-script-docs/)
- [TradingView Pine Script Strategies FAQ](https://www.tradingview.com/pine-script-docs/faq/strategies/)
- [TradingView Repainting documentation](https://www.tradingview.com/pine-script-docs/v5/concepts/repainting/)
- [TradingView Pine Script v6 announcement](https://www.tradingview.com/blog/en/pine-script-v6-has-landed-48830/)
- [NVIDIA State of AI in Financial Services 2026 Trends](https://blogs.nvidia.com/blog/ai-in-financial-services-survey-2026/)
- [McKinsey, Seizing the agentic AI advantage](https://www.mckinsey.com/capabilities/quantumblack/our-insights/seizing-the-agentic-ai-advantage)
