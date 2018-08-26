---
layout: lib
permalink: graph/GraphUtil
title: ユーティリティ

---


時短ライブラリ

# 木の直径


```cpp
/// Tree Diametor {{"{{"}}{ ///

int treeDiameter(UnWeightedGraph tree) {
  int far = 0, dep = -1;
  function< void(int, int, int) > dfs = [&](int i, int p, int d) {
    if(dep < d) far = i, dep = d;
    for(int j : tree[i])
      if(j != p) dfs(j, i, d + 1);
  };
  dfs(0, -1, 0);
  dep = -1;
  dfs(far, -1, 0);
  return dep;
}

/// ---}}} ///
```


# ダイクストラ


```cpp
// dijkstra {{"{{"}}{
using P = tuple< ll, int >;
vector< ll > dist(n, 1e18);
priority_queue< P, vector< P >, greater< P > > pq;
pq.emplace(0, s);
dist[s] = 0;
while(pq.size()) {
  ll di;
  int i;
  tie(di, i) = pq.top();
  pq.pop();
  if(dist[i] < di) continue;
  for(auto to : g) {
    int j = to.first;
    ll ndi = di + to.second;
    if(dist[j] > ndi) {
      dist[j] = ndi;
      pq.emplace(ndi, j);
    }
  }
}
// }}}
```


```cpp
// dijkstra {{"{{"}}{
using P = tuple< ll, int, int >;
vector< vector< ll > > dist(h, vector< ll >(w, 1e18));
deque< P > pq;
pq.emplace_back(0, sy, sx);
dist[sy][sx] = 0;
while(pq.size()) {
  ll di;
  int y, x;
  tie(di, y, x) = pq.front();
  pq.pop_front();
  if(dist[y][x] < di) continue;
  for(int d = 0; d < 4; d++) {
    int ny = y + dy[d];
    int nx = x + dx[d];
    if(ny < 0 || nx < 0 || ny >= h || nx >= w) continue;
    if(dist[ny][nx] > di + 1) {
      dist[ny][nx] = di + 1;
      pq.emplace_back(di + 1, j);
    }
  }
}
// }}}
```
