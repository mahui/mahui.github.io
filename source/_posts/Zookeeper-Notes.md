---
title: Zookeeper笔记
date: 2017-07-17 22:44:57
tags:
---

# CreateMode 解释
```
// 当前连接断开后该 znode 不会自动删除
PERSISTENT (0, false, false),

// 当前连接断开后该 znode 不会自动删除，它的名字会附加一个单调递增数字。
PERSISTENT_SEQUENTIAL (2, false, true),

// 当前连接断开后该 znode 会自动删除
EPHEMERAL (1, true, false),

// 当前连接断开后该 znode 会自动删除，它的名字会附加一个单调递增数字。
EPHEMERAL_SEQUENTIAL (3, true, true);
```
以上自增数字疑似全局共享
