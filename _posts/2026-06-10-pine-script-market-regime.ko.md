---
layout: post
title: "Pine Script에는 신호보다 시장 국면이 먼저 필요하다"
date: 2026-06-10 10:00:00 +0900
lang: ko
ref: pine-script-market-regime-2026-06-10
permalink: /ko/2026/06/10/pine-script-market-regime/
categories: [pine-script, ai, trading, market]
---

매매 신호는 혼자 존재하지 않는다.

신호는 언제나 시장 국면 안에 있다. 유동성이 차분한 시기의 이동평균선 교차와 CPI 발표 주간의 이동평균선 교차는 같은 사건이 아니다. 저금리 강세장에서의 돌파와 인플레이션이 끈적하고, 금리가 높고, 헤드라인 하나에 변동성이 뛰는 시장에서의 돌파도 같은 사건이 아니다.

그래서 Pine Script는 진입 조건만 설명해서는 부족하다. 그 진입 조건이 의미를 가져도 되는 환경까지 설명해야 한다.

2026년 6월 초 현재, 시장은 단순한 이야기가 아니다. 연방준비제도는 4월 회의에서 연방기금금리 목표 범위를 3.50%에서 3.75%로 유지했고, 경제 전망을 둘러싼 불확실성이 높은 수준이라고 밝혔다. 4월 PCE 가격지수는 전년 대비 3.8%, 근원 PCE는 3.3% 상승했다. 5월 고용보고서는 비농업 신규 고용 17만 2천 명과 실업률 4.3%를 보여 주었다. 한편 VIX는 6월 4일 15.40에서 6월 5일 21.51로 올랐다가, 6월 8일 18.92로 내려왔다.

이 조합은 전략 설계에서 중요하다.

시장이 여전히 위험 감수를 보상할 수 있지만, 느슨한 가정은 벌할 수 있다는 뜻이다. 추세추종 아이디어가 여전히 작동할 수 있지만, 틀렸을 때의 비용은 빠르게 바뀔 수 있다는 뜻이다. 변동성, 슬리피지, 포지션 크기, 이벤트 리스크를 무시한 백테스트는 실제 시장보다 더 깔끔해 보일 수 있다는 뜻이다.

이런 시장을 위한 Pine Script 전략은 다른 질문을 해야 한다.

단지 이렇게 묻는 것이 아니다.

> 언제 매수해야 하는가?

다음 질문도 함께 해야 한다.

> 이 매수 신호는 어떤 시장 국면에서만 존재해도 되는가?

차이는 작아 보인다. 하지만 코드에서는 크다.

단순한 돌파 규칙은 이렇게 말할 수 있다.

> 가격이 최근 20개 봉의 최고가를 종가 기준으로 돌파하면 매수한다.

이것은 신호다. 하지만 국면을 의식하는 버전은 더 많은 것을 묻는다.

- 변동성이 너무 빠르게 확대되고 있는가?
- 지수는 장기 추세 위에 있는가, 아래에 있는가?
- 해당 종목은 넓은 시장과 함께 움직이는가, 반대로 움직이는가?
- 손절 거리는 ATR 대비 합리적인가?
- 변동성이 높을 때 전략은 포지션 크기를 줄여야 하는가?
- 주요 거시 이벤트 구간에는 진입을 막아야 하는가?
- 알림은 봉 마감까지 기다려야 하는가?

이 질문들은 장식이 아니다. 전략의 일부다.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/regime-filter.png" alt="매매 신호가 실행 전에 시장 국면 필터를 통과하는 모습을 보여 주는 리서치 이미지" loading="lazy" decoding="async">
  <figcaption>신호는 거래가 되기 전에 시장 국면 필터를 통과할 때 더 쓸모 있어진다.</figcaption>
</figure>

Pine Script를 이해하는 유용한 방식은 이것이다.

> 진입 로직은 기회를 설명한다. 국면 로직은 허가를 설명한다.

전략은 많은 기회를 찾을 수 있다. 하지만 모든 기회를 취할 필요는 없다.

변동성이 큰 환경에서 돌파는 강함을 뜻할 수 있다. 동시에 소진을 뜻할 수도 있다. 저항선 위의 종가는 실제 매수세일 수도 있고, 일시적인 유동성이 만든 스톱 사냥일 수도 있다. 코드는 미래를 알 수 없다. 하지만 모든 돌파를 같은 대상으로 취급하지 않을 수는 있다.

예를 들어 트레이더는 변동성 필터를 추가할 수 있다.

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

이 코드는 완성된 매매 시스템이 아니다. 설계 태도의 스케치다.

중요한 것은 정확한 VIX 수준도, 정확한 ATR 백분위도 아니다. 중요한 것은 전략이 시장 날씨라는 개념을 가진다는 점이다. 차분한 시장에서는 더 촘촘한 돌파를 받아들일 수 있다. 스트레스를 받는 시장에서는 더 큰 버퍼를 요구하거나, 포지션 크기를 줄이거나, 아예 거래를 멈출 수 있다.

많은 트레이더는 이런 조정을 수동으로 한다. 차트를 보고 "오늘은 너무 위험해 보인다"고 말한다. 하지만 이 규칙이 트레이더의 머릿속에만 남아 있으면 정직하게 백테스트할 수 없다. Pine Script는 그 모호한 문장을 눈에 보이게 만든다.

또 하나의 미묘한 지점은 시장 국면이 방향만의 문제가 아니라는 것이다.

시장은 강세이면서 동시에 위험할 수 있다. 주식은 200일 이동평균선 위에 있으면서 스프레드는 넓어질 수 있다. 지수는 고점 근처에 있는데 주도주는 좁아질 수 있다. 전략은 전체로는 수익이 나지만 특정 세 종류의 주간에만 취약할 수 있다.

그래서 진지한 Pine Script 작업은 적어도 네 개의 층을 분리해야 한다.

첫 번째 층은 방향이다. 자산은 상승 추세인가, 하락 추세인가, 횡보인가?

두 번째 층은 변동성이다. 현재 가격 범위는 정상인가, 압축되어 있는가, 확대되어 있는가?

세 번째 층은 유동성과 비용이다. 손절 주문과 시장가 주문이 백테스트 가정보다 더 큰 슬리피지를 겪을 가능성이 있는가?

네 번째 층은 실행 타이밍이다. 전략은 봉 마감에 행동하는가, 다음 봉 시가에 행동하는가, 봉 안에서 행동하는가, 아니면 낮은 시간대 확인 이후 행동하는가?

대부분의 초보 스크립트는 이 네 층을 하나의 신호로 뭉갠다. "RSI가 30을 상향 돌파했다"거나 "가격이 이동평균선을 돌파했다"고 말한다. 지표에는 충분할 수 있다. 전략에는 거의 충분하지 않다.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/cost-assumptions.png" alt="슬리피지, 수수료, 스프레드, 변동성이 백테스트 안의 숨은 비용으로 작동하는 모습을 보여 주는 리서치 이미지" loading="lazy" decoding="async">
  <figcaption>변동성이 큰 시장에서 비용 가정은 작은 세부 사항이 아니다. 전략과 착시를 가르는 차이가 될 수 있다.</figcaption>
</figure>

현재 시장에서는 비용 가정이 특히 중요하다.

TradingView 전략은 broker emulator를 통해 실행된다. 이 에뮬레이터는 유용하지만 여전히 시뮬레이션이다. 기본적으로 차트 데이터와 봉 안에서 가격이 어떻게 움직였는지에 대한 가정을 사용한다. TradingView의 Bar Magnifier는 더 낮은 시간대 데이터를 사용해 과거 체결을 더 정교하게 만들 수 있지만, 그것도 백테스트를 실제 주문장으로 바꾸지는 않는다.

즉 수수료와 슬리피지가 0인 상태에서 훌륭해 보이는 전략은 대부분 비용 착시일 수 있다.

Pine Script는 그 착시를 줄일 도구를 준다. `strategy()` 선언에서 수수료, 슬리피지, 증거금, 주문 크기, 피라미딩, 체결 동작을 설정할 수 있다. 이것들은 지루한 설정이 아니다. 전략 논리의 일부다.

예를 들면 다음과 같다.

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

이것은 Strategy Tester 패널에 모든 가정을 숨기는 전략보다 더 정직한 이야기를 한다.

AI가 생성한 Pine Script의 위험한 부분은 이런 가정을 건너뛰면서도 그럴듯한 코드를 만들 수 있다는 점이다. 반대로 AI가 생성한 Pine Script의 쓸모 있는 부분은 에이전트가 매번 이런 가정을 물어보도록 훈련될 수 있다는 점이다.

전략을 생성하기 전에 에이전트는 물어야 한다.

> 어떤 수수료와 슬리피지를 가정해야 하는가?

> 피라미딩은 허용되는가?

> 포지션 크기는 고정형인가, 변동성 조정형인가, 자기자본 기반인가?

> 고변동성 국면에서는 전략이 진입을 막아야 하는가?

> 진입은 봉 마감 확인을 기다려야 하는가?

> 상위 시간대 데이터는 리페인팅 없이 처리되어야 하는가?

> 이 전략은 백테스트 전용인가, 알림용인가, 외부 시스템을 통한 실거래 실행용인가?

이 질문들은 첫 초안을 느리게 만든다. 하지만 두 번째 초안을 훨씬 더 쓸모 있게 만든다.

헤드라인에 흔들리는 시장에서는 리페인팅도 더 중요해진다. 가격이 캔들 안에서 급하게 움직이면 봉 안에서 반응하고 싶은 유혹이 생긴다. 하지만 과거 봉과 실시간 봉은 모든 스크립트에서 같은 방식으로 행동하지 않는다. TradingView 문서는 `request.security()`와 상위 시간대 데이터가 조심스럽게 처리되지 않으면 리페인팅 문제를 만들 수 있다고 반복해서 경고한다.

이것은 단순한 기술적 경고가 아니다. 전략적 경고다.

과거 데이터에서 모든 거시 이벤트 반전을 잡아내는 것처럼 보이는 스크립트는 실제로 그 시점에는 알 수 없었던 정보를 쓰고 있을 수 있다. 캔들의 고점이나 저점에 정확히 나타나는 신호는 캔들이 닫히기 전에 확정되지 않았을 수 있다. 상위 시간대 추세 필터는 과거에는 안정적으로 보이지만 실시간에서는 흔들릴 수 있다.

이런 환경에서 더 안전한 기본값은 종종 지루하다.

    confirmedSignal = barstate.isconfirmed and rawSignal

이 한 줄이 전략을 수익성 있게 만들지는 않는다. 다만 테스트를 덜 마법처럼 만든다.

알림도 마찬가지다.

차분한 시장에서 나쁜 알림은 귀찮은 일이다. 변동성이 큰 시장에서 나쁜 알림은 비싼 일이 될 수 있다. Pine Script 알림이 "BUY"나 "SELL"만 보낸다면 다음 실행 계층에 필요한 맥락이 빠져 있다.

더 쓸모 있는 알림 메시지는 국면, 가격, 손절, 리스크 크기, 거래 이유를 포함한다. 목표는 행동을 트리거하는 것만이 아니다. 나중에 그 행동을 감사할 수 있게 만드는 것이다.

예를 들어 알림 페이로드는 다음을 포함할 수 있다.

    {
      "symbol": "{{ticker}}",
      "side": "buy",
      "reason": "breakout_with_trend_filter",
      "regime": "normal_volatility",
      "risk_model": "atr_stop",
      "timeframe": "{{interval}}"
    }

이것이 중요한 이유는 Pine Script가 보통 최종 실행 계층이 아니기 때문이다. 전략과 지표는 알림을 만들 수 있지만, 실거래 시스템은 대개 웹훅, 외부 서버, 브로커 API, 리스크 통제를 거쳐 이어진다. 알림 메시지가 모호하면 전체 자동화 체인이 취약해진다.

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/alert-context.png" alt="Pine Script 알림이 국면, 리스크, 실행 맥락을 외부 거래 시스템으로 전달하는 모습을 보여 주는 리서치 이미지" loading="lazy" decoding="async">
  <figcaption>알림은 단순히 매수나 매도를 말하는 데 그쳐서는 안 된다. 리스크 통제와 사후 검토에 필요한 맥락을 함께 전달해야 한다.</figcaption>
</figure>

현재 시장에는 또 하나의 조용한 문제가 있다. 전략의 반감기다.

시장이 AI 인프라, 에너지 충격, 금리 기대, 지정학적 헤드라인처럼 하나의 테마에 지배될 때, 어떤 전략은 짧은 기간 동안 강력해 보일 수 있다. 모멘텀 규칙이 작동하는 이유가 구조적으로 강해서가 아니라, 하나의 테마가 일시적으로 모든 것을 압도하고 있기 때문일 수 있다.

바로 이 지점에서 Pine Script는 트레이더가 최근 성과를 의심하도록 도와야 한다.

좋은 스크립트는 순이익만 보여 줘서는 안 된다. 다음과 같은 질문을 유도해야 한다.

- 이 전략은 현재 시장 테마 이전에도 작동했는가?
- 낮은 변동성과 높은 변동성 구간을 모두 견뎠는가?
- 최고의 거래가 하나의 짧은 군집에서만 나왔는가?
- 성과가 하나의 자산, 하나의 섹터, 하나의 달에 의존했는가?
- 현실적인 슬리피지와 수수료 이후에도 돈을 벌었는가?
- 파라미터를 조금 바꿔도 안정적으로 남았는가?

AI는 여기서 도움이 될 수 있다. 하지만 에이전트가 제대로 설계된 경우에만 그렇다. 에이전트는 높은 백테스트 수익률을 너무 빨리 칭찬해서는 안 된다. 견고성 점검을 요구해야 한다.

예를 들어 Pine Script를 생성한 뒤 에이전트는 이렇게 제안할 수 있다.

> 같은 로직을 ATR 길이 10, 14, 21로 테스트해 보라.

> 돌파 길이 20, 30, 50을 테스트해 보라.

> 슬리피지를 두 배로 늘려 전략을 실행해 보라.

> VIX가 선택한 임계값보다 높을 때 롱 진입을 비활성화해 보라.

> 현재 거시 국면 전후의 결과를 비교해 보라.

> 진입이 봉 마감을 기다려도 전략이 작동하는지 확인해 보라.

이것이 코드 생성과 전략 리서치의 차이다.

코드 생성은 이렇게 말한다.

> 여기 Pine Script가 있다.

전략 리서치는 이렇게 말한다.

> 여기 Pine Script가 있고, 이것을 깨뜨릴 수 있는 가정들이 있다.

두 번째 버전이 더 가치 있다.

현재 금융 환경은 시장이 하나의 기계가 아니라는 사실을 잘 보여 준다. 시장은 금리, 인플레이션, 유동성, 섹터 리더십, 포지셔닝, 변동성, 뉴스가 쌓인 기계들의 묶음이다. Pine Script 전략이 이 모든 것을 완전히 이해할 수는 없다. 하지만 그것들이 존재하지 않는 척하는 일은 멈출 수 있다.

그것이 실용적인 목표다.

좋은 Pine Script 전략은 모든 거시 이벤트를 예측할 필요가 없다. 자기 신호가 약해지는 때, 비용 가정이 취약해지는 때, 백테스트가 거짓말을 할 수 있는 때를 알아야 한다.

조용한 시장에서는 단순한 스크립트도 똑똑해 보일 수 있다.

어려운 시장에서는 가정이 드러난다.

그래서 다음 세대의 AI 트레이딩 에이전트는 Pine Script를 더 빨리 쓰는 데 그쳐서는 안 된다. 국면, 비용, 체결, 리페인팅, 알림, 리스크라는 숨은 가정을 명시적으로 만들어야 한다.

매매 아이디어는 코드가 되어야 한다. 하지만 2026년의 코드는 자신이 어떤 시장에 서 있는지도 알아야 한다.

## 참고한 문서

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
