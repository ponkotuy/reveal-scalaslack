# ScalaでWebAPIラッパーを作る

---

## おまえだれよ
- ぽんこつ(@ponkotuy)
- Maverick所属
- Scala開発とかできます
- 会社ではチューニングとかリファクタとか
- 個人で作ったやつ
  - MyFleetGirls: [myfleet.moe](https://myfleet.moe)
  - ぽんこつ とらべる: [travel.ponkotuy.com](http://travel.ponkotuy.com)

---

## scala-slack
SlackのAPIをラップするライブラリ
- 設計が秀逸
- APIが足りない
- PullRequestしても反応がない
- 自分でフォークしよ

この経験について話します

---

## HttpClientラッパ
必要なHttpClientはラップしよう

```scala
class HttpClient(baseURL: String = "https://slack.com/api") {
  def get(method: String, params: Map[String, String]): JValue = {...}
}
```

理由は

- テストしやすい
- HTTPライブラリの変更が容易

---

## Responseを型に落とす
返ってきたJSONをcase classに落とす

一番重要かつ面倒くさい

--

### エラーの返り方を統一する
- 例外？Either？独自定義型？
- scala-slackは例外

--

### 情報量の違う同じ物
例:SlackのChannel

- channels.info
- channels.list
- channels.rename

の3つが返すChannelのデータが全て異なる

-> 共通の基底となるtraitを作るべき

--

### メソッドの引数
引数に渡すパラメタは

- id
- Responseそのもの

を選べると良い

(scala-slackでは未実装)

---

## API呼び出しはclassにする

- objectにしない
- 引数に先程のHttpClientを取る

->テストしやすくなる

```scala
class Channels(client: HttpClient, token: String) {
  def info(channel: String): ChannelInfoResponse = {...}
}
```

---

## 単体テストする
- HttpClientを単体テスト用のに置き換え
- Mock or 継承(scala-slackはMock)
- クエリとJSONのパースがテスト可能
- CIサービスに登録するとなお良い

--

```scala
class ChannelsSpec extends FlatSpec with MockitoSugar with Matchers with BeforeAndAfterEach {
  private var mockHttpClient: HttpClient = _
  private var channels: Channels = _

  override def befereEash() {
    // set mock
    when(mockHttpClient.get("channels.info", params)
      .thenReturn(json)
    channels = new Channels(mockHttpClient, testApiKey)
  }

  "Channels.info()" should "get channel info" in {
    val response = channels.info("C024BE91L")
    ... // check value
  }
}
```

---

## MavenCentralに登録
- ある意味一番難しい
- ググれ

sbtで自動でできることを除けばJavaと同じ

---

## 手を動かす

品質の良いScalaのライブラリが増える

↓

みんな幸せ

ガンガンライブラリを作ろう

---

# おしまい
