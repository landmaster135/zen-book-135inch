# はじまり
この前、コーディングしている時にふと気になったのですが、ディープコピーの要否って一体どうなんだろう？と思って、書いた記事になります。

## 追記分（2022/01/07）


# まず、このコードを。
## 失敗コード
まず、このコードを実行すると、`displayList`にちゃんと値が入っていないことが確認できます。どうやら~~シャローコピーになっている~~参照をコピーしただけなのが原因のようです。

~~~javascript
import fetch from 'node-fetch';
import _ from 'lodash';

const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';

const main = async () => {
    let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
    let dataList = await res.json();
    let partOfDataList = {};
    let displayList = [];

    // declare for count
    let i = 0;
    const number_of_display = 5;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        displayList.push(partOfDataList); // fix on 20220110
        // displayList.push({...partOfDataList}) // fix on 20220110
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220110
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
~~~

## `console.log(displayList);`の結果（失敗コード）

~~~shell
{
  title: '【Javascript】連想配列とかをディープコピーする前に',
  pubDate: '2022-01-03 06:16:42'
}
{
  title: '【Docker】ファイル実行できるNode.js環境を作る',
  pubDate: '2022-01-02 14:00:39'
}
{ title: 'Web上の画像をGoogleドライブに保存する', pubDate: '2021-12-26 10:06:05' }
{
  title: '【Javascript】Kindleの蔵書のタイトルだけを一覧で取得するツールを作りました',
  pubDate: '2021-12-25 20:43:42'
}
{ title: '【Python】ジェネレータとyieldでちと遊んだ', pubDate: '2021-12-19 13:41:03' }
[
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  },
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  },
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  },
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  },
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  }
]
~~~

そこで、以下の方法で解消を試みました。
いずれの方法でも解消できています。

# 解消法その１（変数を毎回宣言）
## 調査用コード
まずは、while句の外で宣言していた`partOfDataList`をwhileの中で都度宣言してみました。参照コピーですが実現可能です。

~~~javascript
import fetch from 'node-fetch';
import _ from 'lodash';

const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';

const main = async () => {
    let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
    let dataList = await res.json();
    // let partOfDataList = {};
    let displayList = [];

    // declare for count
    let i = 0;
    const number_of_display = 5;

    const start = performance.now();
    while(i < number_of_display){
        let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        displayList.push(partOfDataList); // fix on 20220110
        // displayList.push({...partOfDataList}) // fix on 20220110
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220110
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
~~~

# 解消法その２（~~ディープコピー~~スプレッド構文）
## 調査用コード
次は、`push`する際に~~ディープ~~シャローコピーするスプレッド構文を使う方法です。

~~~javascript
import fetch from 'node-fetch';
import _ from 'lodash';

const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';

const main = async () => {
    let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
    let dataList = await res.json();
    let partOfDataList = {};
    let displayList = [];

    // declare for count
    let i = 0;
    const number_of_display = 5;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        // displayList.push(partOfDataList); // fix on 20220110
        displayList.push({...partOfDataList}) // fix on 20220110
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220110
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
~~~

# 解消法その３（ディープコピー）
## 調査用コード
次は、`push`する際にディープコピーするを使う方法です。lodashを使いました。

~~~javascript:解消法その３（ディープコピー）.js
import fetch from 'node-fetch';
import _ from 'lodash';

const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';

const main = async () => {
    let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
    let dataList = await res.json();
    let partOfDataList = {};
    let displayList = [];

    // declare for count
    let i = 0;
    const number_of_display = 5;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        // displayList.push(partOfDataList); // fix on 20220110
        // displayList.push({...partOfDataList}) // fix on 20220110
        displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220110
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
console.log(end - start);
~~~

## `console.log(displayList);`の結果（解消法その１～３）

~~~shell
{
  title: '【Javascript】連想配列とかをディープコピーする前に',
  pubDate: '2022-01-03 06:16:42'
}
{
  title: '【Docker】ファイル実行できるNode.js環境を作る',
  pubDate: '2022-01-02 14:00:39'
}
{ title: 'Web上の画像をGoogleドライブに保存する', pubDate: '2021-12-26 10:06:05' }
{
  title: '【Javascript】Kindleの蔵書のタイトルだけを一覧で取得するツールを作りました',
  pubDate: '2021-12-25 20:43:42'
}
{ title: '【Python】ジェネレータとyieldでちと遊んだ', pubDate: '2021-12-19 13:41:03' }
[
  {
    title: '【Javascript】連想配列とかをディープコピーする前に',
    pubDate: '2022-01-03 06:16:42'
  },
  {
    title: '【Docker】ファイル実行できるNode.js環境を作る',
    pubDate: '2022-01-02 14:00:39'
  },
  { title: 'Web上の画像をGoogleドライブに保存する', pubDate: '2021-12-26 10:06:05' },
  {
    title: '【Javascript】Kindleの蔵書のタイトルだけを一覧で取得するツールを作りました',
    pubDate: '2021-12-25 20:43:42'
  },
  {
    title: '【Python】ジェネレータとyieldでちと遊んだ',
    pubDate: '2021-12-19 13:41:03'
  }
]
~~~

# どれが良いのか？
~~２~~３通りの方法で出来ましたが、果たしてどれを使っていきましょうか？
僕としては、~~「変数を毎回宣言」する方法の方が可読性としては勝っていて良さげだなと感じました。~~
→　追記：[standard software](https://zenn.dev/standard_soft)さんからの指摘で、以前は書き方に問題がありました。しっかり書いたら特に可読性に違いは無さそうですね。
しかし、毎回宣言するのは、速度的に遅そうじゃない？　どうなんだろう？
次は、実行速度を調べてみました。

# 実行速度の調査
## 調査用コード
次の調査に使うコードはこんな感じ。ざっと、30000件のレコードが入った連想配列の処理速度を計測しました。何回か実行して計測しました。
例えば、その１は以下の感じです。

~~~javascript:解消法その１（変数を毎回宣言）.js
import fetch from 'node-fetch';
import _ from 'lodash';

const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';

const main = async () => {
    let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
    let dataList = await res.json();
    // let partOfDataList = {};
    let displayList = [];

    // declare for count
    let i = 0;
    const number_of_display = 30000;

    const start = performance.now();
    while(i < number_of_display){
        let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        displayList.push(partOfDataList); // fix on 20220110
        // displayList.push({...partOfDataList}) // fix on 20220110
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220110
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
~~~

## 測定結果
30000件での測定結果は以下のようになりました。毎回宣言のほうが少し速そうな感じはしますね。
→　（2022/01/10追記）これは、執筆当初、開発者ツールで動くスクリプトで動かした結果になります。

| Sessions |  ~~ディープコピー~~シャローコピー  |  変数を毎回宣言  |
| ---- | ---- | ---- |
| 1st |  2076.10  |  1967.60  |
| 2nd |  2061.19  |  1974  |
| 3rd |  2050.19  |   2024.59 |
| 4th |  2100.39  |  2092  |
| 5th |  2130  |  1961.10  |
| Avg. |  2083.57  |  2003.86  |

## 測定結果その２（2022/01/10追記分）
追加で、ディープコピーを含めた３通りの方法をnode.js環境で試しました。（ファイル出力で試したので先程より格段に速くなっているのはご了承下さい。）

| Sessions |  変数を毎回宣言  |  シャローコピー  |  ディープコピー  |
| ---- | ---- | ---- |  ----  |
| 1st |  215.71  |  222.50  |  257.43  |
| 2nd |  225.32  |  227.12  |  269.18  |
| 3rd |  216.66  |  218.44  |  255.03  |
| 4th |  215.89  |  217.68  |  248.55  |
| 5th |  214.94  |  219.55  |  248.06  |
| Avg. |    |    |    |

# おしまい
ディープコピーする前に、変数を毎回宣言する方法も検討してみては？

# 参考
https://qiita.com/\_kt15\_/items/0ae5085d61fa5598c76e
https://qiita.com/BlueSilverCat/items/d27a551eb4498d69f7d0
