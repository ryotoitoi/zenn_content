---
title: "【個人用メモ】Zennの記事内で数式のイコールを揃える"
emoji: "🔖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["math", "数式"]
published: true
---
n番煎じかもしれないが毎回調べるのも面倒なので自分用にまとめておく。

# 手順
1. `$$ \begin{aligned} ~~~ \end{aligned} $$`の中に数式を書く。
1. 改行は`\\`、イコールは`&=`で書く。

## 例
T-Learnerの数式
```
$$
\begin{aligned}
\hat\tau &= E [Y(1) - Y(0)|X = x] \\
&= E [Y(1)|X=x] - E[Y(0)|X = x]  \\
&= \hat\mu_{1}(x) - \hat\mu_{0}(x)
\end{aligned}
$$
```

## 出力
$$
\begin{aligned}
\hat\tau &= E [Y(1) - Y(0)|X = x] \\
&= E [Y(1)|X=x] - E[Y(0)|X = x]  \\
&= \hat\mu_{1}(x) - \hat\mu_{0}(x)
\end{aligned}
$$

## まとめ
普段は`LaTeX`を使用して数式を書いているためZennで採用されている`KaTeX`とごっちゃになるためにまとめた。

# LaTeXとKaTexで違ったところ
LaTeX → KaTex
- `\begin{eqnarray*}` → `\begin{aligned}`
- `&=&` → `&=`

改行のやり方は一緒