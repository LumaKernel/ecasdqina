---
layout: lib
title: RadixHeap
permalink: data-structure/Heap/RadixHeap

---

コレクション


```cpp
/// --- RadixHeap Library {{"{{"}}{ ///

template < class T = ll, class U = uint64_t >
struct RadixHeap {
private:
  vector< pair< U, T > > v[sizeof(U) * 8 + 1];
  size_t sz = 0;
  U last = 0;
  size_t logbinll(U x) {
    size_t h = 0;
    x = x & 0xFFFFFFFF00000000 ? (h += 32, x) & 0xFFFFFFFF00000000 : x;
    x = x & 0xFFFF0000FFFF0000 ? (h += 16, x) & 0xFFFF0000FFFF0000 : x;
    x = x & 0xFF00FF00FF00FF00 ? (h += 8, x) & 0xFF00FF00FF00FF00 : x;
    x = x & 0xF0F0F0F0F0F0F0F0 ? (h += 4, x) & 0xF0F0F0F0F0F0F0F0 : x;
    x = x & 0xCCCCCCCCCCCCCCCC ? (h += 2, x) & 0xCCCCCCCCCCCCCCCC : x;
    x = x & 0xAAAAAAAAAAAAAAAA ? (h += 1, x) & 0xAAAAAAAAAAAAAAAA : x;
    return h;
  }

public:
  void push(U x, T y = T()) {
    assert(last <= x);
    v[logbinll(x ^ last)].emplace_back(x, y);
    sz++;
  }
  pair< U, T > pop() {
    assert(sz);
    if(!v[0].size()) {
      size_t i = 1;
      while(!v[i].size()) i++;
      last = min_element(begin(v[i]), end(v[i]))->first;
      for(const pair< U, T >& e : v[i]) v[logbinll(e.first ^ last)].emplace_back(e);
      v[i].clear();
    }
    sz--;
    pair< U, T > res = v[0].back();
    v[0].pop_back();
    return res;
  }
  size_t size() { return sz; }
};

/// }}}--- ///
```


# 参考

* [色々なダイクストラ高速化](https://www.slideshare.net/yosupo/ss-46612984){:target="_blank"}
