---
layout: post
title: "draft-bernardos-mext-dmm-pmip-01"
date: 2013-04-22 21:04
comments: true
categories: DMM
---

# A PMIPv6-based solution for Distributed Mobility Management #
### [draft-bernardos-mext-dmm-pmip-01](http://tools.ietf.org/html/draft-bernardos-mext-dmm-pmip-01) ###
>July 11, 2011

    本文描述了一种在代理移动IPv6域中分散数据转发平面的解决方案，即试图避免（数据包）经过LMA时所引入的次优数据路径问题。

----------

## 1. Introduction ##
集中式的移动性解决方案容易出现一些**问题和制约**：更长（次优）的路由路径、可扩展性问题、信令开销（最有可能是一个更长的相关性切换时延）、更复杂的网络部署、由潜在的单点故障引起的更高的脆弱性、移动性管理服务中粒度的缺乏。

本文基于网络的移动性支持来实现DMM。

## 2.  Terminology  ##

>MAAR (Mobility Anchor and Access Router).移动节点连接的第一跳路由器。它们也负责对它们所匹配的IPv6前缀的移动性管理。

>CMD (Central Mobility Database).在移动域中为移动节点存储绑定缓存入口(BCE)的节点

>A-MAAR(Anchor MAAR) 。移动节点刚访问过的MAAR，它通告了MN在active flow中使用的IPv6前缀。

>S-MAAR (Serving MAAR)。 移动节点现在连接的MAAR。

## 3.  Description of the solution ##
    思想：使移动锚点更靠近移动节点，从而克服传统集中式移动性管理的限制
	本文中，中央锚点被移动到网络的边缘，部署在移动节点的默认网关中。
	也就是说，为一系列的移动节点提供IP连通性的锚点，同时也是这些移动节点的移动性管理者。

#### 3.1.  Initial registration ####
MAAR像普通路由器一样工作，这样就没有多余的封装和处理产生。

     +-----+      +---+                +--+
     |MAAR1|      |CMD|                |CN|
     +-----+      +---+                +*-+
        |           |                   *
       MN           |                   *     +---+
	attach detection|               *****    _|CMD|_
        |           |         flow1 *       / +-+-+ \
        |           |               *      /    |    \
        |--- PBU -->|               *     /     |     \
        |           |               *    /      |      \
        |          BCE          +---*-+-'    +--+--+    `+-----+
        |       creation        |   * |      |     |     |     |
        |           |           |MAAR1+------+MAAR2+-----+MAAR3|
        |<-- PBA ---|           |   * |      |     |     |     |
        |           |           +---*-+      +-----+     +-----+
        |           |               *
        |           |         Pref1 *
        |           |              +*-+
        |           |              |MN|
        |           |              +--+

    (Operations sequence)                (Packets flow)

                 Figure 1: First attachment to the network

#### 3.2.  The CMD as PBU/PBA relay ####
两个MAAR之间就建立隧道

    +-----+      +---+      +-----+           +--+            +--+
    |MAAR1|      |CMD|      |MAAR2|           |CN|            |CN|
    +-----+      +---+      +-----+           +*-+            +*-+
      |           |           |               *               *
      |           |          MN               *     +---+     *
      |           |        attach.        *****    _|CMD|_    *
      |           |          det.   flow1 *       / +-+-+ \   *flow2
      |           |<-- PBU ---|           *      /    |    \  *
      |          BCE          |           *     /     | *******
      |        check+         |           *    /      | *    \
      |        update         |       +---*-+-'    +--+-*+    `+-----+
      |<-- PBU ---|           |       |   * |      |    *|     |     |
    route         |           |       |MAAR1|______|MAAR2+-----+MAAR3|
    update        |           |       |   **(______)**  *|     |     |
      |--- PBA -->|           |       +-----+      +-*--*+     +-----+
      |         BCE           |                      *  *
      |        update         |                Pref1 *  *Pref2
      |           |--- PBA -->|                     +*--*+
      |           |         route         ---move-->|*MN*|
      |           |         update                  +----+

         (Operations sequence)                (Packets flow)

             Figure 2: Scenario after a handover, CMD as relay

当CMD从新的S-MAAR收到第一个PBU后，它向所有的A-MAAR转发PBU的拷贝，这个PBU在BCE中表示为现在的P-CoA(例如切换之前的MAAR)和以前的P-CoA。它们用PBA回复CMD，CMD将这些PBA聚集成一个PBA来通知S-MAAR，这样就最终建立了与A-MAARs之间的隧道。


#### 3.3.  The CMD as MAAR locator ####
允许A-MAAR直接向新的S-MAAR发送信息

    +-----+      +---+      +-----+           +--+            +--+
    |MAAR1|      |CMD|      |MAAR2|           |CN|            |CN|
    +-----+      +---+      +-----+           +*-+            +*-+
      |           |           |               *               *
      |           |          MN               *     +---+     *
      |           |        attach.        *****    _|CMD|_    *
      |           |          det.   flow1 *       / +-+-+ \   *flow2
      |           |<-- PBU ---|           *      /    |    \  *
      |          BCE          |           *     /     | *******
      |        check+         |           *    /      | *    \
      |        update         |       +---*-+-'    +--+-*+    `+-----+
      |<-- PBU ---|           |       |   * |      |    *|     |     |
    route         |           |       |MAAR1|______|MAAR2+-----+MAAR3|
    update        |           |       |   **(______)**  *|     |     |
      |--------- PBA -------->|       +-----+      +-*--*+     +-----+
      |--- PBA -->|         route                    *  *
      |          BCE        update             Pref1 *  *Pref2
      |         update        |                     +*--*+
      |           |           |           ---move-->|*MN*|
      |           |           |                     +----+

         (Operations sequence)                (Packets flow)

            Figure 3: Scenario after a handover, CMD as locator


####3.4.  The CMD as PBU/PBA proxy ####
CMD在通知A-MAAR位置变化之前，向新的S-MAAR发送PBA

    +-----+      +---+      +-----+           +--+            +--+
    |MAAR1|      |CMD|      |MAAR2|           |CN|            |CN|
    +-----+      +---+      +-----+           +*-+            +*-+
       |           |           |               *               *
       |           |          MN               *     +---+     *
       |           |        attach.        *****    _|CMD|_    *
       |           |          det.   flow1 *       / +-+-+ \   *flow2
       |           |<-- PBU ---|           *      /    |    \  *
       |          BCE          |           *     /     | *******
       |        check+         |           *    /      | *    \
       |        update         |       +---*-+-'    +--+-*+    `+-----+
       |<-- PBU ---x--- PBA -->|       |   * |      |    *|     |     |
    route          |         route     |MAAR1|______|MAAR2+-----+MAAR3|
    update         |         update    |   **(______)**  *|     |     |
       |--- PBA  ->|           |       +-----+      +-*--*+     +-----+
       |          BCE          |                      *  *
       |         update        |                Pref1 *  *Pref2
       |           |           |                     +*--*+
       |           |           |           ---move-->|*MN*|
       |           |           |                     +----+

         (Operations sequence)                (Packets flow)

             Figure 4: Scenario after a handover, CMD as proxy