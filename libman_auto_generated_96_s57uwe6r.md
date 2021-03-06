---
layout: lib
title: 'sとs[i, -1]の最長共通接頭辞 (z-algorithm)'
permalink: string/z-algorithm

---


Longest Common Prefix; LCP

$s[i, -1]$ とは $s[i, N-1]$ のことです．

SA使うやつ([LCP配列]({{ "string/LCP-array" | absolute_url }}))はSAでのとなりあう２つのLCPなのでちょっと違う．


```cpp
// size of longest common prefix between s and s[i, -1]
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


# 検証

* [F - 最良表現 - AtCoder](https://beta.atcoder.jp/contests/arc060/submissions/2213612){:target="_blank"}<!--_-->
  * 検証のために解説を書き写しただけでよく理解していない解法
  * KMPで解けるからいいんだよ！

# 練習問題

* [ECR - E Vasya and Big Integers - codeforces](https://codeforces.com/contest/1051/problem/E){:target="_blank"}<!--_-->

# 参考

* [文字列の頭良い感じの線形アルゴリズムたち３ - あなたは嘘つきですかと聞かれたら「YES」と答えるブログ](http://snuke.hatenablog.com/entry/2014/12/03/214243){:target="_blank"}<!--_-->

