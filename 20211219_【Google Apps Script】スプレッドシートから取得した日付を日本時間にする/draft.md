# はじまり
今回、ぶち当たった課題が、GASでスプレッドシートから読み取った日付が何だかズレている。というものです。
具体的には、上のスプレッドシートの「2021-12-18」は、GASで`new Date()`で取得した日付を入れたのですが、その日付をGASで読み取ると「2021-12-17」的な感じにズレていました。
![](https://storage.googleapis.com/zenn-user-upload/31e10c3a6b88-20211219.png)
![](https://storage.googleapis.com/zenn-user-upload/3b13fec4ef40-20211219.png)


# マニフェストファイルの修正
## 設定用jsonファイルの表示
GASのエディタ画面で歯車マークをクリックすると、**「appsscript.json」マニフェスト ファイルをエディタで表示する**のチェックボックスがあるのでチェックを入れます。
すると、「appscript.json」のファイルを編集できるようになります。
![](https://storage.googleapis.com/zenn-user-upload/6780c71f8e68-20211219.png)

## appscript.json（修正前）
~~~javascript
{
  "timeZone": "America/New_York",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
~~~

## appscript.json（修正後）
~~~javascript
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
~~~

## 修正後の表示
マニフェストファイルを修正した後は、下の画像のように読み取った日付が「2021-12-18」的な感じになりました。
![](https://storage.googleapis.com/zenn-user-upload/b6336ea1666b-20211219.png)

# 原因を考える
スプレッドシートの設定では、事象が起きた時もちゃんと日本時間に設定されていました。
![](https://storage.googleapis.com/zenn-user-upload/0a3ce1f55ff3-20211219.png)

いや、むしろスプレッドシートの時間が日本時間に設定されていたからこそ、GASの時間がニューヨーク時間になるように時差を埋め合わせてくれたのかもしれませんね。

そもそも、修正前の`new Date()`の時間もよく見たら、2021-12-19の日にちで実行したのに、2021-12-18になってたし。
![](https://storage.googleapis.com/zenn-user-upload/3b13fec4ef40-20211219.png)


# おしまい
日付の部分は、一般的なDBのサーバ時間と同様に、GASとスプレッドシートでもタイムゾーンを気を付けなければならなかったのでした。
