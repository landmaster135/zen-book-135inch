# はじまり
この前、コーディングしている時にふと気になったのですが、ディープコピーの要否って一体どうなんだろう？と思って、書いた記事になります。

# まず、このコードを。
まず、このコードを実行すると、`displayList`にちゃんと値が入っていないことが確認できます。どうやらシャローコピーになっているようです。
~~~javascirpt
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
    displayList.push([partOfDataList]);
    // displayList.push([partOfDataList].map(list => ({...list})))
    i++;
}

console.log(displayList);
~~~
`console.log(displayList);`の結果
![](https://storage.googleapis.com/zenn-user-upload/21e169d189c2-20220103.png)

そこで、以下の方法で解消を試みました。

# 解消法その１（ディープコピー）
まずは、`push`する際にディープコピーする方法です。値はちゃんと入っていることが確認できます。
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
    // displayList.push([partOfDataList]);
    displayList.push([partOfDataList].map(list => ({...list})))
    i++;
}

console.log(displayList);
~~~
`console.log(displayList);`の結果
![](https://storage.googleapis.com/zenn-user-upload/d72fa46ca30f-20220103.png)

# 解消法その２（変数を毎回宣言）
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
    displayList.push([partOfDataList]);
    // displayList.push([partOfDataList].map(list => ({...list})))
    i++;
}

console.log(displayList);
~~~
`console.log(displayList);`の結果
![](https://storage.googleapis.com/zenn-user-upload/0f929f1a4d21-20220103.png)

# どっちが良いのか？
２通りの方法で出来ましたが、果たしてどちらの方を使っていきましょうか？
僕としては、「変数を毎回宣言」する方法の方が可読性としては勝っていて良さげだなと感じました。
しかし、毎回宣言するのは、速度的に遅そうじゃない？　どうなんだろう？
次は、実行速度を調べてみました。

# 実行速度の調査
次の調査に使うコードはこんな感じ。ざっと、30000件のレコードが入った連想配列の処理速度を計測しました。交互に何回か実行して計測しました。
~~~javascript:解消法その１（ディープコピー）.js
const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';
var rss_data;

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
    // displayList.push([partOfDataList]);
    displayList.push([partOfDataList].map(list => ({...list})))
    i++;
}
const end = performance.now();

console.log(end - start);
~~~

~~~javascript:解消法その２（変数を毎回宣言）.js
const endpoint = 'https://api.rss2json.com/v1/api.json';
const feedUrl = 'https://zenn.dev/kinkinbeer135ml/feed';
var rss_data;

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
    displayList.push([partOfDataList]);
    // displayList.push([partOfDataList].map(list => ({...list})))
    i++;
}
const end = performance.now();

console.log(end - start);
~~~

# 測定結果
測定結果は以下のようになりました。毎回宣言のほうが少し速そうな感じはしますね。
| Sessions |  ディープコピー  |  変数を毎回宣言  |
| ---- | ---- | ---- |
| 1st |  2076.10  |  1967.60  |
| 2nd |  2061.19  |  1974  |
| 3rd |  2050.19  |   2024.59 |
| 4th |  2100.39  |  2092  |
| 5th |  2130  |  1961.10  |
| Avg. |  2083.57  |  2003.86  |

# おしまい
ディープコピーする前に、変数を毎回宣言する方法も検討してみては？

# 参考
https://qiita.com/_kt15_/items/0ae5085d61fa5598c76e
https://qiita.com/BlueSilverCat/items/d27a551eb4498d69f7d0
