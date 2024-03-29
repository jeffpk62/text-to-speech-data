---

copyright:
  years: 2019
lastupdated: "2019-06-04"

subcollection: text-to-speech-data

---

{:shortdesc: .shortdesc}
{:external: target="_blank" .external}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:deprecated: .deprecated}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# 単語のタイミングの取得
{: #timing}

WebSocket インターフェースの機能は、HTTP の `GET /v1/synthesize` メソッドおよび `POST /v1/synthesize` メソッドと同じです。 また、WebSocket インターフェースを使用して、入力中の特定の位置やすべての単語のタイミング情報を以下のようにして取得することもできます。
{: shortdesc}

-   音声の中でマーカーが出現するタイミングを識別する場合は、入力テキストに SSML の `<mark>` 要素を指定する。
-   入力テキストのすべてのストリングのタイミング情報を取得する場合は、JSON テキスト・メッセージの `timings` パラメーターを指定する。

タイミング情報は、音声と入力テキストを同期する際に役立ちます。 例えば、合成音声の内容にロボットのジェスチャーを合わせることができます。

日本語の入力テキストでは、`timings` パラメーターはサポートされません。
{: note}

## サービスが単語のタイミングを戻す仕組み
{: #timingHow}

マークまたは単語のタイミング情報を戻すために、サービスは、独立したバイナリー・ストリームとテキスト・ストリームを多重送信して応答を構成します。

-   `<mark`> 要素ごとに、サービスは JSON テキスト・メッセージを戻します。 各メッセージには、合成音声の開始時を基点として、マークが出現する正確な時間が示されています。
-   すべてのストリングの単語のタイミングを戻す場合は、サービスは JSON テキスト・メッセージを 1 つ以上戻します。 各メッセージには、各単語およびその (合成音声の開始時を基点とした) 開始時間と終了時間を示す配列が含まれています。

サービスから送信されるバイナリー・ストリームとテキスト・ストリームは互いに独立しています。 そのため、サービスから送信する音声チャンクの数や、ユーザーがテキスト・メッセージと音声メッセージを受け取るタイミングをサービスで制御することはほとんどできません。 例えば、音声合成が音声圧縮よりも速い場合は、音声がまったく到着していないのにテキスト・メッセージがすべて到着する可能性があります。

実際問題として、サービスから送信される音声チャンクの数は予測不能です。各テキスト・メッセージの前後に複数の音声チャンクが送信される場合もあります。 また、特定のマークまたは単語のタイミング情報の前と後に出現する音声データが、単一のバイナリー・チャンクに含まれる場合もあります。

ただし、タイミング情報を含むテキスト・メッセージは、必ず、対応する音声を含むバイナリー・チャンクより先に到着します。 また、音声メッセージは必ず順番に到着するので、バイナリー結果からテキストの音声合成を正確に構築できます。

## SSML マークの指定
{: #mark}

オプションの SSML 要素 `<mark>` は、合成するテキスト中にマーカーを置くための空タグです。 `<mark>` 要素の前にあるすべてのテキストが合成されると、クライアントに通知されます。

この要素は、マークを一意的に識別するストリングを指定する単一の `name` 属性を受け入れます。 この名前の先頭は英数字にする必要があります。 サービスは、マークの名前と、合成音声の開始時を基点としてマークが出現する時間を返します。 入力テキストには任意の数のマークを指定できます。

以下の JavaScript コードのスニペットには、`here` という名前の `<mark>` 要素のインスタンスが指定されています。

```javascript
function onOpen(evt) {
  var message = {
    text: 'Hello <mark name="here"/> world',
    accept: '*/*'
  };
  websocket.send(JSON.stringify(message));
}
```
{: codeblock}

マークの前にあるテキストの合成が終了すると、サービスは、マークの名前とそのマークが音声の中で出現する時間 (秒単位) を示す、テキスト・メッセージを送信します。

```javascript
{
  "marks": [
    ["here", 0.5019387755102041]
  ]
}
```
{: codeblock}

タイミング情報を含むテキスト・メッセージは、必ず、そのマークの位置を含む音声チャンクより先に到着します。

## すべての単語のタイミング情報の要求
{: #timingRequest}

要求で JSON オブジェクトのオプションの `timings` パラメーターをサービスに渡すと、入力テキストのすべてのストリングのタイミング情報が返されます。 この利便性のおかげで、入力の単語ごとに SSML の `<mark>` 要素を指定する必要はありません。 ストリング `words` を含む配列を渡して、単語のタイミング情報を要求します。 空の配列を渡したり、パラメーターを省略したりすると、タイミング情報は返されません。

サービスは、個々の `<mark>` 要素のタイミング情報を返すときと同じ方法で、単語のタイミング情報を WebSocket 接続を介して返します。 1 つ以上の JSON テキスト・メッセージを戻します。 各メッセージには、各単語およびその (合成音声の開始時を基点とした) 開始時間と終了時間 (秒単位) を示す配列が含まれています。 例えば、以下の例では単語のタイミング情報を要求しています。

```javascript
function onOpen(evt) {
  var message = {
    text: 'I have a pet bird.',
      accept: '*/*',
      timings: ['words']
  };
  websocket.send(JSON.stringify(message));
}
```
{: codeblock}

応答で、サービスは以下のようなテキスト・メッセージを返します。

```javascript
{
  "words": [
    ["I", [0.0690258394023930, 0.1655782733012873]]
    ["have", [0.1655789302434486, 0.3722901056092351]]
    ["a", [0.3722906798320199, 0.4012192331086645]]
  ]
}
{
  "words": [
    ["pet", [0.4012195492838347, 0.5798213856109801]]
    ["bird.", [0.5798218710823425, 0.7440360383928273]]
  ]
}
```
{: codeblock}

この応答は例にすぎません。 サービスは、入力のタイミング情報を含むテキスト・メッセージを 1 つ以上返すことがあります。 さらには、複数のメッセージの間に、バイナリーの音声チャンクの応答を挟む場合もあります。 ただし、単語のタイミング情報を含むテキスト・メッセージ送信は、必ず、その単語を含む音声チャンクより先に到着します。

### プレーン・テキストのタイミング
{: #timingText}

サービスの合成プロセスには、数値、日付、時間、通貨額、頭字語、および略語を読み上げるテキスト正規化ステップが含まれます。 この結果は、これらのストリングがどのように発話されるかに対応します。 例えば、ストリング *$200* は *two*、*hundred*、および *dollars* の 3 つの単語として発話されます。 単語のタイミング情報は音声と入力テキストの同期に使用されるため、サービスは入力の非正規化スペルに対応するタイミング情報を返します。

例えば、以下の入力テキストについて考えます。

```
The coldest recorded temperature is -89.2 degrees Celsius in Antarctica on July 21, 1983!
```
{: codeblock}

サービスは、次のストリングの音声タイミングを返します。

"*The*"、"*coldest*"、"*recorded*"、"*temperature*"、"*is*"、"*-89.2*"、"*degrees*"、"*Celsius*"、"*in*"、"*Antarctica*"、"*on*"、"*July*"、"*21,*"、"*1983!*"

"*-89.2*" は、音声では 5 つの個別の単語 (*minus*、*eighty*、*nine*、*point*、*two*) で発話されますが、テキスト・メッセージには、単一ユニットとしてストリングのタイミング情報である *minus* の開始時間と *two* の終了時間が示されます。

前の例に示したように、非正規化ストリングには句読点も含まれる場合があります。 サービスは、自身が返すテキスト・メッセージに、タイミング情報とともに単語の前後にある句読点を含めます。 例えば、ストリング "*21,*" および "*1983!*"には、サービスがテキスト・メッセージで返す句読点が含まれます。 句読点は無音になりますが、単語の音声タイミングにその無音は*含まれません*。

例えば、以下の条件文を含む入力テキストについて考えてみましょう。

```
If it is sunny, I will go to the beach.
```
{: codeblock}

サービスは、無音を生成する句読点で終わる "*sunny,*" および "*beach.*" を含め、入力中のすべてのストリングのタイミング情報を返します。 ただし、"*sunny,*" のタイミング情報に、コンマが生成する無音は含まれません。また、"*beach.*" のタイミング情報に、ピリオドが生成する無音は含まれません。 この情報には、発話されるストリングのタイミングのみが反映されます。

### SSML テキストのタイミング
{: #timingSSML}

プレーン・テキストから合成する場合は、サービスは、単語のタイミング応答として、ストリングに含まれているブランク・スペースを除くすべての入力文字を返します。 SSML の場合には、これは当てはまりません。音声を生成しない SSML 要素があるからです。 以下のリストに、単語のタイミング情報に影響を与える可能性がある SSML 要素についてまとめます。

-   `<say-as>`: `<say-as>` の開始タグと閉じタグで囲まれたテキストを正規化ステップで処理する方法を示します。 属性によって、組み込みテキストの発話方法を指定します。 以下の例は、日付の発話方法を示しています。

    ```xml
    The baby was born on <say-as interpret-as="date" format="mdy">3/4/2016</say-as>.
    ```
    {: codeblock}

    サービスは、次のストリングのタイミング情報を返します。"*The*"、"*baby*"、"*was*"、"*born*"、"*on*"、"*3/4/2016.*"。 サービスは、ストリング "*3/4/2016*" を "*march fourth two thousand sixteen*" として正規化します。 このストリングの単語のタイミング情報には、"*march*" の開始時間および "*sixteen*" の終了時間が表されます。

    以下の例は、単語 `Hello` が読み上げられることを示しています。

    ```xml
    <say-as interpret-as="letters">Hello</say-as>.
    ```
    {: codeblock}

    サービスは、ストリング "*Hello*" のタイミング情報を返します。 サービスは、正規化ステップでこの単語を 1 文字ずつつづります。 応答に含まれる単語のタイミング情報には、文字 "*h*" の開始時間と文字 "*o*" の終了時間が表されます。
-   `<phoneme>`: `<phoneme>` の開始タグと閉じタグで囲まれたテキストの発音を指定します。 ただし、テキストと閉じタグは両方ともオプションです。 以下の例では、組み込みテキストおよび閉じタグが含まれています。

    ```xml
    The <phoneme alphabet="ibm" ph=".0tx.1me.0fo">tomato</phoneme> was ripe.
    ```
    {: codeblock}

    サービスは、次のストリングのタイミング情報を返します。"*The*"、"*tomato*"、"*was*"、"*ripe.*"。

    反対に、以下の例では単項の `<phoneme>` 要素を使用し、組み込みテキストおよび閉じタグはありません。

    ```xml
    The <phoneme alphabet="ibm" ph=".0tx.1me.0fo"/> was ripe.
    ```
    {: codeblock}

    この場合、サービスは次のストリングのタイミング情報を返します。"*The*"、"*&lt;phoneme&gt;*"、"*was*"、"*ripe.*"。
-   `<sub>`: 発話音声の中で、`<sub>` タグの開始タグと閉じタグで囲まれたテキストを、要素の `alias` 属性に指定したテキストに置き換えます。 例えば、以下の入力には単一の `<sub>` タグが含まれています。

    ```xml
    I work at <sub alias="International Business Machines">IBM</sub>.
    ```
    {: codeblock}

    サービスはストリング "*I*"、"*work*"、"*at*"、"*{{site.data.keyword.IBM_notm}}.*" のタイミング情報を生成します。 サービスは、ストリング "*{{site.data.keyword.IBM_notm}}*" を "*International Business Machines*" として正規化します。 このストリングのタイミング情報には、"*International*" の開始時間と "*Machines*" の終了時間が表されます。
-   `<break>`: 発話テキスト中に休止を挿入します。 生成された無音は、単語のタイミング情報の中で、`<break>` 要素の前の単語の終了時間からこの要素の後の単語の開始時間までの間隔として表されます。
- `<paragraph>` (または `<p>`): 音声に無音を追加できます。 サービスは、この無音に関してタイミング情報を返しません。
- `<sentence>` (または `<s>`): 音声に無音を追加できます。 サービスは、この無音に関してタイミング情報を返しません。

リスト内で言及されていない SSML 要素は、単語のタイミング情報に影響を与えません。 サービスの SSML サポートについて詳しくは、[SSML の使用](/docs/services/text-to-speech-data?topic=text-to-speech-data-ssml)を参照してください。

## マーク要素の例
{: #timingExample}

ここでは、クライアントとサービスの間のシンプルな WebSocket セッションを示す例を紹介します。 これらの例は、接続を開く操作ではなく、データの交換に焦点を当てています。 クライアントは、2 つの `<mark>` 要素 (`SIMPLE` と `EXAMPLE`) を含むテキスト・メッセージを送信し、WAV フォーマットの音声を返すように要求します。

```javascript
{
  "text": "This is a <mark name=\"SIMPLE\"/>simple <mark name=\"EXAMPLE\"/> example.",
  "accept": "audio/wav"
}
```
{: codeblock}

まず、サービスは音声フォーマットを確認するメッセージを送信します。 次に、結果と一緒に複数のメッセージを送信します。 サービスからクライアントに送信される音声チャンクの数や、テキスト・メッセージと音声メッセージの送信順序をサービスで保証することはできません。

次に示す両方の応答が考えられます。 どちらの場合も、サービスは、バイナリー・ストリーム中のマークの位置を示すテキスト・メッセージを 2 つ送信します。 ただし、音声を含むバイナリー・メッセージが送信される数は予測不能です。 マークのタイミング情報は、必ず、そのマークの位置を含む音声チャンクより先に到着します。

### 1 つ目の応答例

1 つ目の応答例では、複数のテキスト・メッセージが複数の音声メッセージを挟んで返されます。

```javascript
{
  "binary_streams": [
    {
      "content_type": "audio/wav"
    }
  ]
}
... one or more chunks of binary audio
    all audio precedes the SIMPLE mark ...
{
  "marks": [
    [
      "SIMPLE",
      0.7848991042702103
    ]
  ]
}
... one or more chunks of binary audio
    audio can precede and follow the SIMPLE mark
    all audio precedes the EXAMPLE mark ...
{
  "marks": [
    [
      "EXAMPLE", 1.0034702987337102
    ]
  ]
}
... one or more chunks of binary audio
    audio can precede and follow the EXAMPLE mark ...
```
{: codeblock}

### 2 つ目の応答例

2 つ目の応答例では、テキスト・メッセージがどの音声メッセージよりも先に到着します。

```javascript
{
  "binary_streams": [
    "content_type": "audio/wav"}
  ]
}
{
  "marks": [
    [
      "SIMPLE", 0.7848991042702103
    ]
  ]
}
{
  "marks": [
    [
      "EXAMPLE", 1.0034702987337102
    ]
  ]
}
... one or more chunks of binary audio ...
```
{: codeblock}
