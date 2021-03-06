---
layout: lib
title: 直線追加任意順序のCHT
permalink: dynamic-programming/convex-hull-trick/CHT-Ex

---


[CHT]({{ "dynamic-programming/convex-hull-trick/CHT" | absolute_url }}) を直線を任意順序で追加できるようにしたもの

[Li-Chao Tree]({{ "dynamic-programming/convex-hull-trick/LiChaoTree" | absolute_url }}) で代用できる

直線の数を$N$，クエリの数を$Q$ とする

# 実装

オンラインクエリ可能

[ここ](http://d.hatena.ne.jp/sune2/20140310/1394440369){:target="_blank"}<!--_--> で紹介されているものに手を加えたもの


```cpp
// CHTEx<T, is-minimize = true, D = double>
// N is no. of added lines
// === --- ===
// .add(a, b [, id]) : f(x) = ax + b : amortized O(log N)
// .query(x) : O(log N)
// .get(x) : line : O(log N)
// === --- ===
// line.a, line.b, line.id
// line.calc(x)
// === --- ===
// f can duplicate
/// --- Convex Hull Trick Extended {{"{{"}}{ ///
#include <functional>
#include <iostream>
#include <limits>
#include <set>
#include <utility>

template < class T = long long, bool isMinimize = true, class D = double >
struct CHTEx {
  static T INF;
  static T EPS;

  static inline bool comp(const T &a, const T &b) {
    if(isMinimize)
      return a < b;
    else
      return a > b;
  }

private:
  struct Line { // ax + b
    T a, b;
    int id;
    Line(const T &a, const T &b, int id = 0) : a(a), b(b), id(id) {}
    bool operator<(const Line &rhs) const { // (a, b)
      return a != rhs.a ? comp(rhs.a, a) : comp(b, rhs.b);
    }
    T calc(const T &x) { return a * x + b; }
    friend ostream &operator<<(ostream &os, const Line &line) {
      os << "(" << line.a << ", " << line.b << ")";
      return os;
    }
  };
  struct CP {
    T numer, denom; // x-coordinate; denom is non-negative for comparison
    Line p;
    CP(const T &n) : numer(n), denom(1), p(T(0), T(0)) {}
    // p1 < p2
    CP(const Line &p1, const Line &p2) : p(p2) {
      if(p1.a == INF || p1.a == -INF)
        numer = -INF, denom = T(1);
      else if(p2.a == INF || p2.a == -INF)
        numer = INF, denom = T(1);
      else {
        numer = p1.b - p2.b, denom = p2.a - p1.a;
        if(denom < 0) numer = -numer, denom = -denom;
      }
    }
    bool operator<(const CP &rhs) const {
      if(numer == INF || rhs.numer == -INF) return false;
      if(numer == -INF || rhs.numer == INF) return true;
      return (D) numer * rhs.denom < (D) rhs.numer * denom;
    }
  };
  set< Line > lines;
  set< CP > cps;
  typedef typename set< Line >::iterator It;

public:
  CHTEx() { clear(); }
  void add(const T &a, const T &b, int id = 0) {
    const Line p(a, b, id);
    It pos = lines.insert(p).first;
    if(check(*prev(pos), p, *next(pos))) {
      // ax + b is unnecessary
      lines.erase(pos);
      return;
    }
    cps.erase(CP(*prev(pos), *next(pos)));
    {
      It it = prev(pos);
      while(it != lines.begin() && check(*prev(it), *it, p)) --it;
      // lines (it, pos) is unnecessary
      // [it, pos - 1] : [pos - 1, pos] is still not added
      eraseRange(it, prev(pos));
      // [it + 1, pos - 1]
      lines.erase(++it, pos);
      pos = lines.find(p);
    }
    {
      It it = next(pos);
      while(next(it) != lines.end() && check(p, *it, *next(it))) ++it;
      // lines (pos, it) is unnecessary
      // [pos + 1, it] : [pos, pos + 1] is still not added
      eraseRange(++pos, it);
      // [pos + 1, it - 1]
      lines.erase(pos, it);
      pos = lines.find(p);
    }
    cps.insert(CP(*prev(pos), *pos));
    cps.insert(CP(*pos, *next(pos)));
  }
  T query(const T &x) const { return get(x).calc(x); }
  Line get(const T &x) const {
    assert(cps.size());
    return (--cps.lower_bound(CP(x)))->p;
  }
  void clear() {
    cps.clear();
    lines.clear();

    // sentinel
    lines.insert({Line(INF, 0), Line(-INF, 0)});
    cps.insert(CP(Line(INF, 0), Line(-INF, 0)));
  }
  friend ostream &operator<<(ostream &os, const CHTEx &a) {
    os << "CHT-Ex[" << (isMinimize ? "min" : "max") << "]\n";
    assert(a.lines.size() >= 2);
    os << a.lines.size() - 2 << " lines (excluding sentinel)\n";
    for(auto it = next(a.lines.begin()); next(it) != a.lines.end(); ++it)
      os << *it << "\n";
    os << a.cps.size() << " cross points"
       << "\n";
    for(auto &p : a.cps)
      os << "(x = " << p.numer << "/" << p.denom << "; " << p.p.a << ", " << p.p.b << ")"
         << "\n";
    return os;
  }

private:
  // erase cp which can be made from lines [a, b]
  void eraseRange(It a, It b) {
    for(It it = a; it != b; ++it) cps.erase(CP(*it, *next(it)));
  }
  // p2 is unnecessary?
  bool check(const Line &p1, const Line &p2, const Line &p3) {
    if(p1.a == p2.a) return 1;
    if(p1.a == INF || p1.a == -INF || p3.a == INF || p3.a == -INF) return 0;
    //  cp(p2, p3).x <= cp(p2, p1).x
    return (D)(p2.a - p1.a) * (p3.b - p2.b) + EPS >= (D)(p2.b - p1.b) * (p3.a - p2.a);
  }
};

template < class T, bool isMinimize, class D >
T CHTEx< T, isMinimize, D >::INF = numeric_limits< T >::has_infinity
                                       ? numeric_limits< T >::infinity()
                                       : numeric_limits< T >::max();

template < class T, bool isMinimize, class D >
T CHTEx< T, isMinimize, D >::EPS = 1e-19;

/// }}}--- ///
```


インターフェースは [CHT]({{ "dynamic-programming/convex-hull-trick/CHT" | absolute_url }}) と同じ

* `add` : ならし $O(\log N)$
* `get` : $O(\log N)$

# EPSについて

`(aの正の最小値)(bの正の最小値)` より小さな正の値を採用するといい

たとえばデフォルトでは `1e-19` になっているが，これは両方が小数9桁ずつを想定している

---

その他のTIPSに関しては [CHT]({{ "dynamic-programming/convex-hull-trick/CHT" | absolute_url }}) の方に載せています

# 検証

* [D - Computer Game - codeforces](https://codeforces.com/contest/1067/submission/45442782){:target="_blank"}<!--_-->
* [C - Kalila and Dimna in the Logging Industry - codeforces](https://codeforces.com/contest/319/submission/48890326){:target="_blank"}<!--_-->
  * overflow対策で `D = double` としています

# 練習問題

* [NIKKEI 2019本選 G - Greatest Journey (1200) - AtCoder](https://atcoder.jp/contests/nikkei2019-final/tasks/nikkei2019_final_g){:target="_blank"}<!--_-->

# 参考

* [Convex Hull Trick - 競技プログラミング+αなブログ](http://d.hatena.ne.jp/sune2/20140310/1394440369){:target="_blank"}<!--_-->

