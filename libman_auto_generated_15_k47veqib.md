---
layout: lib
title: 平衡二部探索木(Balanced Binary Search Tree; BBST)
permalink: data-structure/BBST

---

平衡二分探索木には二種類あるという結論に至った．

その道筋を記したブログを，今書いている．

いつ書き終わるか知らない．

とりあえず結論．

# Sequenceタイプ

lazy-segでできることに加えて，条件付きで区間reverseができるもの．

連続している区間を扱う．

つまり動的セグ木にできることができるとは限らない．

keyはindex

# Multisetタイプ

multisetにできることができる．（インターフェースはちょっと違く実装したけど）

keyは比較が定義されていることが前提．

k-thをとったりすることができる．

あとは通常のmultisetに対する利点といえば，  
merge, splitだろうか．

---

実装時にこれらのうち共通部分を〜とかしようとしたけどどうしたらいいのやらという感じでやめた．