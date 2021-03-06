---
layout: lib
title: '高速ゼータ変換, 高速メビウス変換, 添字ANDで畳み込み, 添字ORで畳み込み'
permalink: algorithm/FastZetaTransform

---


添字は二進法で捉え，集合として扱う

入門しようという人はページ下部の参考をまず見てほしい

---

高速ゼータ変換・高速メビウス変換は添字の包含関係での畳み込みをする

ゼータ変換とメビウス変換は逆操作の関係にあるが，高速ゼータ・メビウス変換は [隣接代数](https%3A%2F%2Fja.wikipedia.org%2Fwiki%2F%E9%9A%A3%E6%8E%A5%E4%BB%A3%E6%95%B0_%28%E9%A0%86%E5%BA%8F%E7%90%86%E8%AB%96%29){:target="_blank"}<!--_--> というもののなかで，順序として包含関係を用いたものになっているようだ

集合の包含関係以外でもいろいろあり，例えば自然数と約数の関係を用いるとメビウス変換が [メビウスの反転公式]({{ "math/number-theory/mobius-inversion" | absolute_url }}) となる

# 高速ゼータ変換

$a$ から $b$ を求める

### 上位集合に対するゼータ変換

$\displaystyle b\_j = \sum\_{j \subset i} a\_i$

### 下位集合に対するゼータ変換

$\displaystyle b\_j = \sum\_{i \subset j} a\_i$

---


```cpp
// fzt {{"{{"}}{
#include <vector>

// to upper : b[j] = sum(i: j in i, a[i])
// n is power of 2
template < class T >
vector< T > fzt(vector< T > a, bool toUpper) {
  int n = a.size();
  for(int i = 1; i < n; i <<= 1)
    for(int j = 0; j < n; j++)
      if((j & i) == 0) {
        if(toUpper) {
          a[j] += a[j | i];
        } else {
          a[j | i] += a[j];
        }
      }
  return a;
}
// }}}
```


---

`dp[S][i] = Sをi bitまですべて包含している集合の和` というbitDPをしていると考えることができる

必ずしも足し算である必要はなく，`max` などでもいける (FZTは一般にモノイド上で行える)

# 高速メビウス変換

メビウス変換はゼータ変換の逆操作にあたる (ただ逆のことをする，それだけにとどまる)

### 上位集合に対するメビウス変換

$\displaystyle b\_j = \sum\_{j \subset i} (-1)^{\|A \setminus S\|} a\_i$

### 下位集合に対するメビウス変換

$\displaystyle b\_j = \sum\_{i \subset j} (-1)^{\|A \setminus S\|} a\_i$

上記の式は [一般化された包除原理]({{ "math/PIE#一般化された包除原理" | absolute_url }}) そのものである

ゼータ変換の演算として足し算を採用すると上記のようになる，といったことにすぎない

---


```cpp
// fmt {{"{{"}}{
#include <vector>
// to upper : b[j] = sum(i: j in i, (-1)^|i\j| * a[i])
// n is power of 2
template < class T >
vector< T > fmt(vector< T > a, bool toUpper) {
  int n = a.size();
  for(int i = 1; i < n; i <<= 1)
    for(int j = 0; j < n; j++)
      if((j & i) == 0) {
        if(toUpper) {
          a[j] -= a[j | i];
        } else {
          a[j | i] -= a[j];
        }
      }
  return a;
}
// }}}
```


---

少し特殊な式だが，まとめて [包除原理]({{ "math/PIE" | absolute_url }}) を行う，といったことができる

このとき注意すべきは，このときの「包除原理」は上記の「一般化された包除原理」と区別すべきで，メビウス変換は「包除原理」ありきではないこと．あくまで，「包除原理」を「一般化された包除原理」と似た式として使える，と考えるとわかりやすい

また，下記に示す添字AND, ORでの畳み込みに利用できる

---

FZTの一般化と同じように考えると，FMTは逆のことをやっているだけ．ループは逆にしなくていいのか，と思うかもしれないが，これんは対象性がある．すなわち，**"FZTを逆方向にやっていた"** と捉えることで，FMTが正順でも問題ないことが確認できる

この操作は一般的に考えると逆元が必要．しかしその一般化が役に立つかは足し算などに存在するいい性質があるかどうかなどによると思う

# 添字ANDで畳み込み

実数の数列 $a$, $b$ に対し

$$c_k = \sum_{k=i \cap j} a_i b_j$$

を求める. $\|a\| = \|b\| = \|c\| = N$ であり， $N$ は2冪とする

※ 2冪丸めは行わないので自前でやること


```cpp
// n must be 2^k
// require FZT, FMT
// Convolution with AND {{"{{"}}{
#include <vector>
template < class T >
vector< T > convAND(vector< T > a, vector< T > b) {
  a = fzt(a, 1);
  b = fzt(b, 1);
  for(size_t i = 0; i < a.size(); i++) a[i] *= b[i];
  return fmt(a, 1);
}
// }}}
```


下位集合に対するFZTして，項ごとにかけ合わせ，下位集合に対するFMTをしている

これが添字ANDによる畳込みになっていることはすぐに確かめられる

**下位集合に対するFZTして，項ごとにかけ合わせ**たものを考える

これは各項，ある添字について "ANDするとその添字の下位集合になるもの" をかけ合わせたもの を全て足し合わせたもの，になっていることから，FMTをとることでANDがその添字になるものをかけ合わせたものの総和となる

これは環をなす足し算, 掛け算で行うことができる (ていうか畳み込みってそういうものだよね)  
足し算をFZT, FMTに使うとよい

# 添字ORで畳み込み

実数の数列 $a$, $b$ に対し

$$c_k = \sum_{k=i \cup j} a_i b_j$$

を求める. $\|a\| = \|b\| = \|c\| = N$ であり， $N$ は2冪とする

※ 2冪丸めは行わないので自前でやること


```cpp
// n must be 2^k
// require FZT, FMT
//  Convolution with OR {{"{{"}}{
#include <vector>
template < class T >
vector< T > convOR(vector< T > a, vector< T > b) {
  a = fzt(a, 0);
  b = fzt(b, 0);
  for(size_t i = 0; i < a.size(); i++) a[i] *= b[i];
  return fmt(a, 0);
}
// }}}
```


このプログラムが正しいことはANDと同様に示せる

# 練習問題

高速ゼータ変換

* [E - Or Plus Max (700) - AtCoder](https://beta.atcoder.jp/contests/arc100/tasks/arc100_c){:target="_blank"}<!--_-->

高速メビウス変換

* なし

# 参考

* [高速ゼータ変換 - とどの日記](http://d.hatena.ne.jp/todo314/touch/20120614/1339695202){:target="_blank"}<!--_-->
* [色々な畳み込み - kazuma8128’s blog](http://kazuma8128.hatenablog.com/entry/2018/05/31/144519){:target="_blank"}<!--_-->

