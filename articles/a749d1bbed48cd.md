---
title: "【メモ】Macでインターネットアカウントを追加する時、無限リロードで沼ったら"
emoji: "🐡"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["mac"]
published: true
---
# Macでカウントを追加したいのに無限リロードになった時の対処法
Macで、GmailやGoogleカレンダーなどを連携させたい時にはSystem Preferencesの
Internet Accountsから新しいアカウントの追加を行う。
本当は、それだけでいいのですが、下図のような状況になってしまう自体が発生してしまうことがあります。
謎の無限リロード…

![image_01](/images/mac_internet_account/image_01.png)

# 原因と解決方法
## 原因
デフォルトブラウザを「Brave」に設定していることが原因でした。

## 解決方法
デフォルトブラウザを「Chrome」に設定すると解決です。\
デフォルトブラウザはSystem Preference・GeneralのDefault web browserから変更する。