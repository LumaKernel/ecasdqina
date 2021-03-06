---
layout: lib
title: 永続遅延セグメントツリー
permalink: data-structure/SegmentTree/PersistentLazySegmentTree

---


top-down にやる

# 実装

2冪丸めはしていない

[memory_pool_no_deallocate]({{ "misc/memory-pool-allocator#memory_pool_no_deallocate" | absolute_url }}) を使うのが一番速いはず (詳しくは検証に提出したコードなどを参照)


```cpp
// .binary_search(l, r, maximize : bool, exists : (x, l, r) => bool) : [l, r)
// PersistentLazySegmentTree {{"{{"}}{
#include <cassert>
#include <initializer_list>
#include <iterator>
#include <vector>
// node {{"{{"}}{
template < class M_act >
struct PersistentLazySegmentTreeNode {
  using Monoid = typename M_act::Monoid;
  using X = typename Monoid::T;
  using M = typename M_act::M;
  using node_type = PersistentLazySegmentTreeNode;
  using pointer = node_type *;
  using const_pointer = const node_type *;
  using size_type = unsigned long long;

  X data = Monoid::identity();
  M lazy = M_act::identity();
  pointer left = 0, right = 0;

  PersistentLazySegmentTreeNode() {}
  PersistentLazySegmentTreeNode(const X &data) : data(data) {}
  PersistentLazySegmentTreeNode(pointer left, pointer right) : left(left), right(right) {}

  PersistentLazySegmentTreeNode(const node_type &node) { *this = node; }

  void eval(size_type sz) {
    if(lazy != M_act::identity()) {
      data = M_act::actInto(lazy, sz, data);
      if(sz > 1) {
        assert(left && right);
        left->lazy = M_act::op(lazy, left->lazy);
        right->lazy = M_act::op(lazy, right->lazy);
      }
      lazy = M_act::identity();
    }
  }

  void update() { data = Monoid::op(getData(left), getData(right)); }

  friend X getData(const_pointer a) { return a ? a->data : Monoid::identity(); }
};
// }}}

template < class M_act,
           class Allocator = std::allocator< PersistentLazySegmentTreeNode< M_act > > >
struct PersistentLazySegmentTree {
public:
  static Allocator alc;

public:
  using Monoid = typename M_act::Monoid;
  using X = typename Monoid::T;
  using M = typename M_act::M;

  using size_type = unsigned long long;
  using index_type = long long;
  using node_type = PersistentLazySegmentTreeNode< M_act >;
  using pointer = node_type *;
  using const_pointer = const node_type *;

  size_type n;

  pointer root;

private:
  PersistentLazySegmentTree(size_type n, pointer root) : n(n), root(root) {}

public:
  PersistentLazySegmentTree() : n(0), root(0) {}
  PersistentLazySegmentTree(size_type n) : PersistentLazySegmentTree(n, nullptr) {}

  template < class InputIter,
             class = typename std::iterator_traits< InputIter >::value_type >
  PersistentLazySegmentTree(InputIter first, InputIter last)
      : PersistentLazySegmentTree(distance(first, last)) {
    root = build< InputIter >(first, 0, n);
  }
  PersistentLazySegmentTree(std::initializer_list< X > ilist)
      : PersistentLazySegmentTree(ilist.begin(), ilist.end()) {}

private:
  template < class InputIter >
  pointer build(InputIter &ite, size_type l, size_type r) {
    pointer p = alc.allocate(1);
    if(r - l == 1) {
      std::allocator_traits< Allocator >::construct(alc, p, *(ite++));
    } else {
      pointer left = build< InputIter >(ite, l, (l + r) >> 1),
              right = build< InputIter >(ite, (l + r) >> 1, r);
      std::allocator_traits< Allocator >::construct(alc, p, left, right);
      p->update();
    }
    return p;
  }

private:
  static pointer clone(const_pointer node) {
    pointer p = alc.allocate(1);
    if(node) {
      std::allocator_traits< Allocator >::construct(alc, p, *node);
    } else {
      std::allocator_traits< Allocator >::construct(alc, p);
    }
    return p;
  }

public:
  PersistentLazySegmentTree set(size_type i, const X &x) const {
    assert(i < n);
    pointer new_root = clone(root);
    set(i, x, new_root, 0, n);
    return PersistentLazySegmentTree(n, new_root);
  }

private:
  void set(size_type i, const X &x, pointer &k, size_type l, size_type r) const {
    if(r - l > 1) {
      k->left = clone(k->left);
      k->right = clone(k->right);
      k->eval(r - l);
      if(i < ((l + r) >> 1)) {
        set(i, x, k->left, l, (l + r) >> 1);
      } else {
        set(i, x, k->right, (l + r) >> 1, r);
      }
      k->update();
    } else {
      k->data = x;
    }
  }

public:
  PersistentLazySegmentTree act(index_type l, index_type r, const M &m) const {
    if(l < 0) l = 0;
    if(r > (index_type) n) r = n;
    if(l >= r) return *this;
    pointer new_root = clone(root);
    act(l, r, m, new_root, 0, n);
    return PersistentLazySegmentTree(n, new_root);
  }

private:
  void act(size_type a, size_type b, const M &m, pointer &k, size_type l,
           size_type r) const {
    if(r - l > 1) {
      k->left = clone(k->left);
      k->right = clone(k->right);
    }
    k->eval(r - l);
    if(b <= l || r <= a) {
      return;
    }
    if(a <= l && r <= b) {
      k->lazy = m;
      k->eval(r - l);
      return;
    }
    act(a, b, m, k->left, l, (l + r) >> 1);
    act(a, b, m, k->right, (l + r) >> 1, r);
    k->update();
  }

public:
  X get(size_type i) const {
    assert(i < n);
    return fold(i, i + 1);
  }

  X fold(index_type l, index_type r) const {
    if(l < 0) l = 0;
    if(r > (index_type) n) r = n;
    if(l >= r) return Monoid::identity();
    return fold(l, r, M_act::identity(), root, 0, n);
  }

private:
  X fold(size_type a, size_type b, M folded_act, const_pointer k, size_type l,
         size_type r) const {
    if(b <= l || r <= a) return Monoid::identity();
    if(k) folded_act = M_act::op(folded_act, k->lazy);
    if(a <= l && r <= b) {
      return M_act::actInto(folded_act, r - l, k ? k->data : Monoid::identity());
    }
    return Monoid::op(fold(a, b, folded_act, k ? k->left : 0, l, (l + r) >> 1),
                      fold(a, b, folded_act, k ? k->right : 0, (l + r) >> 1, r));
  }

public:
  static const index_type not_found = -1;
  template < class T >
  index_type binary_search(bool maximize, const T &exists) {
    return binary_search(0, n, maximize, exists);
  }
  template < class T >
  index_type binary_search(index_type l, index_type r, bool maximize,
                           const T &exists) const {
    if(l < 0) l = 0;
    if(r > static_cast< index_type >(n)) r = n;
    if(l >= r) return not_found;
    return binary_search(l, r, maximize, exists, M_act::identity(), root, 0, n);
  }

private:
  template < class T >
  index_type binary_search(size_type a, size_type b, bool maximize, const T &exists,
                           M folded_act, const_pointer k, size_type l,
                           size_type r) const {
    if(b <= l || r <= a) return not_found;
    if(k) folded_act = M_act::op(folded_act, k->lazy);
    if(a <= l && r <= b) {
      X x = M_act::actInto(folded_act, r - l, k ? k->data : Monoid::identity());
      if(!exists(x, l, r)) return not_found;
      if(r - l == 1) return l;
    }
    index_type i;
    if(maximize) {
      i = binary_search(
          a, b, maximize, exists, folded_act, k ? k->right : 0, (l + r) >> 1, r);
      if(i == not_found)
        i = binary_search(
            a, b, maximize, exists, folded_act, k ? k->left : 0, l, (l + r) >> 1);
    } else {
      i = binary_search(
          a, b, maximize, exists, folded_act, k ? k->left : 0, l, (l + r) >> 1);
      if(i == not_found)
        i = binary_search(
            a, b, maximize, exists, folded_act, k ? k->right : 0, (l + r) >> 1, r);
    }
    return i;
  }

public:
  size_type size() const { return n; }
};
template < class M_act, class Allocator >
Allocator PersistentLazySegmentTree< M_act, Allocator >::alc;
// }}}

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
  static constexpr X actInto(const M &, long long, const X &x) {
    return x;
  }
};

template < class U = long long >
struct RangeMin {
  using T = U;
  static T op(const T &a, const T &b) { return std::min< T >(a, b); }
  static constexpr T identity() { return T(inf_monoid); }
};

template < class U = long long >
struct RangeMax {
  using T = U;
  static T op(const T &a, const T &b) { return std::max< T >(a, b); }
  static constexpr T identity() { return T(-inf_monoid); }
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
struct RangeAnd< std::bitset< N > > {
  using T = std::bitset< N >;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return std::bitset< N >().set(); }
};

/// }}}--- ///

/// --- M_act examples {{"{{"}}{ ///
template < class U = long long, class V = U >
struct RangeMinAdd {
  using X = U;
  using M = V;
  using Monoid = RangeMin< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, long long, const X &x) { return m + x; }
};

template < class U = long long, class V = U >
struct RangeMaxAdd {
  using X = U;
  using M = V;
  using Monoid = RangeMax< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, long long, const X &x) { return m + x; }
};

template < class U = long long, class V = U >
struct RangeMinSet {
  using M = U;
  using Monoid = RangeMin< U >;
  using X = typename Monoid::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return M(-inf_monoid); }
  static X actInto(const M &m, long long, const X &x) { return m == identity() ? x : m; }
};

template < class U = long long, class V = U >
struct RangeMaxSet {
  using M = U;
  using Monoid = RangeMax< U >;
  using X = typename Monoid::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return M(-inf_monoid); }
  static X actInto(const M &m, long long, const X &x) { return m == identity() ? x : m; }
};

template < class U = long long, class V = U >
struct RangeSumAdd {
  using X = U;
  using M = V;
  using Monoid = RangeSum< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, long long n, const X &x) { return m * n + x; }
};

template < class U = long long, class V = U >
struct RangeSumSet {
  using X = U;
  using M = V;
  using Monoid = RangeSum< U >;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return M(-inf_monoid); }
  static X actInto(const M &m, long long n, const X &x) {
    return m == identity() ? x : m * n;
  }
};

template < class U, class V = U >
struct RangeProdMul {
  using X = U;
  using M = V;
  using Monoid = RangeProd< U >;
  static M mpow(M a, long long b) {
    X r(1);
    while(b) {
      if(b & 1) r = r * a;
      a = a * a;
      b >>= 1;
    }
    return r;
  }
  static M op(const M &a, const M &b) { return a * b; }
  static constexpr M identity() { return M(1); }
  static X actInto(const M &m, long long n, const X &x) { return x * mpow(m, n); }
};

template < class U, class V = U >
struct RangeProdSet {
  using X = U;
  using M = V;
  using Monoid = RangeProd< U >;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return V::unused; }
  static X actInto(const M &m, long long n, const X &) {
    if(m == identity()) return;
    return RangeProdMul< U, V >::mpow(m, n);
  }
};

template < class U = long long, class V = U >
struct RangeOr2 {
  using X = U;
  using M = V;
  using Monoid = RangeOr< U >;
  static M op(const M &a, const M &b) { return a | b; }
  static constexpr M identity() { return M(0); }
  static X actInto(const M &m, long long, const X &x) { return m | x; }
};

template < class U = long long, class V = U >
struct RangeAnd2 {
  using X = U;
  using M = V;
  using Monoid = RangeAnd< U >;
  static M op(const M &a, const M &b) { return a & b; }
  static constexpr M identity() { return M(-1); }
  static X actInto(const M &m, long long, const X &x) { return m & x; }
};

template < class U, size_t N >
struct RangeAnd2< U, std::bitset< N > > {
  using X = U;
  using M = std::bitset< N >;
  using Monoid = RangeAnd< U >;
  static M op(const M &a, const M &b) { return a & b; }
  static constexpr M identity() { return std::bitset< N >().set(); }
  static X actInto(const M &m, long long, const X &x) { return m & x; }
};
/// }}}--- ///

using m_act = RangeSumAdd<>;
// using Seg = PersistentLazySegmentTree< m_act >;
using node_type = PersistentLazySegmentTree< m_act >::node_type;

#ifdef DEBUG
constexpr int POOL_SIZE = 500;
#else
constexpr int POOL_SIZE = 4 * Q * lgN;
#endif

// using Seg = PersistentLazySegmentTree<
//     m_act, memory_pool_no_deallocate< POOL_SIZE >::core< node_type > >;
```


# 検証

* [Persistent oak - CodeChef](https://www.codechef.com/viewsolution/23362984){:target="_blank"}<!--_-->

# 練習問題

* [Persistent oak - CodeChef](https://www.codechef.com/problems/OAK){:target="_blank"}<!--_-->

# 参考

なし

