---
layout: lib
title: 木のダブリング
permalink: graph/DoublingTree

---


ダブリング自体は木でなくともできます．(数列とか)

$O(\log N)$ で1クエリを処理でき，[HL分解]({{ "graph/HL-Decomposition" | absolute_url }})より時間計算量で有利ですが，  
空間計算量は $O(N \log N)$ とちょっと増えます
[HL分解]({{ "graph/HL-Decomposition" | absolute_url }})とちがって，  
遅延させたりもできず，通常のセグ木でできることができます


```cpp
// DoublingTree ( <tree> , initial? )
// .addEdge(a, b)
// .set(i, val) or .assign( <data> )
// === initiation ===
// when single tree : .build(root = 0)
// when forest      : .dfs(roots) & .init()
// === query ===
// .lca(a, b)
// .query(hi, a, hi_inclusive = true)
// .climb(from, steps)
// .descend(from, to, steps)
// === --- ===
// .depth[a]
// .par[i][a] // climb 2^i times from [a]
// .dat[i][a] // apply to 2^i edges from [a] to ancestor
/// --- Doubilng Tree {{"{{"}}{ ///
#include <cassert>
#include <iterator>
#include <vector>
template < class Monoid >
struct DoublingTree {
  using T = typename Monoid::T;
  size_t n;
  int logn;
  vector< vector< int > > tree;
  vector< int > depth; // 0-indexed
  // [logn][n]
  vector< vector< int > > par;
  vector< vector< T > > dat;
  int log(int n) {
    int h = 1;
    while((1 << h) < n) h++;
    return h;
  }
  DoublingTree() : n(0) {}
  DoublingTree(size_t n, const T &initial = Monoid::identity())
      : n(n),
        logn(log(n)),
        tree(n),
        depth(n),
        par(logn, vector< int >(n)),
        dat(logn, vector< T >(n, initial)) {}
  template < class InputIter, class = typename iterator_traits< InputIter >::value_type >
  DoublingTree(InputIter first, InputIter last, const T &initial = Monoid::identity())
      : DoublingTree(distance(first, last), initial) {
    copy(first, last, begin(tree));
  }
  DoublingTree(const vector< vector< int > > &tree, const T &initial = Monoid::identity())
      : DoublingTree(begin(tree), end(tree), initial) {}
  void addEdge(size_t a, size_t b) {
    assert(a < n && b < n);
    tree[a].push_back(b);
    tree[b].push_back(a);
  }
  void set(size_t i, const T &val) {
    assert(i < n);
    dat[0][i] = val;
  }
  template < class InputIter, class = typename iterator_traits< InputIter >::value_type >
  void assign(InputIter first, InputIter last) {
    assert(distance(first, last) <= n);
    copy(first, last, begin(dat[0]));
  }
  template < class T >
  void build(const vector< T > &roots) {
    for(T &root : roots) dfs(root);
    init();
  }
  void build(size_t root = 0) {
    assert(root < n);
    dfs(root);
    init();
  }
  bool initiated = 0;
  void init() {
    assert(!initiated);
    initiated = 1;
    for(int k = 1; k < logn; k++) {
      for(size_t i = 0; i < n; i++) {
        int p = par[k - 1][i];
        if(p == -1) {
          par[k][i] = -1;
          continue;
        }
        dat[k][i] = Monoid::op(dat[k - 1][p], dat[k - 1][i]);
        par[k][i] = par[k - 1][p];
      }
    }
  }
  void dfs(size_t i, int p = -1, int d = 0) {
    assert(i < n);
    depth[i] = d;
    par[0][i] = p;
    for(int j : tree[i])
      if(j != p) {
        dfs(j, i, d + 1);
      }
  }
  int climb(size_t a, ll steps) {
    assert(initiated);
    assert(a < n);
    for(int i = logn - 1; i >= 0 && a != -1; i--)
      if(steps >= (1 << i)) a = par[i][a], steps -= 1 << i;
    assert(a == -1 || steps == 0);
    return a;
  }
  int descend(size_t from, size_t to, ll steps = 1) {
    assert(initiated);
    assert(from < n && to < n);
    assert(depth[to] >= depth[from]);
    int up = depth[to] - depth[from] - steps;
    if(up < 0) up = 0;
    return climb(to, up);
  }
  int steps(size_t from, size_t to) {
    assert(initiated);
    assert(from < n && to < n);
    return depth[from] + depth[to] - depth[lca(from, to)] * 2;
  }
  int lca(size_t a, size_t b) {
    assert(initiated);
    assert(a < n && b < n);
    if(depth[a] < depth[b]) swap(a, b);
    for(int k = logn - 1; k >= 0; k--) {
      int na = par[k][a];
      if(na == -1 || depth[na] < depth[b]) continue;
      a = na;
    }
    if(a == b) return a;
    for(int k = logn - 1; k >= 0; k--) {
      int na = par[k][a];
      int nb = par[k][b];
      if(na == nb) continue;
      a = na, b = nb;
    }
    return par[0][a];
  }
  T query(size_t hi, size_t a, bool inclusive = true) {
    assert(initiated);
    assert(hi < n && a < n);
    T res = Monoid::identity();
    for(int k = logn - 1; k >= 0; k--) {
      int na = par[k][a];
      if(na == -1 || depth[na] < depth[hi]) continue;
      res = Monoid::op(dat[k][a], res);
      a = na;
    }
    if(inclusive) res = Monoid::op(dat[0][hi], res);
    return res;
  }
  int size() { return n; }
};
/// }}}--- ///

/// --- Monoid examples {{"{{"}}{ ///
constexpr long long inf_monoid = 1e18 + 100;
#include <algorithm>
struct Nothing {
  using T = char;
  using Monoid = Nothing;
  using M = T;
  static constexpr T op(const T &, const T &) { return T(); }
  static constexpr T identity() { return T(); }
  template < class X >
  static constexpr X actInto(const M &, ll, ll, const X &x) {
    return x;
  }
};

template < class U = long long >
struct RangeMin {
  using T = U;
  static T op(const T &a, const T &b) { return min(a, b); }
  static constexpr T identity() { return T(inf_monoid); }
};

template < class U = long long >
struct RangeMax {
  using T = U;
  static T op(const T &a, const T &b) { return max(a, b); }
  static constexpr T identity() { return -T(inf_monoid); }
};

template < class U = long long >
struct RangeSum {
  using T = U;
  static T op(const T &a, const T &b) { return a + b; }
  static constexpr T identity() { return T(0); }
};

template < class U >
struct RangeProd {
  using T = U;
  static T op(const T &a, const T &b) { return a * b; }
  static constexpr T identity() { return T(1); }
};

template < class U = long long >
struct RangeOr {
  using T = U;
  static T op(const T &a, const T &b) { return a | b; }
  static constexpr T identity() { return T(0); }
};

#include <bitset>

template < class U = long long >
struct RangeAnd {
  using T = U;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return T(-1); }
};

template < size_t N >
struct RangeAnd< bitset< N > > {
  using T = bitset< N >;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return bitset< N >().set(); }
};

/// }}}--- ///

using DBL = DoublingTree< RangeSum<> >;
```


# 検証

* [D - 旅行会社高橋君 - AtCoder](https://atcoder.jp/contests/arc039/submissions/2136670){:target="_blank"}<!--_-->
* HackerRankの問題でやろうとしたらMLEった

# 練習問題

* [E - Wandering TKHS (1200) - AtCoder](https://atcoder.jp/contests/agc029/tasks/agc029_e){:target="_blank"}<!--_-->

