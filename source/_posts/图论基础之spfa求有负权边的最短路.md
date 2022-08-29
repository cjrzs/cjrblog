---
title: 图论基础之spfa求有负权边的最短路
math: true
date: 2021-09-07
comment: true
tags:
- 经典必会
- spfa
- AcWing算法基础课
- 最短路问题
- LeetCode
categories:
- 算法
- 图论
---

# [AcWing 851. spfa求最短路 ](https://www.acwing.com/problem/content/853/)

> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环， 边权可能为负数。
> 
> 请你求出 1 号点到 n 号点的最短距离，如果无法从 1 号点走到 n 号点，则输出 impossible。
> 
> 数据保证不存在负权回路。
> 
> **输入格式**
> 第一行包含整数 n 和 m。
> 
> 接下来 m 行每行包含三个整数 x,y,z，表示存在一条从点 x 到点 y 的有向边，边长为 z。
> 
> **输出格式**
> 输出一个整数，表示 1 号点到 n 号点的最短距离。
> 
> 如果路径不存在，则输出 impossible。
> 
> **数据范围**
> 
>     1≤n,m≤105,
>     图中涉及边长绝对值均不超过 10000。
> 
> **输入样例**
> 
>     3 3
>     1 2 5
>     2 3 -3
>     1 3 4
> **输出样例**
> 
>     2

## 题目描述
给定一个有n个点和m条边的有向图，图中可能存在重边和自环，并且++存在负权边++。问题是让我们求出从1号点到n号点的最短距离，如果无法从1号点走到n号点，输出-1。

## bellman-ford的思路及核心代码回顾

spfa本质上是对bellman-ford的一个优化，因此我们先回顾一下bellman-ford的思路及实现。

在bellman-ford中，使用的方式是每次把所有的边与目标点的距离都更新一步。用代码表示即为：
```python
for i in range(n):
    back = deepcopy(dist)
    for a, b, w in g[i]:
        dist[b] = min(dist[b],back[a] + w)
```

其中几个变量的定义如下：

1. g为图本身。g[i]表示的是a到b的距离是w；

2. dist数组表示的是其中元素到目标点的距离；

3. back数组表示的是dist数组的备份，既最外层循环的上一轮循环中的dist；


我们注意看代码中点b的更新。其中dist[b]表示此时b到目标点的距离，而在本次循环中，a与b之间有一条边，权重是w，那么如果**目标点到a的距离，加上a到b的距离，小于目标点直接到b的话，说明找到了更短的一条路径**（既先到a再到b），所以可以用(back[a]+w)更新dist[b]。

那么这样做有什么问题呢？显而易见的，**如果b到a再到目标点的距离没有b直接到目标点更短，则本次的遍历就是无效的**，也就是说dist[b]不会被更新，而spfa正是优化了这种无效的遍历。（这段话需要认真体会）

## Code

从代码中我们可以看出来核心思路是完全没有变化的。既代码中的注释部分：

如果目标点到当前点再到b 小于 目标点直接到b 则更新。

时间复杂度上，我们只遍历到目标点距离变小的点，所以时间复杂度平均为O(m)，而最坏则还是会达到bellman-ford的时间复杂u度O(nm)。

```python
from collections import defaultdict, deque
# 读入数据
n, m = list(map(int, input().split()))
g = defaultdict(lambda: defaultdict(lambda: int))
for _ in range(m):
    a, b, w = list(map(int, input().split()))
    g[a][b] = w

# 初始化距离
dist = [float('inf')] * (n + 10)
dist[1] = 0

# 判重数组，如果需要判断的点已经在队列，则不需要再加入
visited = [False] * (n + 10)
visited[1] = True

# 使用队列来存储，变小的点，初始化为目标点
q = deque()
q.append(1)

while q:
    # 弹出当前点  当前点不在队列所以判重数组置为False
    cur = q.popleft()
    visited[cur] = False
    # 遍历当前点可以到达的点b
    for b in g[cur]:
        # 如果目标点到当前点再到b 小于 目标点直接到b 则更新
        if dist[b] > g[cur][b] + dist[cur]:
            dist[b] = g[cur][b] + dist[cur]
            # 因为b是被更新过的点  所以可以更新它的可到达的点
            if not visited[b]:
                q.append(b)
                visited[b] = True

print(dist[n] if dist[n] != float('inf') else 'impossible')
```


