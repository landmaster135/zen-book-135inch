# はじまり
以前にkindleの蔵書のタイトル一覧を読み取るツールを作ったのですが画像も欲しくなり、それを作るまでの記事になります。以前の記事はこれ。
https://zenn.dev/kinkinbeer135ml/articles/1500f99b37aece

# Chrome Extension APIは断念・・・
まず、参考にしたソースが、この拡張機能です。
https://chrome.google.com/webstore/detail/image-downloader/cnpniohnfphhjihaiiggeabnkjhpaldj

機能は正に僕がやろうとしていることなので、その中のソースを見てみると、`chrome.downloads.download`という記述があったので調べてみると、どうやら **Chrome Extension API** とかいうAPIを使っている模様。色々調べてみて、`"permissions"`に`"downloads"`を入れなきゃとか色々書いてある模様。
しかし、とりあえず、Chrome拡張機能として実行しないと、 **Chrome Extension API自体** を使用できない。という事実は判明しました。。。そこまで大層なものを作る気はないので、このAPIを使うのは辞めました。
通常のJavascriptを打っても動かないというのはこの記事に書いてありました。
https://note.com/taraco123/n/n95a5516168f5

# Googleドライブに突っ込む。→GASでダウンロードするのは断念・・・
次に、ローカルに落とすのではなく、Googleドライブに落とそうと思い、Google Apps ScriptでURL読み込んでダウンロード出来ないかと思い、`Parse`で試してみました。
しかし、ページを読み込んで下に行かないと、全ての蔵書が表示されないので、GASからDOM内のURLを読み取ってダウンロードする方法は断念・・・。（Cypressみたいなツールがあるから自動化できそうだけど、今回は保留。。。）

# Googleドライブに突っ込む。開発者ツールで取得した結果をスプシに貼り付ける。
今回はこの方法に落ち着きました。
開発者ツールで叩けば、DOMのURLは取ってこれるので、そのURLをGoogleスプシに貼り付けて、そのURLからGoogleドライブに画像をダウンロードするという感じです。
![](https://storage.googleapis.com/zenn-user-upload/9351d0e3ce2f-20211226.png)
↑ここから取得します。

~~~javascript
function listFormated(listReadFromGss) {
  var listFormated = [];
  for (let j = 0; j < listReadFromGss.length; j++) {
    // "if" statement in one liner. If '', nothing to do.
    listReadFromGss[j][0]=='' ? true : listFormated.push(listReadFromGss[j][0]);
  }
  return listFormated
}

/**
 * Get number of record in Google Spreadsheet.
 *
 * @param {"bookmarkSites"} sheetName - Name of sheet that you wanna know number of record.
 * @return {number} Number of record
 * @customfunction
 */
function get_row_to_read_actual_in_GSS(sheetName) {
  // declare list for warning message.
  var warningMessage = 'Warning: Number of row passing over \"row_to_read\". Tweak me.';
  var errorMessage   = 'RowIndexOutOfBoundsError: Number of row reached \"row_to_read\". Tweak me.';

  // declare variables for row and column index.
  var column_for_id = 1;
  var row_to_read   = 500;
  var row_to_read_actual;

  // declare list.
  var idList;

  // get sheet.
  var ss = SpreadsheetApp.getActive();
  var sheet = ss.getSheetByName(sheetName);

  // memorize number of row to read
  console.time(`SELECT TOP ${row_to_read - 1} id FROM \'${sheetName}\'`);
  idList = sheet.getRange(2, column_for_id, row_to_read - 1, 1).getValues();
  console.timeEnd(`SELECT TOP ${row_to_read - 1} id FROM \'${sheetName}\'`);
  idList_formated = listFormated(idList);
  
  // warning message. If condition is false, nothing to do.
  row_to_read_actual = Number(idList_formated.reduce((a,b)=>Math.max(a,b)));
  row_to_read - row_to_read_actual <= 2 ? console.warn(warningMessage) : false;
  if(row_to_read_actual >= row_to_read - 1){
    console.error(errorMessage);
    return 0;
  }
  return row_to_read_actual;
}

function saveImageFromUrl(){  
  // get sheet.
  var ss = SpreadsheetApp.getActive();
  var sheet = ss.getSheetByName(sheetName3rd);
  var row_to_read_actual = get_row_to_read_actual_in_GSS(sheetName3rd);
  var bookList = [];

  console.time(`SELECT TOP ${row_to_read_actual - 1} * FROM \'${sheetName3rd}\'`);
  bookList = sheet.getRange(2, column_for_tempGet_of_id, row_to_read_actual,column_for_tempGet_of_fromDlImageUrl).getValues();
  console.timeEnd(`SELECT TOP ${row_to_read_actual - 1} * FROM \'${sheetName3rd}\'`);

  let image;
  let outputFileId;
  let outputFolder = DriveApp.getFolderById(imageFolderId);
  let outputFolderName = outputFolder.getName();
  let outputFileIdList = [];

  console.time(`Files were Created in \'${outputFolderName}\'. `);
  bookList.forEach(row => {
    image = UrlFetchApp.fetch(row[column_for_tempGet_of_fromDlImageUrl - gap_between_arrayIndex_and_sheetRow]).getBlob();
    outputFileId = outputFolder.createFile(image).getId();
    console.info(`File was Cradted to \'${outputFileId}\'.`);
    outputFileIdList.push([outputFileId]);
  });
  console.timeEnd(`Files were Created in \'${outputFolderName}\'. `);

  console.time(`UPDATE \'${sheetName3rd}\' SET toDlImageUrl *`);
  bookList = sheet.getRange(2, column_for_tempGet_of_toDlImageUrl, row_to_read_actual, 1).setValues();
  console.timeEnd(`UPDATE \'${sheetName3rd}\' SET toDlImageUrl *`);
}
~~~

# 実行結果
こんな風にGoogleドライブに落とすことができました。
![](https://storage.googleapis.com/zenn-user-upload/c4ef0df373b1-20211226.png)

ファイルを作るのに掛かった時間は、207ファイルあって277秒掛かりました。1ファイルあたり1.3秒位掛かっているようでした。
![](https://storage.googleapis.com/zenn-user-upload/d78d6228b89b-20211226.png)

# おしまい
手作業が入る部分がありますが、画像ファイルを一括でダウンロードする方法は確立できました。これで本の管理アプリを作れそうだ。
