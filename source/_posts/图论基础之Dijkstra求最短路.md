---
title: 图论基础之Dijkstra求最短路
math: true
date: 2021-08-03
comment: true
tags:
- 经典必会
- Dijkstra
- AcWing算法基础课
- LeetCode
- 最短路问题
categories:
- 算法
- 图论
---

# [AcWing 849. Dijkstra求最短路 I](https://www.acwing.com/problem/content/851/)（图论中的基础最短路问题）
> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环，所有边权均为正值。
> 
> 请你求出 1 号点到 n 号点的最短距离，如果无法从 1 号点走到 n 号点，则输出 −1。
> 
> 输入格式
> 第一行包含整数 n 和 m。
> 
> 接下来 m 行每行包含三个整数 x,y,z，表示存在一条从点 x 到点 y 的有向边，边长为 z。
> 
> 输出格式
> 输出一个整数，表示 1 号点到 n 号点的最短距离。
> 
> 如果路径不存在，则输出 −1。
> 
> 数据范围
> 1≤n≤500,
> 1≤m≤105,
> 图中涉及边长均不超过10000。
> 
> 输入样例：
> 3 3
> 1 2 2
> 2 3 1
> 1 3 4
> 输出样例：
> 3

## 题目描述
如题，给定一个n个点和m条边的有向图，**所有边权均为正**。求1号点到n号点的最短距离，如果没办法从1号点到n号点，则输出-1。

## dijkstra思路

如题目标题所写，本题就是Dijkstra求最短路的一个标准应用场景。即，**求n个点m条边组成的图中，某个点到其他点的最短距离且边权为正**。
![](/assets/图论基础dij.png)

Dijkstra算法本身并不难理解，较麻烦的地方在于实现上，所以记住一个好用的模板还是很重要的。Dijkstra本质是贪心原理：

[在上图中，我们如果想知道【从1号点到其他点的最短距离】。]{.red}

1. 首先我们从a点出发，可以经过z1距离到b号点，经过z3距离到c号点。因此，目前我们可以得出初步结论a点到b点的距离是z1，a到c的距离是z3；

2. 由于a与b中没有其他点，所以z1一定就是a到b的最短距离；

3. 我们又注意到a到c点的距离为z3，但是z3一定是a到c的最短距离吗？

    ==不一定的，因为a到c还有另外一条路，即，a到b再到c，那么我们如何判断两条路径的哪条更短一些呢？==

    ==因为我们已经知道了z1是a到b的最短距离，那么我们直接用z1加上z2就是a到b再到c的距离，如果该距离比a直接到c更小，那么就可以用a到b的最小距离z1加上z2的结果，来更新a到c的最小距离；==

[如此，我们就求出了1号点a到其他两个点的最短距离分别是多少；]{.red}

## Code

### 朴素版

根据上面的分析，看一下我们在代码中需要如何进行操作。

1. 首先必须有一个数据结构来记录距离，我们选用数组，因为数组的下标就可以代表当前的点编号，而其对应的值就是它到1号点的距离。用dist表示；

2. 我们必须有一个数据结构来记录已经确定最短路径的点，因为分析中我们提到只有当z1为最短路径的时候，才可以用它加上z2来更新a到c。所以我们有必要记录已经确定的点。那么因为最短路径有且只有一个，所以我们用集合set来存储已经确定的最短距离的点。用st表示；

3. 必须有一个数据结构来存储给出的信息，即点a到b的距离z1。我们直接使用邻接矩阵来存储，即矩阵第一维表示出发点，第二维表示到达点，其对应的值表示边长的距离，也就是边的权重。例如：g[a][b]=z1；

4. 初始化dist，第一个点为0，其他所有点为正无穷；

5. 循环所有的点（1~n）。找到不在st中的距离最近的点t，放到st中，用t更新其他点的距离。更新条件为如果t到x的距离加上t到1号点的距离小于x直接到1号点的距离，则可以更新。当循环结束后，我们一定可以确定某一个点的最短距离（每次确定的都是不在st中且与1号点最近的点，贪心原理，证明过程不在此赘述）；

6. 把过程5再循环n遍就求出了n个点到1号点的最短距离。

```python
# Dijkstra算法
n, m = map(int, input().split())
# 本题中n的范围是500 m是10^5大于n^2 所以是稠密图 要用邻接矩阵 用g表示
# 其中为什么要n+1？这是让下标都从1开始，方便处理且符合人类的思考逻辑
g = [[float('inf')] * (n + 1) for _ in range(n + 1)]
# dist分析中已经提过，用来记录1号点到i的距离
dist = [float('inf')] * (n + 1)
# st 表示每个点数是否已经是确定的。
st = set()
# 初始化邻接矩阵。
for _ in range(m):
    x, y, z = map(int, input().split())
    # 因为题目中说可能有重边，也就是一个点到另外一个点可能有多个边
    # 所以我们只保存最短的那条边。
    g[x][y] = min(g[x][y], z)
# 初始化第一个点为0，就是它到自己的距离。
dist[1] = 0

# 注意要循环n遍，每遍都会确认一个点， 
# 确认的点为不在st中，并且与1号点最近的点
for _ in range(n):
    # 初始化t
    t = -1
    for i in range(1, n + 1):
        # 如果i不是已经确定的点， 
        # 并且是未赋值的（也就是等于初值-1）或者dist中t大于i说明i更短
        # 所以用i更新t， 并且最后把t加到集合中
        if i not in st and (t == -1 or dist[i] < dist[t]):
            t = i
    st.add(t)
    # 用t更新其他点的距离
    for j in range(1, n + 1):
        # 这里j直接到1号点，或者j先到t再到1号点，那个最小就用那个。
        dist[j] = min(dist[j], dist[t] + g[t][j])
# 最后求出1到n的最短距离就行了。
print(-1 if dist[-1] == float('inf') else dist[-1])
```

比较明显，时间复杂度是$O(n^2)$

### 堆优化

我们很容易发现上面的算法最大时间复杂度是O(n^2)的。又可以发现最耗时的其实是在**找不在st中并且当前距离1号点最小的数t**的这一步。那我们要**在一堆数中找出最小的数**可不可以借用什么现成的数据结构呢？没错，那就是堆。代码整体思路没有变化。

优化后，时间复杂度是$O(mlog_n)$。
```python
from collections import defaultdict
from heapq import heappush, heappop

n, m = map(int, input().split())
# 点的数量与边的数量差不多 所以是稀疏图 用邻接表存储
g = defaultdict(lambda: defaultdict(lambda: float('inf')))
for _ in range(m):
    x, y, z = map(int, input().split())
    g[x][y] = min(g[x][y], z)

dist = [float('inf')] * (n + 1)
dist[1] = 0
heap = []
# 需要把当前节点到1号点的距离，和当前节点编号都入堆
heappush(heap, (0, 1))
st = set()

while heap:
    # 弹出当前最小的点
    d, node = heappop(heap)
    # 如果当前点再st，则略过即可
    if node in st:
        continue
    st.add(node)
    # 枚举当前点可以到达的点p
    for t in g[node]:
        # 用t更新其他距离
        if dist[t] > d + g[node][t]:
            dist[t] = d + g[node][t]
            heappush(heap, (dist[t], t))
print(dist[-1] if dist[-1] != float('inf') else -1)
```

### 时间复杂度



实际应用一下我们的模板。


# [LeetCode 743. 网络延迟时间](https://leetcode.cn/problems/network-delay-time/)

## 题目描述

不再过多赘述题意，点击题目链接可以跳转到原题。读完题立刻就能发现，这道题也近乎是一道最短路问题的纯模板题。



## 题目分析

和我们上面分析的模板唯一不同的地方就在于【从某个节点k发出一个信号，需要多久才可以使所有点都收到信号】，这说明我们要求出k到所有点的最短距离，那么这些距离中最大的那个就是结果。

## Code
```python
def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
    g = [[float('inf')] * (n + 1) for _ in range(n + 1)]
    for x, y, z in times:
        g[x][y] = z
    dist = [float('inf')] * (n + 1)
    # 注意！！！唯有此处不同，从k点出发，那么初始化k到它自己的距离是0
    dist[k] = 0
    st = set()
    for _ in range(n):
        t = -1
        for i in range(1, n + 1):
            if i not in st and (t == -1 or dist[t] > dist[i]):
                t = i
        st.add(t)
        for j in range(1, n + 1):
            dist[j] = min(dist[j], dist[t] + g[t][j])
    res = max(dist[1:])
    return res if res < float('inf') else -1
```




