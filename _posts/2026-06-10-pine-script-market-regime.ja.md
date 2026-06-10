---
layout: post
title: "Pine Scriptに必要なのはシグナルだけでなく市場レジームだ"
date: 2026-06-10 10:00:00 +0900
lang: ja
ref: pine-script-market-regime-2026-06-10
permalink: /ja/2026/06/10/pine-script-market-regime/
categories: [pine-script, ai, trading, market]
---

売買シグナルは単独では存在しない。

シグナルは常に市場レジームの中にある。流動性が落ち着いている時期の移動平均クロスと、CPI発表週の移動平均クロスは同じ出来事ではない。低金利の強気相場で起きるブレイクアウトと、インフレが粘り強く、金利が高く、ヘッドラインでボラティリティが跳ねる市場で起きるブレイクアウトも同じではない。

だから Pine Script はエントリーだけを記述すればよいわけではない。そのエントリーが意味を持ってよい環境も記述する必要がある。

2026年6月上旬時点で、市場は単純な物語ではない。連邦準備制度は4月会合でフェデラルファンド金利の誘導目標レンジを3.50%から3.75%に据え置き、経済見通しをめぐる不確実性は高い水準にあると述べた。4月のPCE価格指数は前年同月比3.8%上昇し、コアPCEは3.3%上昇した。5月の雇用統計では非農業部門雇用者数が17万2千人増え、失業率は4.3%だった。一方、VIXは6月4日の15.40から6月5日に21.51へ上昇し、6月8日には18.92へ戻った。

この組み合わせは戦略設計にとって重要だ。

市場はまだリスクテイクに報いる可能性がある。しかし、粗い仮定を罰する可能性もある。トレンドフォローの考え方はまだ機能しうる。しかし、間違えたときの代償は急に変わりうる。ボラティリティ、スリッページ、ポジションサイズ、イベントリスクを無視したバックテストは、実際の市場よりきれいに見えるかもしれない。

このような市場のために書かれる Pine Script 戦略は、別の問いを立てるべきだ。

単にこう問うだけではない。

> いつ買うべきか。

同時にこう問う必要がある。

> この買いシグナルは、どの市場レジームでのみ存在を許されるのか。

この違いは小さく見える。だがコードでは大きい。

単純なブレイクアウトルールはこう言える。

> 価格が直近20本の最高値を終値で上抜いたら買う。

これはシグナルだ。しかし、レジームを意識するバージョンはさらに多くを問う。

- ボラティリティは速すぎるペースで拡大していないか。
- 指数は長期トレンドの上にあるか、下にあるか。
- 銘柄は広い市場と同じ方向に動いているか、逆行しているか。
- ストップ幅はATRに対して妥当か。
- ボラティリティが高いとき、戦略はサイズを落とすべきか。
- 主要なマクロイベントの時間帯にはエントリーを止めるべきか。
- アラートはバー確定まで待つべきか。

これらの問いは飾りではない。戦略の一部だ。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/regime-filter.png" alt="売買シグナルが実行前に市場レジームのフィルターを通過する様子を示すリサーチ画像" loading="lazy" decoding="async">
  <figcaption>シグナルは取引になる前に市場レジームのフィルターを通ることで、より有用になる。</figcaption>
</figure>

Pine Script を考えるうえで有用な見方はこれだ。

> エントリーロジックは機会を説明する。レジームロジックは許可を説明する。

戦略は多くの機会を見つけることができる。だが、そのすべてを取る必要はない。

高ボラティリティ環境でのブレイクアウトは強さを意味することがある。同時に、消耗を意味することもある。抵抗線を上回る終値は本物の需要かもしれないし、一時的な流動性が作ったストップ狩りかもしれない。コードは未来を知らない。だが、すべてのブレイクアウトを同じものとして扱わないことはできる。

たとえば、トレーダーはボラティリティフィルターを追加できる。

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

これは完成した売買システムではない。設計姿勢のスケッチだ。

重要なのは正確なVIX水準でも、正確なATRパーセンタイルでもない。重要なのは、戦略が市場の天候という概念を持つことだ。穏やかな市場では、スクリプトはより狭いブレイクアウトを受け入れるかもしれない。緊張した市場では、より大きなバッファを求め、サイズを落とし、あるいは完全に取引を止めるかもしれない。

多くのトレーダーはこの調整を手動で行う。チャートを見て「今日は危険すぎる」と言う。しかし、このルールが頭の中にだけ残るなら、正直にバックテストすることはできない。Pine Script は曖昧な文を可視化する。

もう一つ微妙な点がある。市場レジームは方向だけの問題ではない。

市場は強気でありながら危険でもありうる。株価は200日移動平均線の上にありながらスプレッドは広がりうる。指数は高値圏にありながらリーダーシップは狭まりうる。戦略は全体として利益を出していても、特定の三種類の週にだけ脆弱かもしれない。

だから真剣な Pine Script 作業では、少なくとも四つの層を分けるべきだ。

第一の層は方向だ。資産は上昇トレンドか、下降トレンドか、横ばいか。

第二の層はボラティリティだ。現在のレンジは通常か、圧縮されているか、拡大しているか。

第三の層は流動性とコストだ。ストップ注文や成行注文は、バックテストの想定より大きなスリッページを受けやすいか。

第四の層は実行タイミングだ。戦略はバー確定で動くのか、次のバーの始値で動くのか、バー内で動くのか、下位時間足の確認後に動くのか。

初心者のスクリプトの多くは、この四層を一つのシグナルに押し込める。「RSIが30を上抜いた」「価格が移動平均を上抜いた」と言う。それはインジケーターには十分かもしれない。戦略にはほとんどの場合、十分ではない。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/cost-assumptions.png" alt="スリッページ、手数料、スプレッド、ボラティリティがバックテスト内の隠れたコストとして働く様子を示すリサーチ画像" loading="lazy" decoding="async">
  <figcaption>ボラティリティの高い市場では、コスト仮定は小さな詳細ではない。戦略と錯覚を分ける差になりうる。</figcaption>
</figure>

現在の市場では、コスト仮定がとくに重要になる。

TradingView のストラテジーは broker emulator を通じて実行される。このエミュレーターは有用だが、あくまでシミュレーションだ。標準ではチャートデータと、バー内で価格がどのように動いたかについての仮定を使う。TradingView の Bar Magnifier は下位時間足データを使って過去の約定をより精密にできるが、それでもバックテストを実際の板に変えるわけではない。

つまり、手数料ゼロ、スリッページゼロで非常によく見える戦略は、多くの場合コストの錯覚かもしれない。

Pine Script はその錯覚を小さくする道具を与えてくれる。`strategy()` 宣言では、手数料、スリッページ、証拠金、注文サイズ、ピラミッディング、約定動作を設定できる。これらは退屈な設定ではない。仮説の一部だ。

たとえば次のように書ける。

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

これは Strategy Tester パネルにすべての仮定を隠す戦略より、ずっと正直な物語を語る。

AIが生成する Pine Script の危険な点は、こうした仮定を飛ばしながら、見栄えのよいコードを作れてしまうことだ。反対に、AIが生成する Pine Script の有用な点は、エージェントに毎回こうした仮定を尋ねさせるよう訓練できることだ。

戦略を生成する前に、エージェントは問うべきだ。

> どの手数料とスリッページを想定するのか。

> ピラミッディングは許可するのか。

> ポジションサイズは固定か、ボラティリティ調整か、資産額ベースか。

> 高ボラティリティレジームではエントリーを止めるべきか。

> エントリーはバー確定を待つべきか。

> 上位時間足データはリペイントしないよう扱うべきか。

> この戦略はバックテスト専用か、アラート用か、外部システムを通じたライブ実行用か。

これらの問いは最初のドラフトを遅くする。だが二つ目のドラフトをはるかに有用にする。

ヘッドラインに動かされる市場では、リペイントもさらに重要になる。価格がローソク足の内部で激しく動くと、バー内で反応したくなる。しかし過去バーとリアルタイムバーは、すべてのスクリプトで同じように振る舞うわけではない。TradingView のドキュメントは、`request.security()` と上位時間足データが慎重に扱われなければリペイント問題を生む可能性があると繰り返し警告している。

これは単なる技術的警告ではない。戦略上の警告だ。

過去データ上でマクロ主導の反転をすべて捉えているように見えるスクリプトは、その時点では実際には利用できなかった情報を使っているかもしれない。ローソク足の高値や安値にぴったり出るシグナルは、足が閉じる前には確定していなかったかもしれない。上位時間足のトレンドフィルターは、履歴上では安定して見えてもリアルタイムでは揺れるかもしれない。

この環境では、より安全な初期値はしばしば退屈だ。

    confirmedSignal = barstate.isconfirmed and rawSignal

この一行が戦略を利益化するわけではない。テストを少しだけ魔法でなくす。

アラートにも同じことが言える。

穏やかな市場で悪いアラートは迷惑なだけだ。ボラティリティの高い市場では、悪いアラートは高くつく。Pine Script のアラートが "BUY" や "SELL" だけを送るなら、次の実行層に必要な文脈が欠けている。

より有用なアラートメッセージには、レジーム、価格、ストップ、リスクサイズ、取引理由が含まれる。目的は行動を起こすことだけではない。あとでその行動を監査できるようにすることだ。

たとえば、アラートペイロードには次のような情報を含められる。

    {
      "symbol": "{{ticker}}",
      "side": "buy",
      "reason": "breakout_with_trend_filter",
      "regime": "normal_volatility",
      "risk_model": "atr_stop",
      "timeframe": "{{interval}}"
    }

これが重要なのは、Pine Script が通常、最終的な実行層ではないからだ。ストラテジーやインジケーターはアラートを作れるが、実際の売買システムは多くの場合、Webhook、外部サーバー、ブローカーAPI、リスク管理を通って続く。アラートメッセージが曖昧なら、自動化チェーン全体が脆くなる。

<figure class="article-figure">
  <img src="/assets/images/posts/pine-script-market-regime/alert-context.png" alt="Pine Scriptのアラートがレジーム、リスク、実行文脈を外部取引システムへ運ぶ様子を示すリサーチ画像" loading="lazy" decoding="async">
  <figcaption>アラートは買いか売りを言うだけで終わってはいけない。リスク管理と事後検証に必要な文脈を運ぶべきだ。</figcaption>
</figure>

現在の市場には、もう一つ静かな問題がある。戦略の半減期だ。

市場がAIインフラ、エネルギーショック、金利期待、地政学的ヘッドラインのような単一テーマに支配されると、一部の戦略は短い期間だけ強く見える。モメンタムルールが機能するのは、その構造が強いからではなく、一つのテーマが一時的にすべてを圧倒しているからかもしれない。

ここで Pine Script は、トレーダーが最近の成績を疑う助けになるべきだ。

よいスクリプトは純利益だけを示してはいけない。次のような問いを促すべきだ。

- この戦略は現在の市場テーマ以前にも機能したか。
- 低ボラティリティ期と高ボラティリティ期の両方を生き残ったか。
- 最良の取引は短いクラスターからだけ生まれていないか。
- 成績は一つの資産、一つのセクター、一つの月に依存していないか。
- 現実的なスリッページと手数料を入れても利益が出たか。
- パラメーターを少し変えても安定していたか。

AIはここで役に立つ。ただし、エージェントが正しく設計されている場合に限る。エージェントは高いバックテスト収益を早く称賛しすぎてはいけない。堅牢性チェックを求めるべきだ。

たとえば Pine Script を生成したあと、エージェントはこう提案できる。

> 同じロジックをATR長10、14、21でテストする。

> ブレイクアウト長20、30、50をテストする。

> スリッページを2倍にして戦略を実行する。

> VIXが選んだしきい値を上回るとき、ロングエントリーを無効化する。

> 現在のマクロレジームの前後で結果を比較する。

> エントリーがバー確定を待っても戦略が機能するか確認する。

これがコード生成と戦略リサーチの違いだ。

コード生成はこう言う。

> これがあなたの Pine Script です。

戦略リサーチはこう言う。

> これがあなたの Pine Script であり、これを壊しうる仮定です。

後者のほうが価値がある。

現在の金融環境は、市場が一つの機械ではないことを思い出させる。市場は金利、インフレ、流動性、セクターリーダーシップ、ポジショニング、ボラティリティ、ニュースが積み重なった機械の束だ。Pine Script 戦略がそのすべてを完全に理解することはできない。だが、それらが存在しないふりをやめることはできる。

それが実用的な目標だ。

よい Pine Script 戦略は、すべてのマクロイベントを予測する必要はない。自分のシグナルが弱いとき、コスト仮定が脆いとき、バックテストが嘘をつくかもしれないときを知るべきだ。

静かな市場では、単純なスクリプトも賢く見える。

難しい市場では、仮定が見えるようになる。

だから次世代のAIトレーディングエージェントは、Pine Script をより速く書くだけであってはならない。レジーム、コスト、約定、リペイント、アラート、リスクという隠れた仮定を明示すべきだ。

売買アイデアはコードになるべきだ。しかし2026年のコードは、自分がどのような市場に立っているのかも知るべきだ。

## 参考資料

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
