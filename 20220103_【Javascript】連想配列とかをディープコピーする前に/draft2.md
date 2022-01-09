# はじまり
この前、コーディングしている時にふと気になったのですが、ディープコピーの要否って一体どうなんだろう？と思って、書いた記事になります。

## 追記分（2022/01/07）


# まず、このコードを。
## 失敗コード
まず、このコードを実行すると、`displayList`にちゃんと値が入っていないことが確認できます。どうやらシャローコピーになっているようです。

~~~javascript
const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';
var rss_data;

let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
let dataList = await res.json();
let partOfDataList = {};
let displayList = [];

// declare for count
let i = 0;
const number_of_display = 5;

while(i < number_of_display){
    // let partOfDataList = {};
    partOfDataList['title']   = dataList.items[i].title;
    partOfDataList['pubDate'] = dataList.items[i].pubDate;
    console.log(partOfDataList);
    displayList.push(partOfDataList); // fix on 20220107
    // displayList.push({...partOfDataList}) // fix on 20220107
    i++;
}

console.log(displayList);
~~~

## `console.log(displayList);`の結果（失敗コード）
![](https://storage.googleapis.com/zenn-user-upload/21e169d189c2-20220103.png)

そこで、以下の方法で解消を試みました。

# 解消法その１（~~ディープコピー~~スプレッド）
## 調査用コード
まずは、`push`する際に~~ディープコピーする~~スプレッド構文を使う方法です。値はちゃんと入っていることが確認できます。

~~~javascript
const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';
var rss_data;

let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
let dataList = await res.json();
let partOfDataList = {};
let displayList = [];

// declare for count
let i = 0;
const number_of_display = 5;

while(i < number_of_display){
    // let partOfDataList = {};
    partOfDataList['title']   = dataList.items[i].title;
    partOfDataList['pubDate'] = dataList.items[i].pubDate;
    console.log(partOfDataList);
    // displayList.push(partOfDataList); // fix on 20220107
    displayList.push({...partOfDataList}); // fix on 20220107
    i++;
}

console.log(displayList);
~~~

## `console.log(displayList);`の結果（解消法その１）
![](https://storage.googleapis.com/zenn-user-upload/b3c8d8e00046-20220107.png)

# 解消法その２（変数を毎回宣言）
## 調査用コード
次は、while句の外で宣言していた`partOfDataList`をwhileの中で都度宣言してみました。この方法でも解消できています。

~~~javascript
const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';
var rss_data;

let res = await fetch(`${endpoint}?rss_url=${feedUrl}`);
let dataList = await res.json();
// let partOfDataList = {};
let displayList = [];

// declare for count
let i = 0;
const number_of_display = 5;

while(i < number_of_display){
    let partOfDataList = {};
    partOfDataList['title']   = dataList.items[i].title;
    partOfDataList['pubDate'] = dataList.items[i].pubDate;
    console.log(partOfDataList);
    displayList.push(partOfDataList); // fix on 20220107
    // displayList.push({...partOfDataList}); // fix on 20220107
    i++;
}

console.log(displayList);
~~~

## `console.log(displayList);`の結果（解消法その２）
![](https://storage.googleapis.com/zenn-user-upload/b3e0587f2b84-20220107.png)

# どっちが良いのか？
２通りの方法で出来ましたが、果たしてどちらの方を使っていきましょうか？
僕としては、「変数を毎回宣言」する方法の方が可読性としては勝っていて良さげだなと感じました。
しかし、毎回宣言するのは、速度的に遅そうじゃない？　どうなんだろう？
次は、実行速度を調べてみました。

# 実行速度の調査
## 調査用コード
次の調査に使うコードはこんな感じ。ざっと、30000件のレコードが入った連想配列の処理速度を計測しました。交互に何回か実行して計測しました。

~~~javascript:解消法その１（~~ディープコピー~~ スプレット構文）.js
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
    const number_of_display = 30000;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        displayList.push(partOfDataList); // fix on 20220107
        // displayList.push({...partOfDataList}) // fix on 20220107
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220107
        i++;
    }
    const end = performance.now();

    console.log(end - start);
}

main()
~~~

~~~javascript:解消法その２（変数を毎回宣言）.js
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
    const number_of_display = 30000;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        // displayList.push(partOfDataList); // fix on 20220107
        displayList.push({...partOfDataList}) // fix on 20220107
        // displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220107
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
~~~

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
    const number_of_display = 30000;

    const start = performance.now();
    while(i < number_of_display){
        // let partOfDataList = {};
        partOfDataList['title']   = dataList.items[i%5].title;
        partOfDataList['pubDate'] = dataList.items[i%5].pubDate;
        console.log(partOfDataList);
        // displayList.push(partOfDataList); // fix on 20220107
        // displayList.push({...partOfDataList}) // fix on 20220107
        displayList.push(_.cloneDeep(partOfDataList)) // fix on 20220107
        i++;
    }
    const end = performance.now();
    console.log(displayList);

    console.log(end - start);
}

main()
console.log(end - start);
~~~

## 測定結果
30000件での測定結果は以下のようになりました。毎回宣言のほうが少し速そうな感じはしますね。
| Sessions |  ディープコピー  |  変数を毎回宣言  |
| ---- | ---- | ---- |
| 1st |  2076.10  |  1967.60  |
| 2nd |  2061.19  |  1974  |
| 3rd |  2050.19  |   2024.59 |
| 4th |  2100.39  |  2092  |
| 5th |  2130  |  1961.10  |
| Avg. |  2083.57  |  2003.86  |

| Sessions |  ディープコピー  |  変数を毎回宣言  |
| ---- | ---- | ---- |
| 1st |  1994.40  |    |
| 2nd |    |    |
| 3rd |    |    |
| 4th |    |    |
| 5th |    |    |
| Avg. |    |    |

# おしまい
ディープコピーする前に、変数を毎回宣言する方法も検討してみては？

# 参考
https://qiita.com/\_kt15\_/items/0ae5085d61fa5598c76e
https://qiita.com/BlueSilverCat/items/d27a551eb4498d69f7d0
