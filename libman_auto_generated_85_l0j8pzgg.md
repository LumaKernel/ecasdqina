---
layout: lib
title: 3次元凸包
permalink: geometory/ConvexHull3D

---


# 使い方

[幾何基本ライブラリ]({{ "geometory/geometory" | absolute_url }}) と [3次元幾何ライブラリ]({{ "geometory/geometory3D" | absolute_url }}) が必要

与えられた点に対して，凸包をなす頂点集合を求める．ついでに副産物を色々求める

与えられる点は，4つ以上の点で，同一平面上ではないことが条件

すべての面が三角形になるように保持・計算する

同一平面上の点については表面であれば通常通り三角形を形成し，内部であれば内部であると判定される

同じ点については自分で，`unique` すること

`insert(a, b, c)` : 点を追加  
`build()` : 凸包を構築  

`vertexSize(), edgeSize(), faceSize()`  
各種サイズ取得

`vector< int > getVertexIds()`  
凸包の頂点のid集合を取得．もとの頂点番号と対応していることを保証しません  

`valarray< Scalar > getVertex(int id) const`  
idに対応する頂点取得

`vector< array< int, 3 > > getPolygons() const`  
凸包を構成する，頂点のid3つからなるポリゴン全取得

`array< int, 3 > getPolygon(int id) const`  
ポリゴンIDからポリゴンを取得

`array< valarray< Scalar >, 3 > getPolygonPoints(int id) const`  
ポリゴンIDから頂点をそのまま取得

`valarray< Scalar > getPolygonOuterNormalVector(int id) const`  
ポリゴンIDから，そのポリゴン面と垂直で，サイズがその面の面積の二倍の外側向きのベクトルを取得

入力が整数であれば，すべて整数の範囲で計算可能

# 実装


```cpp
// require Geometory and Geometory3D
/// --- Convex Hull 3D {{"{{"}}{ ///

// Convex Polyhedron
#include <array>
// N : input size
// V : vertex size
// E : edge size
// F : face size
// K : the number of the points that can see thp polygon that includes horizon
struct ConvexHull3D {
  vector< valarray< Scalar > > points;
  vector< int > used; // point is used as convex?
  vector< array< int, 3 > > polygons;
  vector< valarray< Scalar > > polygonOuterNormalVectors;
  void insert(Scalar a, Scalar b, Scalar c) { points.push_back({a, b, c}); }

private:
  vector< vector< int > > canBeSeenFrom;
  vector< vector< int > > canSee;
  vector< array< int, 3 > > prePolygons;
  vector< valarray< Scalar > > prePolygonOuterNormalVectors;
  vector< int > insides;
  map< pair< int, int >, vector< int > > edgeToFace;
  int vsz;

public:
  void build() {
    assert(points.size() >= 4);
    int vnum = points.size();
    canSee.resize(vnum);
    // choose 3 points that are not on the same line
    {
      bool found = false;
      for(int i = 2; i < vnum; i++)
        if(sign(LINFnorm(cross(points[1] - points[0], points[i] - points[0]))) != 0) {
          swap(points[2], points[i]);
          found = true;
          break;
        }
      assert(found && "on the same line");
    }

    // choose 4 points that are not on the same plane
    {
      auto normalVector = cross(points[1] - points[0], points[2] - points[0]);
      bool found = false;
      for(int i = 3; i < vnum; i++)
        if(sign(dot(normalVector, points[i] - points[0])) != 0) {
          swap(points[3], points[i]);
          found = true;
          break;
        }
      assert(found && "on the same plane");
    }
    // every point to every polygon
    for(int j = 0; j < 4; j++) {
      addPolygon(j, (j + 1) % 4, (j + 2) % 4, (j + 3) % 4, -1);
      for(int i = 4; i < vnum; i++) {
        check(i, j);
      }
    }
    vector< int > removed(prePolygons.size());
    vector< int > willRemoved(prePolygons.size());
    vector< int > changeFlag(vnum, -1);
    for(int i = 4; i < vnum; i++) {
      vector< int > vs;
      vector< tuple< int, int, int, int > > edges;
      int ok = 0;
      for(int polygon : canSee[i]) {
        if(removed[polygon]) continue;
        willRemoved[polygon] = i;
      }
      for(int polygon : canSee[i]) {
        if(removed[polygon]) continue;
        removed[polygon] = 1;
        ok = 1;
        for(int j = 0; j < 3; j++) {
          int x = prePolygons[polygon][j], y = prePolygons[polygon][(j + 1) % 3];
          if(x > y) swap(x, y);
          vector< int > newVec;
          for(int poly : edgeToFace[make_pair(x, y)])
            if(!removed[poly] && willRemoved[poly] != i) {
              newVec.emplace_back(poly);
              int z = 0;
              while(x == prePolygons[poly][z] || y == prePolygons[poly][z]) z++;
              edges.emplace_back(x, y, prePolygons[poly][z], poly);
              for(int v : canBeSeenFrom[poly])
                if(changeFlag[v] != i && v > i) changeFlag[v] = i, vs.emplace_back(v);
              break;
            }
          edgeToFace[make_pair(x, y)] = newVec;
        }
        for(int v : canBeSeenFrom[polygon])
          if(changeFlag[v] != i && v > i) changeFlag[v] = i, vs.emplace_back(v);
        canBeSeenFrom[polygon].clear();
      }
      canSee[i].clear();
      if(!ok) continue; // inside convex
      // maybe O(KF) ?
      for(tuple< int, int, int, int > edge : edges) {
        int x, y, z, poly;
        tie(x, y, z, poly) = edge;
        addPolygon(x, y, i, z, insides[poly]);
        for(int v : vs) {
          check(v, prePolygons.size() - 1);
        }
        removed.emplace_back(0);
        willRemoved.emplace_back(-1);
      }
    }
    // finished
    for(size_t i = 0; i < removed.size(); i++)
      if(!removed[i]) {
        polygons.emplace_back(prePolygons[i]);
        polygonOuterNormalVectors.emplace_back(prePolygonOuterNormalVectors[i]);
      }
    //
    used.resize(vnum);
    for(auto polygon : polygons) {
      for(int v : polygon) used[v] = 1;
    }
    vsz = 0;
    for(int i = 0; i < vnum; i++)
      if(used[i]) vsz++;
  }
  int vertexSize() const { return vsz; }
  int edgeSize() const { return prePolygons.size() / 2 * 3; }
  int faceSize() const { return polygons.size(); }
  // O(N)
  vector< int > getVertexIds() const {
    vector< int > ids;
    ids.reserve(vsz);
    for(size_t i = 0; i < points.size(); i++)
      if(used[i]) ids.emplace_back(i);
    return ids;
  }
  valarray< Scalar > getVertex(int id) const { return points[id]; }
  vector< array< int, 3 > > getPolygons() const { return polygons; }
  array< int, 3 > getPolygon(int id) const { return polygons[id]; }
  array< valarray< Scalar >, 3 > getPolygonPoints(int id) const {
    return {points[polygons[id][0]], points[polygons[id][1]], points[polygons[id][2]]};
  }
  valarray< Scalar > getPolygonOuterNormalVector(int id) const {
    return polygonOuterNormalVectors[id];
  }

private:
  void addPolygon(int i, int j, int k, int inside, int inside2) {
    // adjust normal vector using the point known as inside
    prePolygonOuterNormalVectors.push_back(
        cross(points[j] - points[i], points[k] - points[i]));
    Scalar projection =
        dot(prePolygonOuterNormalVectors.back(), (points[inside] - points[i]));
    Scalar projection2 = inside2 != -1 ? dot(prePolygonOuterNormalVectors.back(),
                                             (points[inside2] - points[i]))
                                       : 0;
    if(sign(projection) == 0) projection = projection2, inside = inside2;
    assert(sign(projection) != 0);
    insides.emplace_back(inside);
    if(sign(projection) > 0) {
      swap(j, k);
      prePolygonOuterNormalVectors.back() *= -1;
    }
    prePolygons.push_back({i, j, k});
    edgeToFace[make_pair(min(i, j), max(i, j))].emplace_back(prePolygons.size() - 1);
    edgeToFace[make_pair(min(j, k), max(j, k))].emplace_back(prePolygons.size() - 1);
    edgeToFace[make_pair(min(k, i), max(k, i))].emplace_back(prePolygons.size() - 1);
    canBeSeenFrom.push_back({});
  }
  void check(int vertex, int polygon) {
    // if vertex is on the polygon's plane, judge as visible
    bool see = sign(dot(points[vertex] - points[prePolygons[polygon][0]],
                        prePolygonOuterNormalVectors[polygon])) >= 0;
    if(see) {
      canSee[vertex].emplace_back(polygon);
      canBeSeenFrom[polygon].emplace_back(vertex);
    }
  }
};

/// }}}--- ///
```


参考にしたスライドの通り実装しようとしたが，`set`や`map`でlogがついてたり，なんか細かいやり方については書かれていなかったのでなるべく計算量が小さくなるように書いたつもり

各座標成分 $[-10^5,10^5]$ でランダムに生成した入力をよませてみると，$10^5$ 個のとき4秒ぐらい，$10^4$個のとき0.3秒ぐらいと， $\Omega(N^2)$ よりは明らかに速い結果が得られたので良しとした (両ケースとも凸包を成す頂点数は150程度だった)．改良については詳細が分かったらまたするかもしれない

最悪ケースに対しては，これもちょっとよく分かっていないが，$O(N^3)$ ぐらいだと信じている

# 応用

* もとの頂点の番号が知りたい
  * swapは2回しか無いのでそれを保持しとく

# 検証

* [L - 凸包が映し出される平面 (KUPC2018) - AtCoder](https://beta.atcoder.jp/contests/kupc2018/submissions/3420739){:target="_blank"}<!--_-->
  * コメントを日本語で書いているのでこっちの方がわかりやすいかもしれない

# 参考

* [3次元の凸包を求める - SlideShare](https://www.slideshare.net/abc3141/3-65377198){:target="_blank"}<!--_-->
