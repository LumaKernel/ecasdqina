---
layout: lib
permalink: string/z-algorithm
title: Zアルゴリズム

---


```cpp
// size of longest common prefix between s and s[i...]
vector< int > Zalgorithm(string s) {
  int n = s.size();
  vector< int > Z(n);
  Z[0] = n;
  int i = 1, j = 0;
  while(i < n) {
    while(i + j < n && s[j] == s[i + j]) ++j;
    Z[i] = j;
    if(j == 0) {
      ++i;
      continue;
    }
    int k = 1;
    while(i + k < n && Z[k] < j - k) Z[i + k] = Z[k], ++k;
    i += k, j -= k;
  }
  return Z;
}
```
