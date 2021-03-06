---
layout: lib
title: きたまさ法
permalink: algorithm/Kitamasa

---


$ K + 1 $ 項間の漸化式と $a_0, \cdots , a_{K-1}$ が与えられたとき， $a_N$ を $O(K^2 \log N)$ で求めます．

高速きたまさ法はこれをさらに高速化し，$O(K \log K \log N)$ で求めます．

コンパニオン行列を用いれば $O(K^3 \log N)$ で求めることができますが，コンパニオン行列を用いた方法は蟻本にあるのでそこまでの導入は省きます．

また，完全な導入は少なめで，きたまさ法の導出をメインに書きます．備忘録程度に．

数列と多項式は密接な関係があるのですが，ここでは数列で表示していきます．

また，数列を $c = (c_i)_{i=0}^{K-1}$ みたいに書いてみました．

# 前提

$c = (c_i)\_{i=0}^{K-1}$ , $a\_0, \cdots , a\_{K-1}$ が与えられる．

任意の整数 $n$ について，

$$
\begin{aligned}
&a_{n+K} &&= c_0a_n + c_1a_{n+1} + \cdots + c_{K-1}a_{n+K-1} \\
&&&= \sum_{i=0}^{K-1} c_ia_{n+i}
\end{aligned}
$$

が成り立つこと．

# 補題1

任意の $n, m, l$ について，  
ある $s = (s_i)_{i=0}^{K-1}$ が存在し，

$$  
a_n = \sum_{i=0}^{K-1}s_ia_{l+i}  
$$

を満たす**ならば**

$$  
a_{n+\textcolor{red}m} = \sum_{i=0}^{K-1}s_ia_{l+\textcolor{red}m+i}  
$$

が成り立つ．

# 補題1の説明

前者を $a_0$ が基準である数列であると捉えると，  
後者は $a_m$ が基準である数列と考えることができます．

K項間フィボナッチ数列は任意の整数 $m$ ずらしても同じ関係を保ちます．

この事自体を証明するのはちょっとわかりませんでした（むずかしそう

直感的に自明だけど，きたまさ法において超重要なこと，というか本質です．

# 任意の要素を初項の線型結合で表す

初項はここでは $a_0, \cdots, a_{K-1}$ のことです．

$a_m$ を考えます．

$a_m = \sum_{i=0}^{K-1} b_{m,i} \cdot a_i$ と表すことができます．

このことは簡単で，

$0 \le m \lt K$ の場合は $b_{m, m} = 1$ 他は $0$ とすれば成り立ちます．（ん？具体例を示します

* $b_0 = \\{1, 0, 0, \cdots\\}$
* $b_1 = \\{0, 1, 0, \cdots\\}$
* $b_2 = \\{0, 0, 1, \cdots\\}$
* $\cdots$

となります．

$m \geq K$であれば，前提を $n=m$, $n=m-1$, ... と減らしながら適用していくことで，初項の線型結合で表すことができます．  
($m \lt 0$ の場合も同様です)

さて，きたまさ法の本質は $b_m = (b_{m, i})_{i=0}^{K-1}$ を求めることです．

$b_u$ と $b_v$ が求まっているとします．

すなわち，

$(1)\ a_u = \sum_{i=0}^{K-1} b_{u,i} \cdot a_i$

$(2)\ a_v = \sum_{j=0}^{K-1} b_{v,j} \cdot a_j$

が分かっているとします．

ここで，補題1を $(1)$ に， $n = u, m = v, l = 0, s = b_u$ として適用します．

$(3)\ a_{u + v} = \sum_{i=0}^{K-1} b_{u, i} \cdot a_{i + v}$

となります．（ここまではいいかな

つぎに， $a_{i+v}$ が含まれているので，補題1を上記の $(2)$ に $n = v, m = i, l = 0, s = b_v$ として適用すると，

$(4)\ a_{i + v} = \sum_{j=0}^{K-1} b_{v, j} \cdot a_{i + j}$

が導かれます．$(4)$ を $(3)$ に代入して，

$$
\begin{aligned}
&(5)\ a_{u + v} &&= \sum_{i=0}^{K-1} b_{u, i} \sum_{j=0}^{K-1} b_{v, j} \cdot a_{i+j} \\
&&&= \sum_{i=0}^{K-1}\sum_{j=0}^{K-1} b_{u, i} \cdot b_{v, j} \cdot a_{i+j}
\end{aligned}
$$

が得られます．さて， $b_ {u + v} = (b_ {u+v,i})_ {i=0}^{K-1}$ が求めたいのですが，  
このままでは係数が $a_0$ から $a_ {2K-2}$ のものになってしまいます．

しかしこれは，前提を $n = {2K-2}, n = {2K-3}, \cdots, n = K$ と適用していくことで，  
$a_{u + v}$ を 初項の線型結合で表すことができます．

上記は $a_{u+v}$ の式なのにどうやって $b_{u+v}$ を求めるんだ，と思っている人が居るかもしれませんが，実装を見てもらえればわかると思います．

これで $b_u$ と $b_v$ から $b_{u+v}$ が得られました．

# 繰り返し二乗法

あとは簡単で，$0$と$1$を足して$N$ を作りなさい，という問題と変わりません．

$b_0 = \\{1, 0, \cdots\\}$ と $b_1 = \\{0,1, \cdots\\}$ は簡単に作れますね．

$b_2$は$u=1,v=1$として(2)を適用すれば良いです．  
同様に，$b_4, b_8, \cdots$ を $O(\log N)$ 個計算すれば，  
今度は $b_0$ に適用していくことで $b_N$ が求まり， $a_N$ は前提に当てはめれば求まります．

# 実装


```cpp
// kitamasa(c, u, v) {{"{{"}}{
#include <vector>
template < class T >
std::vector< T > kitamasa(const std::vector< T > &c, const std::vector< T > &u,
                          const std::vector< T > &v) {
  using size_type = std::size_t;
  size_type k = c.size();
  std::vector< T > r(2 * k - 1);
  for(size_type i = 0; i < k; i++)
    for(size_type j = 0; j < k; j++) r[i + j] += u[i] * v[j];
  for(size_type i = 2 * k - 2; i >= k; i--)
    for(size_type j = 0; j < k; j++) r[i - k + j] += r[i] * c[j];
  r.resize(k);
  return r;
}
// }}}
```


# 検証

* [T - フィボナッチ - TDPC](https://beta.atcoder.jp/contests/tdpc/submissions/3179115){:target="_blank"}<!--_-->

# 練習

* [T - フィボナッチ - TDPC](https://beta.atcoder.jp/contests/tdpc/tasks/tdpc_fibonacci){:target="_blank"}<!--_-->
* [No.214 素数サイコロと合成数サイコロ (3-Medium) - yukicoder](https://yukicoder.me/problems/no/214){:target="_blank"}<!--_-->
  * [僕の提出](https://yukicoder.me/submissions/284598){:target="_blank"}<!--_-->

