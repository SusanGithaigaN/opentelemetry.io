---
title: サンプリング
description: サンプリングと、OpenTelemetryで利用可能なさまざまなサンプリングオプションについて学びましょう。
weight: 80
default_lang_commit: 825b6e2
---

分散トレースでは、リクエストが分散システム内のあるサービスから別のサービスに移動するのを観察できます。
サービスの接続を理解したり、レイテンシーの問題を診断したり、その他の利点の中でも、多くの理由で非常に実用的です。

しかし、リクエストの大半がステータスコード200番で成功し、許容できないレイテンシーやエラーもなく終了する場合、そのようなデータが本当に必要でしょうか。
問題はここからです。
正しいインサイトを見つけるために、常に大量のデータが必要なわけではありません。
_適切なデータのサンプリングが必要なだけです。_

![図は、すべてのデータをトレースする必要はなく、データのサンプルで十分であることを示しています。](traces-venn-diagram.svg)

サンプリングの背後にある考え方は、オブザーバビリティバックエンドに送信するスパンを制御することであり、その結果、インジェストコストを削減することです。
異なる組織には、 _なぜ_ サンプリングしたいのかだけでなく、 _何_ をサンプリングしたいのかについて独自の理由があるでしょう。
あなたは、サンプリング戦略を以下のようにカスタマイズしたいかもしれません。

- **コスト管理**: テレメトリーの量が多ければ、テレメトリーバックエンドベンダーやクラウドプロバイダーから、すべてのスパンをエクスポートして保存するために、多額の料金を請求されるリスクがあります。
- **関心のあるトレースに注目する**: たとえば、フロントエンドチームは特定のユーザー属性を持つトレースだけを見たいかもしれません。
- **ノイズをフィルタリングする**: たとえば、ヘルスチェックを除外したい場合などです。

## 用語解説

サンプリングについて議論する際には、一貫した用語を使用することが重要です。
トレースまたはスパンは、「サンプリングされた」または「サンプリングされていない」と見なされます。

- **サンプリングされた**: トレースまたはスパンが処理され、エクスポートされます。これはサンプラーが母集団の代表として選んだものであるため、「サンプリングされた」とみなされます。
- **サンプリングされていない**: トレースまたはスパンは、処理またはエクスポートされません。サンプラーによって選択されていないため、「サンプリングされていない」とみなされます。

ときどきこれらの用語の定義が混同されることがあります。
誰かが「データをサンプリングアウトしている」と言ったり、処理またはエクスポートされていないデータは「サンプリングされた」と見なされると言ったりするのを見かけるかもしれません。
これらは間違った表現です。

## ヘッドサンプリング

ヘッドサンプリングは、サンプリングの決定をできるだけ早期に行うために用いられるサンプリング技術です。
スパンやトレースのサンプリングまたはドロップの決定は、トレース全体を検査することによって行われるわけではありません。

たとえば、ヘッドサンプリングのもっとも一般的な形式は、[一貫した確率サンプリング](/docs/specs/otel/trace/tracestate-probability-sampling/#consistent-probability-sampling)です。
決定論的サンプリングと呼ばれることもあります。
この場合、サンプリングの決定は、トレースIDと、サンプリングするトレースの望ましい割合に基づいて行われます。
これにより、全トレースの5%など、一貫した割合で、スパンの欠損無く、全トレースがサンプリングされます。

ヘッドサンプリングの利点は以下の通りです。

- わかりやすい
- 設定しやすい
- 効果的
- トレース収集パイプラインのどの時点でも可能

ヘッドサンプリングの主な欠点は、トレース全体のデータに基づいてサンプリングの決定を下すことができないことです。
つまり、ヘッドサンプリングは雑な実装としては有効ですが、システム全体の情報を考慮しなければならないサンプリング戦略にはまったくもって不十分です。
たとえば、ヘッドサンプリングを使用して、エラーを含むすべてのトレースを確実にサンプリングすることは不可能です。
そのためには、テイルサンプリングが必要である。

## テイルサンプリング

テイルサンプリングとは、トレース内のすべてのスパンまたはほとんどのスパンを考慮して、トレースのサンプリングを決定することです。
テイルサンプリングは、ヘッドサンプリングでは不可能な、トレースの異なる部分から得られる特定の基準に基づいてトレースをサンプリングするオプションを提供します。

![図は、スパンがルートスパンからどのように発生するかを示しています。スパンが完了した後、テイルサンプリングプロセッサはサンプリング決定を行います。](tail-sampling-process.svg)

テイルサンプリングの使い方の例としては、以下のようなものがあります。

- エラーを含むトレースを常にサンプリングする
- 全体的なレイテンシーに基づいてトレースをサンプリングする
- トレース内の1つまたは複数のスパンにおける特定の属性の存在または値に基づいてトレースをサンプリングする。たとえば、新しくデプロイされたサービスからより多くのトレースをサンプリングする。
- 特定の基準に基づいてトレースに異なるサンプリングレートを適用する

お分かりのように、テイルサンプリングは、より高度なことを可能にします。
テレメトリーをサンプリングしなければならない大規模なシステムでは、データ量とそのデータの有用性のバランスを取るために、ほとんどの場合、テイルサンプリングを使用する必要があります。

今日のテイルサンプリングには、主に3つの欠点があります。

- テイルサンプリングの実施は難しいです。
  利用可能なサンプリング技術の種類にもよりますが、それは必ずしも「一度設定したら放置できる」ものではありません。
  システムが変われば、サンプリング戦略も変わります。
  大規模で洗練された分散システムでは、サンプリング戦略を実装するルールも大規模で洗練されたものになります。
- テイルサンプリングの運用は難しいものです。
  テイルサンプリングを実装するコンポーネントは、大量のデータを受け入れ、保存できるステートフルなシステムでなければなりません。
  トラフィックパターンによっては、リソースの利用方法が異なる数十、数百のノードが必要になることもあります。
  さらに、テイルサンプラーは、受信するデータ量に追いつけない場合、計算量の少ないサンプリング技術に「フォールバック」する必要があるかもしれません。
  このような要因から、テイルサンプリングコンポーネントを監視して、正しいサンプリング決定を行うために必要なリソースを確保することが重要です。
- テイルサンプラーは、今日、ベンダー固有のテクノロジーの領域で終わることが多いです。
  オブザーバビリティに有料のベンダーを使用している場合、もっとも効果的なテイルサンプリングのオプションは、そのベンダーが提供するものに限定される可能性があります。

最後に、システムによっては、テイルサンプリングがヘッドサンプリングと併用されることがあります。
たとえば、非常に大量のトレースデータを生成する一連のサービスは、最初にヘッドサンプリングを使ってトレースのごく一部だけをサンプリングし、テレメトリーパイプラインの後半でテイルサンプリングを使って、バックエンドにエクスポートする前に、より洗練されたサンプリング決定を行うことができます。
これは、テレメトリーパイプラインが過負荷にならないように保護するために、しばしば行われます。

## サポート

### コレクター

OpenTelemetryコレクターには、以下のサンプリングプロセッサーがあります。

- [Probabilistic Sampling Processor（確率的サンプリングプロセッサー）](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/probabilisticsamplerprocessor)
- [Tail Sampling Processor（テイルサンプリングプロセッサー）](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/tailsamplingprocessor)

### 言語SDK

OpenTelemetry API & SDK の各言語固有の実装については、それぞれのドキュメントページでサンプリングのサポート状況を確認出来ます。

{{% sampling-support-list " " %}}