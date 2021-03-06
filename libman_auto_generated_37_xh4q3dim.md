---
layout: lib
title: 二頂点対最短路問題 (A*)
permalink: graph/A-star

---

<!--*-->

A starと読む

重み付き有向グラフの二頂点対最短路を求める

dijkstraの高速化という感じだが，dijkstraと違って2頂点の間の最短路しかわからない (st最短路, 以下 $s$ から $t$ の最短路を求めるとする)

計算量や特徴はdijkstraと同じ

priority queueを使えば $O((E + V) \log V)$

---

競プロでの出番はまずなさそう

高速化でしか無いので [RadixHeap]({{ "data-structure/Heap/RadixHeap" | absolute_url }}) とかと組み合わせるといいかもしれない

---

ふつうのdijkstraのキーに頂点$v$からゴール$t$までの推定値 $h^\*(v)$ を実際の距離に足したものをキーとしてもつ

ただし $h^\*(v)$ は非負

### 許容的

すべての$v$について，真のゴールまでの距離（最短距離）を $h(v)$ としたとき，
$0 \leq h^\*(v) \leq h(v)$ が成り立つなら $h^\*$ は許容的 (Admissible) であるという

つまり，$h^\*$ が許容的なとき，真の距離の下界になっている

$h^\*$ が許容的なとき，A\*の返す経路は最短経路となる

#### 証明のようなもの

$h^\*(t) = 0$ であるので終了点として判断される状態のキー (キュー内のね) はすべて真の経路長

また，真の最短路を形成する経路のキーはすべて最短距離以下となる ($d + h^\*(v) \leq d + h(v) = \text{真のその状態から作れる最短距離}$, $d$はその状態に至るまでの真の距離, こんな書き方をしたのは2番目以降の最短路でも成り立ちそうだから)

よって，終了点として判断されうる経路の真の経路長が最短距離より大きいなら，最短経路を形成する経路の計算が先に必ず行われる

---

多分こんな感じだと思っている (自分で考えたので間違っていたら教えてください)

二番目以降の最短路についても，枝刈りを行うことはできなくなりますが (実装を参照) ，同様の議論で求めることが可能なのかと思います (練習問題の「K番目の閉路」, 解説はもっと限定的な状況での証明をしている)

### 無矛盾

すべての辺 $(u, v)$ について $cost(u, v) + h^\*(v) - h^\*(u) \geq 0$ なら $h^\*$ は無矛盾 (consistent) であるという

無矛盾であるとき，許容的である

#### 証明

わからん

「Artificial intelligence: a modern approach」にあるらしい

# 実装

$h^\*$が許容的であるときのグラフのst最短路の長さを求めるものです


```cpp
const ll inf = 1e18;
// A* {{"{{"}}{
{
  using P = tuple< ll, ll, int >;
  priority_queue< P, vector< P >, greater< P > > pq;
  vector< ll > dist(n, inf);
  pq.emplace(0, 0, s);
  ll res = inf;
  dist[i] = 0;
  while(pq.size()) {
    ll di, rdi;
    int i;
    tie(di, rdi, i) = pq.top();
    pq.pop();
    if(i == t) {
      res = rdi;
      break;
    }
    if(dist[i] < di) continue;
    for(auto to : g[i]) {
      int j, co;
      tie(j, co) = to;
      if(hstar(j) == inf) continue;
      ll nrdi = rdi + co;
      if(dist[i] > nrdi) pq.emplace(nrdi + hstar(j), nrdi, j);
    }
  }
}
// }}}
```


# 練習問題

* [UTPC2013 J - K番目の閉路](https://beta.atcoder.jp/contests/utpc2013/tasks/utpc2013_10){:target="_blank"}<!--_-->

# 参考

* [A\* - Wikipedia](https://ja.wikipedia.org/wiki/A*){:target="_blank"}<!--_-->

