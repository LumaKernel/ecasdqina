---
layout: lib
permalink: data-structure/misc/BIT2D
title: BIT2D

---



```cpp
// NOTE : query in range and x1 <= x2, y is same
/// --- 2D BIT Library {{"{{"}}{ ///

template < class T = long long, class V = function< T(T, T) > >
struct BIT2D {
  int h, w;
  T identity;
  vector< T > dat;
  V merge;
  BIT2D(int h, int w, T identity = T(),
        V merge = [](T a, T b) { return a + b; })
      : h(h), w(w), identity(identity), dat(h * w, identity), merge(merge) {}
  void add(int y, int x, T const &val) {
    y++;
    for(; y <= h; y += y & -y) addX(y, x, val);
  }
  void set(int y, int x, T val) { add(y, x, merge(val, -get(y, x))); }
  T sum(int y, int x) {
    if(y < 0 || x < 0) return 0;
    y++;
    T r = identity;
    for(; y > 0; y -= y & -y) r = merge(r, sumX(y, x));
    return r;
  }
  T get(int y, int x) { return range(y, x, y, x); }
  T range(int y1, int x1, int y2, int x2) {
    return sum(y2, x2) - sum(y1 - 1, x2) - sum(y2, x1 - 1) +
           sum(y1 - 1, x1 - 1);
  }

private:
  inline int id(int y, int x) { return (y - 1) * w + (x - 1); }
  void addX(int y, int x, T const &val) {
    x++;
    for(; x <= w; x += x & -x) dat[id(y, x)] = merge(dat[id(y, x)], val);
  }
  T sumX(int y, int x) {
    x++;
    T r = identity;
    for(; x > 0; x -= x & -x) r = merge(r, dat[id(y, x)]);
    return r;
  }
};

/// }}}--- ///
```
