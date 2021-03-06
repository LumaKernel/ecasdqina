---
layout: lib
title: 最大フロー (Ford-Fulkerson)
permalink: graph/flow/FordFulkerson

---


動的なもの（あとで辺の重みや辺自体が減ったり増えたり）に対応できる．（という話をどっかで見た）  
[Dinic](graph/flow/Dinic)ではできなかったので注意．

動的なものを考慮した実装．


```cpp
// constructor(n, inf?) // be careful !
// addEdge(from, to, capacity, isDirected? = false) returns edgeID
// === build(s, t) - returns max flow (or inf) ===
// === restoreMinCut(s) ===
// .isCut[edgeID]
// === --- ===
// inf * 2 < LL_MAX
/// --- Max Flow with Ford-Fulkerson Library {{"{{"}}{ ///

// Ford-Fulkerson
// O(FV)
struct MaxFlow {
  struct Edge {
    int from, to;
    ll cap, rev;
    int To(int i) const { return from == i ? to : from; }
    ll& Cap(int i) { return from == i ? cap : rev; }
    ll& Rev(int i) { return from == i ? rev : cap; }
  };
  int n;
  vector< Edge > edges;
  vector< vector< int > > g;
  ll inf;
  MaxFlow(int n, ll inf = 1e18) : n(n), g(n), inf(inf) {}
  int addEdge(int a, int b, ll cap, bool undirected = false) {
    edges.emplace_back((Edge){a, b, cap, undirected ? cap : 0});
    g[a].emplace_back(edges.size() - 1);
    g[b].emplace_back(edges.size() - 1);
    return edges.size() - 1;
  }
  ll build(int s, int t) {
    ll flow = 0;
    while(1) {
      vector< int > used(n, 0);
      ll x = dfs(used, s, t, inf);
      if(x == 0) break;
      flow += x;
      if(flow >= inf) return inf;
    }
    return flow;
  }
  vector< int > isCut;
  void restoreMinCut(int s) {
    isCut = vector< int >(edges.size());
    // bfs
    vector< int > used(n);
    queue< int > q;
    q.emplace(s);
    used[s] = 1;
    while(q.size()) {
      int i = q.front();
      q.pop();
      for(int idx : g[i]) {
        Edge& edge = edges[idx];
        if(!used[edge.To(i)] && edge.Cap(i) > 0) {
          q.emplace(edge.To(i));
          used[edge.To(i)] = 1;
        }
      }
    }
    for(size_t i = 0; i < edges.size(); i++) {
      if(used[edges[i].from] != used[edges[i].to]) isCut[i] = 1;
    }
  }

private:
  ll dfs(vector< int >& used, int i, int t, ll x) {
    if(i == t) return x;
    used[i] = 1;
    for(int idx : g[i]) {
      Edge& edge = edges[idx];
      if(!used[edge.To(i)] && edge.Cap(i) > 0) {
        ll d = dfs(used, edge.To(i), t, min(x, edge.Cap(i)));
        if(d == 0) continue; ////
        edge.Cap(i) -= d;
        edge.Rev(i) += d;
        return d;
      }
    }
    return 0;
  }
};

/// }}}--- ///

const int N = 2e6;
const ll inf = 1e18;
MaxFlow flow(N, inf);
```


# 検証

* [F - Lotus Leaves (800) - AtCoder](https://beta.atcoder.jp/contests/arc074/submissions/2450524){:target="_blank"}<!--_-->

# 練習問題

* [F - Lotus Leaves (800) - AtCoder](https://beta.atcoder.jp/contests/arc074/tasks/arc074_d){:target="_blank"}<!--_-->
* [K - 光と闇の調和 (KUPC2018) - AtCoder](https://beta.atcoder.jp/contests/kupc2018/tasks/kupc2018_k){:target="_blank"}<!--_-->

