---
title: "GASを使ってメールの自動返信システムを作る。〜フォームの回答内容によって送信内容を変更する〜"
emoji: "🌌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GAS", "GoogleAppsScript","自動化","Google"]
published: true
---

# はじめに
前回はこんな記事でGASを使って自動返信システムを作っていきました。
https://zenn.dev/ryotoitoi/articles/3cedb115d816e5

この記事では事前に用意しておいた文章にフォームを回答してもらった内容を反映するような自動返信を作っています。

しかし、この書き方だとできないこともあります。

例えば、
- イベントを複数回予定しており、1つのフォームで集計を行いたい。
- それぞれのイベントで使うZoomのURLは違う。
- 回答してもらったフォームから参加日を判定して、適切なZoomのURLを自動返信したい。

といったことは前回の記事で紹介しているやり方では実現できません。

下の画像のようなフォームで参加希望日を選んでもらってその日に使用するZoomURLを自動返信するイメージです。



![](https://storage.googleapis.com/zenn-user-upload/00c950a5a94f3eb66c79e283.png)

イメージとしては以下のような結果が帰ってくるようなものを作りたいと思います。
（日付やURL、ミーティングIDがちゃんと違っている。）

![](https://storage.googleapis.com/zenn-user-upload/8a8b2b5b5ff8dd33b02dce20.png)
![](https://storage.googleapis.com/zenn-user-upload/46ed49c379616defdf7359e8.png)


次から実際にGASを書いて上のやりたいことを実現できるようにしていきたいと思います!

「とりあえずコードだけ！」という方はこちらへ！
[GASのコード](https://zenn.dev/ryotoitoi/articles/gas-mail-zoom#%E3%81%8F%E3%81%A3%E3%81%A4%E3%81%91%E3%82%8B%E3%81%A8)

# 基本的なGASでの自動返信はこちらで！
基本的なフォームの作成から自動返信システムの作成までの流れはこちらの記事で説明しています。
今回は基本的なことには触れつつも、前回の方法ではできないことをどうやって実装するかの説明に焦点を当てていきと思います。
https://zenn.dev/ryotoitoi/articles/3cedb115d816e5

# 自動返信システムの作成
## フォームの作成
上記の例で挙げたようにイベント参加者を募るためのフォームを作成していく。
今回は簡単化のために以下の質問で構成されているフォームを作成していきます。
- 氏名 (記述)
- メールアドレス（記述）
- 希望参加日時 (プルダウン)

## フォームの回答が記録されるスプレッドシートに追記する
ここから少しずつ前回とは変わってきます。
ここからスプレッドシートを開いて
![](https://storage.googleapis.com/zenn-user-upload/d3e044e1e8e45754e98f44af.png)

次に以下の感じでシートを増やして、追記していきます。
このシートにZoomのURL・開始時間などを事前に入力していきます。
後で詳しく説明しているので、今はまだ何をしているかがわからなくておっけーです。


:::message alert
「フォームの回答1」というシートには手を加えないようにしましょう。
変更していまうとのちのちエラーの原因となる場合があります。
:::

![](https://storage.googleapis.com/zenn-user-upload/572e4c9a4fa9c71708536163.png)

## GASを書いていく
ここからスクリプトエディタを開いてGASを書いていきます。
![](https://storage.googleapis.com/zenn-user-upload/caa1ddc03eb33f57f7d7c20d.png)

### ZoomID記載シートからZoom情報を取得を取得する
まず先程追加したシートから情報を取得するコードを書いていきたいと思います。
こんな感じで書きます。

```js
//ZoomID記載シートからZoom情報を取得
function getZoomInfo(date) {
 const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('ZoomID記載シート');

//ループ対策
 let n = 1;
 while(sheet.getRange(n, 1).getValue()) {
   n++;
   continue;
 }

 for(i=1; i < n; i++) {
  const zoomDate = new Date(sheet.getRange(i, 1).getValue());
  const checkDate = Utilities.formatDate(zoomDate, "JST", "yyyy/MM/dd");
  if(checkDate == date) {
   const url = sheet.getRange(i,2).getValue();
   const id = sheet.getRange(i,3).getValue();
   const pass = sheet.getRange(i,4).getValue();
   const time = sheet.getRange(i,5).getValue();
   const pretime = sheet.getRange(i,6).getValue();
 
   const zoom = {
     ZoomURL: url,
     ZoomID: id,
     ZoomPass: pass,
     ZoomTime: time,
     ZoomPreTime: pretime
     };
     console.log(zoom.ZoomID);
     return zoom;
   }
  }
  return;
 }
```

ここも何をしているかを詳細に理解する必要はありません。

```js
const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('ZoomID記載シート')
```

この部分でシートからの情報を持ってきていて、

```js
const zoom = {
    ZoomURL: url,
    ZoomID: id,
    ZoomPass: pass,
    ZoomTime: time,
    ZoomPreTime: pretime
    };
    console.log(zoom.ZoomID);
    return zoom;
```

この部分であとから使用するものを定義しているという点さえ抑えていただければそれで問題なしです！

### 自動返信を実行する関数を定義する
次に自動返信を行う部分を書いていきたいと思います。
こちらは基本的には前回の記事を参照していただけるとより詳細な理解ができるかと思います。

```js
function onFormSubmit(e) {
  // フォームの回答を取得
  const name = e.namedValues['氏名'][0];
  const email = e.namedValues['メールアドレス'][0];
  const date = e.namedValues['参加希望日'][0];

  const zoom = getZoomInfo(date);
  
  // 自動返信メール件名
  const subject = 'お申し込みありがとうございます！';
      
  // 自動返信メール本文
  const body = name + '様\n' +
    '【当日参加Zoom:URL】\n'+
    '＜日時＞'+ date +'  '+zoom.ZoomTime+'(入室開始'+zoom.ZoomPreTime+'）\n'+
    '\n' +
    '＜ZoomミーティングID＞\n' +
    'トピック: 楽しいイベント\n' +
    '時間: '+ date +'  '+zoom.ZoomTime+'\n' +
    '\n' +
    'Zoomミーティングに参加する\n' +
    zoom.ZoomURL + '\n' +
    'ミーティングID:  '+ zoom.ZoomID +'\n' +
    'パスコード：'+ zoom.ZoomPass +'\n' +
    '\n' +
  // メール送信
  MailApp.sendEmail({
    to: email,
    subject: subject,
    body: body
  });
}
```

ここでは 
```js
zoom.~~
```
となっている部分が前節で定義したものを利用しています。
使いたいタイミングで呼び出しましょう。

```js
MailApp.sendEmail()
```
の部分などがわかならいという場合は前回の記事を参照していただければと思います。

## くっつけると
まとめるとこんな感じのコードになります！
```js
//ZoomID記載シートからZoom情報を取得
function getZoomInfo(date) {
 const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('ZoomID記載シート');

//ループ対策
 let n = 1;
 while(sheet.getRange(n, 1).getValue()) {
   n++;
   continue;
 }

 for(i=1; i < n; i++) {
  const zoomDate = new Date(sheet.getRange(i, 1).getValue());
  const checkDate = Utilities.formatDate(zoomDate, "JST", "yyyy/MM/dd");
  if(checkDate == date) {
   const url = sheet.getRange(i,2).getValue();
   const id = sheet.getRange(i,3).getValue();
   const pass = sheet.getRange(i,4).getValue();
   const time = sheet.getRange(i,5).getValue();
   const pretime = sheet.getRange(i,6).getValue();
 
   const zoom = {
     ZoomURL: url,
     ZoomID: id,
     ZoomPass: pass,
     ZoomTime: time,
     ZoomPreTime: pretime
     };
     console.log(zoom.ZoomID);
     return zoom;
   }
  }
  return;
 }

 function onFormSubmit(e) {
  // フォームの回答を取得
  const name = e.namedValues['氏名'][0];
  const email = e.namedValues['メールアドレス'][0];
  const date = e.namedValues['参加希望日'][0];

  const zoom = getZoomInfo(date);
  
  // 自動返信メール件名
  const subject = 'お申し込みありがとうございます！';
      
  // 自動返信メール本文
  const body = name + '様\n' +
    '【当日参加Zoom:URL】\n'+
    '＜日時＞'+ date +'  '+zoom.ZoomTime+'(入室開始'+zoom.ZoomPreTime+'）\n'+
    '\n' +
    '＜ZoomミーティングID＞\n' +
    'トピック: 楽しいイベント\n' +
    '時間: '+ date +'  '+zoom.ZoomTime+'\n' +
    '\n' +
    'Zoomミーティングに参加する\n' +
    zoom.ZoomURL + '\n' +
    'ミーティングID:  '+ zoom.ZoomID +'\n' +
    'パスコード：'+ zoom.ZoomPass +'\n' +
    '\n' +
  // メール送信
  MailApp.sendEmail({
    to: email,
    subject: subject,
    body: body
  });
}
```

改めてまとめると、
getZoomInfo()とonFormSubmit()という２つの関数があります。
- getZoomInfo()で追記したシートからの情報を持ってきて
- onFormSubmit()で自動返信の処理を行います

コードこれで以上になります！
よくわからない点などあるかもしれませんが試しにコピペしてみるものよいかと思います！

## 追記したシートに事前情報を登録する
以下のように登録していきましょう！
- 日付 - イベント開催日（yyyy/MM/dd)
- URL - ZoomのURL
- ID - ZoomのミーティングID
- Password - Zoomのミーティングパスワード
- time - 開催時刻
- pretime - 入出可能時刻

イメージとしてこんな感じです！
![](https://storage.googleapis.com/zenn-user-upload/3da7b90d04b46f3a26909c6b.png)

## デプロイする
準備は整いました！最後にデプロイして自動返信が帰ってくるようにしましょう！
デプロイの方法はこちらを参照

https://zenn.dev/ryotoitoi/articles/3cedb115d816e5#gas%E3%81%AE%E3%83%88%E3%83%AA%E3%82%AC%E3%83%BC%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%82%92%E3%81%99%E3%82%8B

# 試しに回答してみる
こんな感じで回答してみてどんなメールが帰ってくるか試してみます！
まずは参加希望日を「2021/09/04」にして送ってみます！
![](https://storage.googleapis.com/zenn-user-upload/0c5081548468eb4d9725c533.png)

そうするとこんな感じのメールが帰ってきます！
![](https://storage.googleapis.com/zenn-user-upload/46ed49c379616defdf7359e8.png)

次は参加希望日を「2021/10/18」にして試してみます。
そうするとこんな感じで帰ってきます！
ちゃんと日時やURLやIDが変わって帰ってますね！

![](https://storage.googleapis.com/zenn-user-upload/8a8b2b5b5ff8dd33b02dce20.png)

いい感じに自動返信システムを作ることができました！
お疲れさまでした！

# 最後に
今回はGASを使ってメール業務を自動化していきました。
特に事前に登録しておいた内容をフォームの回答内容によって自動で変えることに焦点を当てていきました！
これでイベントによってフォームを作り変える必要もありません！
退屈なことはGASにやらせよう！
