---
title: "GASを使ってメールの自動返信システムを作る。回答によって送信内容を変更する。"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# はじめに
前回はこんな記事でGASを使って自動返信システムを作っていきました。
https://zenn.dev/ryotoitoi/articles/3cedb115d816e5

この記事では事前に用意しておいた文章にフォームを回答してもらった内容を反映するような自動返信を作っています。

しかし、この書き方だとできないこともあります。

例えば、
- イベントを複数回予定していて1つのフォームで集計を行いたい。
- それぞれのイベントで使うZoomのURLは違う。
- 回答してもらったフォームから参加日を判定して、適切なZoomのURLを自動返信したい。

といったことは前回の記事で紹介しているやり方では実現できません。

下の画像のようなフォームで参加希望日を選んでもらってその日に使用するZoomURLを自動返信するイメージです。

![](https://storage.googleapis.com/zenn-user-upload/00c950a5a94f3eb66c79e283.png)

次から実際にGASを書いて上のやりたいことを実現できるようにしていきたいと思います。

# 基本的なGASでの自動返信はこちらで！
基本的なフォームの作成から自動返信システムの作成までの流れはこちらの記事で説明していますので今回はどうやって実装するかの説明に焦点を当てたいと思います。
https://zenn.dev/ryotoitoi/articles/3cedb115d816e5

# 自動返信システムの作成
## フォームの作成
上記の例で挙げたようにイベント参加者を募るためのフォームを作成していく。
今回は簡単化のために以下の質問で構成されているフォームを作成していきます。
- 氏名 (記述)
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

まず先程追加したシートから情報を取得するコードを書いていきたいと思います。

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