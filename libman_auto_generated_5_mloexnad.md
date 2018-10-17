---
layout: lib
title: フラクショナルカスケーディング
permalink: data-structure/SegmentTree/FractionalCascadingSegmentTree

---



```cpp
// constructor(H)
// index(y, x)
// === init(doUnique) ===
// set(y, x, val)         // index(y, x) must be done
// query(yl, yr, xl, xr)
// === --- ===
// only offline
/// --- Fractional Cascading SegmentTree Library {{"{{"}}{ ///

#include <algorithm>
#include <functional>
#include <vector>
template < class T, class U, class Index = ll >
struct FractionalCascadingSegTree {
  int h;
  vector< T > dat;
  vector< vector< Index > > indices;
  vector< vector< int > > L, R;
  U identity;
  function< void(T &, int x, const U &) > setX;
  function< void(T &, vector< Index > &) > initX;
  function< U(T &, int x1, int x2) > queryX;
  function< U(const U &, const U &) > mergeY;
  FractionalCascadingSegTree() {}
  FractionalCascadingSegTree(
      int tempH, //
      function< void(T &, int, const U &) > const &setX,
      function< void(T &, vector< Index > &) > const &initX,
      function< U(T &, int, int) > const &queryX,
      function< U(const U &, const U &) > const &mergeY, U identity = U(),
      T initial = T())
      : identity(identity),
        setX(setX),
        initX(initX),
        queryX(queryX),
        mergeY(mergeY) {
    h = 1;
    while(h < tempH) h <<= 1;
    dat = vector< T >(2 * h, initial);
    indices = vector< vector< Index > >(2 * h);
    L = R = vector< vector< int > >(2 * h);
  }
  void index(int i, Index j) {
    assert(0 <= i && i < h);
    indices[i + h - 1].emplace_back(j);
  }
  void init(bool doUnique) {
    for(int i = h * 2 - 2; i >= 0; i--) {
      if(i >= h - 1) {
        sort(begin(indices[i]), end(indices[i]));
        if(doUnique)
          indices[i].erase(
              unique(begin(indices[i]), end(indices[i])), end(indices[i]));
        initX(dat[i], indices[i]);
        continue;
      }
      size_t lsz = indices[i * 2 + 1].size();
      size_t rsz = indices[i * 2 + 2].size();
      size_t nsz = lsz + rsz;
      indices[i].resize(nsz);
      L[i].resize(nsz + 1, lsz);
      R[i].resize(nsz + 1, rsz);
      size_t p1 = 0, p2 = 0;
      while(p1 < lsz || p2 < rsz) {
        L[i][p1 + p2] = p1;
        R[i][p1 + p2] = p2;
        if(p1 < lsz &&
           (p2 == rsz || indices[i * 2 + 1][p1] <= indices[i * 2 + 2][p2])) {
          indices[i][p1 + p2] = indices[i * 2 + 1][p1];
          p1++;
        } else {
          indices[i][p1 + p2] = indices[i * 2 + 2][p2];
          p2++;
        }
      }
      initX(dat[i], indices[i]);
    }
  }
  void set(int y, Index x, const U &val) {
    int lower =
        lower_bound(begin(indices[0]), end(indices[0]), x) - begin(indices[0]);
    set(y, lower, val, 0, h, 0);
  }
  void set(int y, int lower, U const &val, int l, int r, int k) {
    if(y + 1 <= l || r <= y) return;
    setX(dat[k], lower, val);
    if(y <= l && r <= y + 1) return;
    set(y, L[k][lower], val, l, (l + r) >> 1, k * 2 + 1);
    set(y, R[k][lower], val, (l + r) >> 1, r, k * 2 + 2);
  }
  U query(int a, int b, Index l, Index r) {
    if(a >= b || l >= r) return identity;
    int lower =
        lower_bound(begin(indices[0]), end(indices[0]), l) - begin(indices[0]);
    int upper =
        lower_bound(begin(indices[0]), end(indices[0]), r) - begin(indices[0]);
    return query(a, b, lower, upper, 0, h, 0);
  }
  U query(int a, int b, int lower, int upper, int l, int r, int k) {
    if(lower == upper) return identity;
    if(b <= l || r <= a) return identity;
    if(a <= l && r <= b) return queryX(dat[k], lower, upper);
    return mergeY(
        query(a, b, L[k][lower], L[k][upper], l, (l + r) >> 1, k * 2 + 1),
        query(a, b, R[k][lower], R[k][upper], (l + r) >> 1, r, k * 2 + 2));
  }
};

/// }}}--- ///

// never forget to init SparseTable by yourself
// FC-SegmentTree Example {{"{{"}}{

using Under = SparseTable< RMQSL >;
using Value = RMQSL;
using Data = Value::T;

FractionalCascadingSegTree< Under, Data > ecas(
    N + 1,
    // set x
    [](Under &dat, int x, const Data &val) -> void {
      dat.set(x, Value::op(dat.get(x), val));
    },
    // init x
    [](Under &dat, const vector< ll > &indices) -> void {
      dat = Under(indices.size()); // serve initial?
    },
    // query [l, r) // l < r
    [](Under &dat, int l, int r) -> Data { return dat.query(l, r); },
    // merge y-direction
    [](Data a, Data b) -> Data { return a + b; }
    // optional
    // , identity
);
// }}}
```


# 検証

* [C - Goodbye Souvenir - CF](https://codeforces.com/contest/848/submission/40994915){:target="_blank"}<!--_-->

# 練習

* [C - Goodbye Souvenir - CF](https://codeforces.com/contest/848/problem/C){:target="_blank"}<!--_-->

# 参考

* [ちょっと変わったセグメント木の使い方 \| ei1333の日記](https://ei1333.hateblo.jp/entry/2017/12/14/000000){:target="_blank"}
