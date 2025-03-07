---
title: Streams API の概念
slug: Web/API/Streams_API/Concepts
---
{{apiref("Streams")}}

[Streams API](/ja/docs/Web/API/Streams_API) は、非常に便利なツールセットを Web プラットフォームに追加し、JavaScript がネットワーク経由で受信したデータのストリームにプログラムでアクセスし、開発者の希望どおりに処理できるようにするオブジェクトを提供します。 ストリームに関連する概念と用語の一部は、初めての場合もあります。 この記事では、それら知っておく必要のあるすべてを説明します。

## 読み取り可能なストリーム

読み取り可能なストリームは、**基になるソース**（underlying source）から流れる {{domxref("ReadableStream")}} オブジェクトによって JavaScript で表されるデータソースです。 基になるソースは、ネットワーク上のどこか、またはデータを取得するドメインのどこかにあるリソースです。

基になるソースには、次の 2 つのタイプがあります。

- **プッシュソース**（Push sources）は、アクセスしたときに常にデータをプッシュします。 ストリームへのアクセスを開始、一時停止、またはキャンセルするのはユーザー次第です。 例には、動画ストリームや TCP/[Web ソケット](/ja/docs/Web/API/WebSockets_API)が含まれます。
- **プルソース**（Pull sources）では、接続後、明示的にデータを要求する必要があります。 例には、[Fetch](/ja/docs/Web/API/Fetch_API) や [XHR](/ja/docs/Web/API/XMLHttpRequest/XMLHttpRequest) の呼び出しを介したファイルアクセス操作が含まれます。

データは、**チャンク**（chunks）と呼ばれる小さな断片で順番に読み取られます。 チャンクは 1 バイトにすることも、特定のサイズの[型付き配列](/ja/docs/Web/JavaScript/Typed_arrays)など、より大きなものにすることもできます。 単一のストリームには、さまざまなサイズとタイプのチャンクを含めることができます。

![](https://mdn.mozillademos.org/files/15819/Readable%20streams.png)

ストリームに置かれたチャンクは、**キューに入れられた**（enqueued）と言われます。 これは、読み取りの準備ができているキューで待機していることを意味します。 **内部キュー**（internal queue）は、まだ読み取られていないチャンクを追跡します（下の内部キューとキューイング戦略のセクションを参照）。

ストリーム内のチャンクは**リーダー**（reader）によって読み取られます。 これにより、一度に 1 つのチャンクでデータが処理されるため、ユーザーは任意の種類の操作を実行できます。 リーダーとそれに付随する他の処理コードは、**コンシューマー**（consumer）と呼ばれます。

また、**コントローラー**（controller）と呼ばれる構造も使用します。 各リーダーには、ストリームを制御する（例えば、必要に応じてキャンセルする）ことができるコントローラーが関連付けられています 。

一度にストリームを読み取ることができるのは 1 つのリーダーのみです。 リーダーが作成され、ストリームの読み取りを開始すると、ストリームは**アクティブなリーダー**（active reader）に**ロックされている**（locked）と言います。 別のリーダーにストリームの読み取りを開始させる場合は、通常、最初のリーダーをキャンセルしてから他の操作を行う必要があります（ですが、ストリームを **tee** することができます。下のティーイングのセクションを参照）。

読み取り可能なストリームには 2 つの異なるタイプがあることに注意してください。 従来の読み取り可能なストリームに加えて、バイトストリームと呼ばれるタイプがあります。 これは、基になるバイトソース（BYOB または bring your own buffer（独自のバッファーを持ち込む）とも呼ばれる）のソースを読み取るための従来のストリームの拡張バージョンです。 これにより、開発者が提供するバッファにストリームを直接読み込むことができ、必要なコピーが最小限に抑えられます。 コードが使用する基になるストリーム（そして拡張により、リーダーとコントローラー）は、最初にストリームがどのように作成されたかによって異なります（{{domxref("ReadableStream.ReadableStream","ReadableStream()")}} コンストラクターのページを参照）。

> **Warning:** **重要**: バイトストリームは、まだどこにも実装されていません。 仕様の詳細が実装に十分な完成状態にあるかどうかについて疑問が提起されています。 これは時間とともに変化する可能性があります。

フェッチ要求からの {{domxref("Response.body")}} などのメカニズムを介して既製の読み取り可能なストリームを使用するか、{{domxref("ReadableStream.ReadableStream","ReadableStream()")}} コンストラクターを使用して独自のストリームを使用できます。

## ティーイング

一度にストリームを読み取ることができるのは 1 つのリーダーだけですが、ストリームを 2 つの同一のコピーに分割し、2 つの別々のリーダーで読み取ることができます。 これを**ティーイング**（teeing）と呼びます。

JavaScript では、これを {{domxref("ReadableStream.tee()")}} メソッドを介して実現します — 元の読み取り可能なストリームの 2 つの同一コピーを含む配列を出力し、2 つの別々のリーダーで個別に読み取ることができます。

例えば、サーバーから応答を取得してブラウザーにストリームしたいが、[ServiceWorker](/ja/docs/Web/API/Service_Worker_API) キャッシュにもストリームしたい場合に、ServiceWorker でこれを行うことができます。 応答のボディを複数回使用することはできず、ストリームを一度に複数のリーダーで読み取ることはできないため、これを行うには 2 つのコピーが必要です。

![](https://mdn.mozillademos.org/files/15820/tee.png)

## 書き込み可能なストリーム

**書き込み可能なストリーム**（writable stream）は、{{domxref("WritableStream")}} オブジェクトによって JavaScript で表されるデータの書き込み先です。 これは、**基になるシンク**（underlying sink、生データが書き込まれる下位レベルの I/O シンク）の上部の抽象化として機能します。

データは、**ライター**（writer）を介して一度に 1 つのチャンクでストリームに書き込まれます。 チャンクは、リーダーのチャンクと同様に、多数の形式をとることができます。 任意のコードを使用して、書き込み可能なチャンクを生成できます。 ライターとそれに関連するコードを**プロデューサー**（producer）と呼びます。

ライターが作成され、ストリームへの書き込みを開始すると、ストリームは**アクティブなライター**（active writer）に**ロックされている**（locked）と言われます。 一度に書き込み可能なストリームに書き込むことができるのは、1 つのライターのみです。 別のライターにストリームへの書き込みを開始させたい場合は、通常、別のライターを取りつける前にそれを中止する必要があります。

**内部キュー**（internal queue）は、ストリームに書き込まれたが、基になるシンクによってまだ処理されていないチャンクを追跡します。

また、コントローラーと呼ばれる構造も使用します。 各ライターには、ストリームを制御する（例えば、必要に応じてストリームを中止する）ことができるコントローラーが関連付けられています 。

![](https://mdn.mozillademos.org/files/15821/writable%20streams.png)

{{domxref("WritableStream.WritableStream","WritableStream()")}} コンストラクターを使用して、書き込み可能なストリームを利用できます。 現在、これらのブラウザーでの可用性は非常に限られています。

## パイプチェーン

Streams API を使用すると、**パイプチェーン**（pipe chain）と呼ばれる構造を使用して、ストリームを相互にパイプすることができます（または、少なくともブラウザーが関連機能を実装する場合はそうなります）。 これを容易にするために、仕様には次の 2 つのメソッドがあります。

- {{domxref("ReadableStream.pipeThrough()")}} — **変換ストリーム**（transform stream）を介してストリームをパイプします。 変換ストリームは、データが書き込まれる書き込み可能なストリームと、データが読み取られる読み取り可能なストリームで構成されるペアです。 これは、データを常に取り込み、新しい状態に変換する一種のトレッドミルとして機能します。 事実上、同じストリームに書き込まれ、同じ値が読み取られます。 簡単な例は、生のバイトが書き込まれ、次に文字列が読み取られるテキストデコーダーです。 仕様には、より有用なアイデアと例があります。 アイデアについては、[ストリームの変換](https://streams.spec.whatwg.org/#ts-model)（英語）と、[この Web ソケットの例](https://streams.spec.whatwg.org/#example-both)（英語）を参照してください。
- {{domxref("ReadableStream.pipeTo()")}} — パイプチェーンの終点として機能する書き込み可能なストリームにパイプします。

パイプチェーンの始まりは**元のソース**（original source）と呼ばれ、終わりは**最終的なシンク**（ultimate sink）と呼ばれます。

![](https://mdn.mozillademos.org/files/15818/PipeChain.png)

> **Note:** **注**: この機能はまだ十分に検討されておらず、多くのブラウザーでは利用できません。 ある時点で、仕様作成者は、`TransformStream` クラスのようなものを追加して、変換ストリームの作成を容易にすることを望んでいます。

## バックプレッシャー

ストリームの重要な概念は**バックプレッシャー**（backpressure）です。 これは、単一のストリームまたはパイプチェーンが読み取り/書き込みの速度を調整するプロセスです。 チェーンの後半のストリームがまだビジーで、さらに多くのチャンクを受け入れる準備ができていない場合、チェーンを介して信号を逆方向に送信して、より前の変換ストリーム（または元のソース）に必要に応じて配信速度を落とすよう指示し、どこもボトルネックにならないようにします。

`ReadableStream` でバックプレッシャーを使用するには、コントローラーの {{domxref("ReadableStreamDefaultController.desiredSize")}} プロパティを照会することで、コンシューマーが希望するチャンクサイズをコントローラーに問い合わせます。 それが低すぎる場合、`ReadableStream` は、基になるソースにデータの送信を停止するように指示でき、ストリームチェーンに沿ってバックプレッシャーをかけます。

後でコンシューマが再びデータを受信したい場合は、ストリームの作成で `pull` メソッドを使用して、データをストリームに与えるよう基になるソースに指示できます。

## 内部キューとキューイング戦略

前に述べたように、まだ処理されて終了していないストリーム内のチャンクは、内部キューによって追跡されます。

- 読み取り可能なストリームの場合、これらはキューに入れられたがまだ読み取られていないチャンクです。
- 書き込み可能なストリームの場合、これらは書き込まれたが、基になるシンクによってまだ処理されていないチャンクです。

内部キューは、**内部キューの状態**（internal queue state）に基づいてバックプレッシャーを通知する方法を決定する**キューイング戦略**（queuing strategy）を採用しています。

一般に、この戦略では、キュー内のチャンクのサイズを**最高水準点**（high water mark）と呼ばれる値と比較します。これは、キューが現実的に管理できる最大の合計チャンクサイズです。

実行される計算は次です。

```
最高水準点 - キュー内のチャンクの合計サイズ = 希望サイズ
```

**希望サイズ**（desired size）は、ストリームの流れを維持するためにストリームが受け入れることができるチャンクのサイズですが、サイズは最高水準点未満です。 計算が実行された後、希望サイズをゼロより大きく保ちながら、ストリームの流れを可能な限り高速に保つために、必要に応じてチャンク生成が減速/高速化されます。 値がゼロ（書き込み可能なストリームの場合はそれ以下）になった場合、ストリームが処理できるよりも速くチャンクが生成されていることを意味し、問題が発生します。

> **Note:** **注**: ゼロまたは負の希望サイズの場合に何が起こるかは、これまで仕様で実際に定義されていません。 忍耐は美徳なり。

例として、1 のチャンクサイズと 3 の最高水準点を考えてみましょう。 これは、最高水準点に到達してバックプレッシャーが適用される前に、最大 3 つのチャンクをキューに入れることができることを意味します。
