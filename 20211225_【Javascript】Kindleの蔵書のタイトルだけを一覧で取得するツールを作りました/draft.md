# はじまり
Kindleにある本でどんな本が面白かったかを管理したいと思い、タイトルだけを取得するツールを作りました。なんか、Web Database APIを使う方法が出来ず、PCにインストールしてXMLを使う方法も面倒くさかったので、タイトルだけ取得したい方はご参考いただければと思います。

# ツール実行後の動き
これを[Kindle Cloud Reader](http://qiita.comhttps://read.amazon.co.jp/)で開発者ツールのコンソールで動かすと、下の画像のように蔵書のタイトルだけを全て読み取ってクリップボードにコピーしてくれます。（使う前に全ての蔵書を画面に表示させる必要はありますので、下へスクロールし切ってから使って下さいませ。）
![](https://storage.googleapis.com/zenn-user-upload/a1552cb6db1a-20211226.png)

# 実際のソース
~~~javascript
let bookCount  = 500;
let array      = [];
let targetTag  = 'p';
let i          = 0;
let resultText = '';
let isTypeErrorOccured    = false;
let countTypeErrorOccured = 0;
try {
    for (i = 0; i < bookCount; i++) {
        array.push(`${[...document.querySelectorAll(targetTag)][i * 2].innerText}`);
    }
}catch(e){
    if (e instanceof TypeError) {
        isTypeErrorOccured    = true;
        countTypeErrorOccured = i;
    }else{
        console.log(e)
    }
}
resultText = getTextFromArray(array);
console.log(resultText);
copyToClipboard(resultText);
if(isTypeErrorOccured){
    console.log(`TypeError occured. bookCount is ${countTypeErrorOccured}. That's all.`);
}else{
    console.log(`TypeError didn't occur. Maybe, bookCount is over ${bookCount}. Change 'bookCount' to reget.`);
}

function getTextFromArray(array){
    var sep = "\n";
    var result = ``;
    array.forEach(function(value) {
        result += value;
        result += `${sep}`;
    });
    return result;
}

function copyToClipboard(content){
    // Create a dummy textarea to copy the string array inside it
    var dummy = document.createElement("textarea");
    document.body.appendChild(dummy);
    dummy.setAttribute("id", "dummy_id");
    document.getElementById("dummy_id").value = content;
    dummy.select();
    document.execCommand("copy");
    document.body.removeChild(dummy);
}
~~~

# 処理の流れ
処理の流れは、
1. 蔵書一覧を読み取って配列に入れる。ここでエラーが発生していたら、全ての蔵書が読み取れています。（即興で作ったので。。。）
2. 配列を文字列にする。
3. 文字列をクリップボードにコピーする。ダミーの`textarea`を作って、その中に入れた文字列をコピーしています。

# おしまい
Kindle蔵書のタイトルだけ一覧を読み取れました。
でも、画像も欲しいと思ってしまった。それはまた今度作ろう。
