---
title: 图论基础之Floyd求多源汇最短路
math: true
date: 2021-10-10
comment: true
tags:
- 经典必会
- floyd
- AcWing算法基础课
- 最短路问题
categories:
- 算法
- 图论
---

# [AcWing 854. Floyd求最短路](https://www.acwing.com/problem/content/856/)
> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环，边权可能为负数。
> 
> 再给定 k 个询问，每个询问包含两个整数 x 和 y，表示查询从点 x 到点 y 的最短距离，如果路径不存在，则输出 impossible。
> 
> 数据保证图中不存在负权回路。
> 
> **输入格式**
> 第一行包含三个整数 n,m,k。
> 
> 接下来 m 行，每行包含三个整数 x,y,z，表示存在一条从点 x 到点 y 的有向边，边长为 z。
> 
> 接下来 k 行，每行包含两个整数 x,y，表示询问点 x 到点 y 的最短距离。
> 
> **输出格式**
> 共 k 行，每行输出一个整数，表示询问的结果，若询问两点间不存在路径，则输出 impossible。
> 
> **数据范围**
> 
>     1≤n≤200,
>     1≤k≤n2
>     1≤m≤20000,
>     图中涉及边长绝对值均不超过 10000。
> 
> **输入样例**
> 
>     3 3 2
>     1 2 1
>     2 3 2
>     1 3 1
>     2 1
>     1 3
> **输出样例**
> 
>     impossible
>     1

## 题目描述
给定一个有n个点和m条边的有向图，图中可能存在重边和自环，并且存在负权边。问题是让我们求++任意两点间++的最短距离。

## Floyd思路
Floyd使用动态规划的思想，能在O(n^3)的时间复杂度下求出任意两点间的最短路径，可以存在负权边，但是不能存在负权回路。

> 状态表示：d(k, i, j)表示从i点开始，只经过1~k这些点，最后到达j的最小距离。

> 状态转移：d(k ,i, j)可以由d(k-1, i, k) + d(k-1, k, j)转换而来。表示的含义是只经过1~k-1这些点，并且从i到k再从k到j。我们发现第一维k只能从k-1转换而来，因此可以忽略，所以有转移方程：d(i, j)=d(i, k)+d(k, j)。


首先k表示枚举出的所有顶点，那么从i到j只能存在两种情况：

1、直接就从i到j；

2、从i先到k再到j；

那么我们枚举出所有的顶点k，判断从i到k再到j是否比i直接到j距离更短，即检查d(i，k) + d(k，j)＜d(i，j) 是否成立。如果成立就更新d(i, j)。

如此，当所有的顶点k都被枚举过之后，d中存储的就是任意两点间的最短距离。

## Code
```python

import sys
n, M, K = map(int, input().split())

d = [[float('inf')] * (n + 1) for _ in range(n + 1)]
# 每一条边自己到自己的距离都是0
for i in range(1, n + 1):
    d[i][i] = 0
for _ in range(M):
    x, y, z = map(int, list(sys.stdin.readline().strip().split()))
    # 如果出现重边的话要取最短的一条
    d[x][y] = min(d[x][y], z)

# Floyd最关键的地方，外层循环是k。
for k in range(1, n + 1):
    for i in range(1, n + 1):
        for j in range(1, n + 1):
            d[i][j] = min(d[i][j], d[i][k] + d[k][j])

for _ in range(K):
    x, y = map(int, list(sys.stdin.readline().strip().split()))
    print('impossible' if d[x][y] == float('inf') else d[x][y]
```

