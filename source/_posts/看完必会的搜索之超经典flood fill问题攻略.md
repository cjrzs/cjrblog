---
title: 看完必会的搜索之超经典flood fill问题攻略
math: true
date: 2022-07-25
comment: true
tags:
- BFS
- 数组模拟队列
- Flood Fill
- AcWing
- LeetCode
categories:
- 算法
- 搜索与图论
valine:
  appId: TsHNizG5g9yTU8EUAa1EIkH5-gzGzoHsz #Your_appId
  appKey: KSmBGGzCyE4Pnop6qF4aD39B #Your_appkey
  placeholder: ヽ(○´∀`)ﾉ♪ # Comment box placeholder
  pageSize: 10 # Pagination size
  lang: zh-CN
  tagMember:
    master:
      - deea5a8d259d17182a53be1772e4c182
    friend:
      - deea5a8d259d17182a53be1772e4c182
---
# 前置知识
##  数组模拟队列
一般我们认为数组模拟队列是要比使用标准库的队列更快的。模拟队列中最常用的三个操作如下：

1. 初始化操作：
```cpp 伪代码
q[N]; // 初始化一个数组
int hh = 0, tt = -1; // 初始化头尾指针
```

2. 取出队列头/尾元素。
```cpp 伪代码
q[hh] // 队头 q.front()
hh ++ // 弹出队头 q.pop()
组合起来就是：q[hh ++ ]
同理，取出队尾： q[tt -- ]
```

3. 增加一个元素
```cpp 伪代码
q[++ tt ] = item // 在末尾添加一个元素item
```

## 四连通的搜索方式
四连通，顾名思义就是**矩阵中的一个点可以和周围`上下左右`四个方向连通**，如图所示，从红色格子出发可以走到其他四个绿色格子：
![](/assets/fd34c5e0-0eb5-49bd-bcec-ff9c40c90e7b.png)


我们使用数组坐标系来完成四连通的搜索。

**数组坐标系的x轴是向下的，y轴是向右的，也就是二维数组的`i,j`的方向**。

使当前点`{x, y}`向周围四个方向扩散，得到新的点`{a, b}`：
```cpp
int dx[4] = { 0,-1,0,1 };
int dy[4] = { -1,0,1,0 };
for (int i = 0; i < 4; i ++ ) {
    a = x + dx[i], b = y + dy[i];
    // do something...
}
```

## 八连通的搜索方式
八连通，顾名思义就是**八个方向都可以连通**，也就是**当前点可以扩展到其他八个方向**，如图所示，从红色格子出发可以走到其他八个绿色格子：
![](/assets/c097ede7-861d-4a55-b1b4-d6a07e1db0aa.png)

八连通如果再使用坐标系的形式可能会造成理解上的困难，因此，我们改为遍历的形式，仍然是把`{x, y}`扩散到`{a, b}`：
```
for (int a = x - 1; a <= x + 1; a ++ )
    for (int b = y - 1; b <= y + 1; b ++ ) {
        // 过滤掉中间的点{x, y}
        if (i == t.x and j == t.y) continue;
        // do something...
    }
```


# Flood Fill
> Flood Fill算法是从一个区域中提取若干个连通的点与其他相邻区域区分开（或分别染成不同颜色）的经典算法。因为其思路类似洪水从一个区域扩散到所有能到达的区域而得名。在GNU Go和扫雷中，Flood Fill算法被用来计算需要被清除的区域。

该算法所解决的问题不管在竞赛中还是面试中都非常常见，属于必须掌握那类的。

先从一个经典的例题开始，来探究该算法。




# [AcWing 1097. 池塘计数](https://www.acwing.com/problem/content/1099/)
> 农夫约翰有一片 N∗M 的矩形土地。
> 
> 最近，由于降雨的原因，部分土地被水淹没了。
> 
> 现在用一个字符矩阵来表示他的土地。
> 
> 每个单元格内，如果包含雨水，则用”W”表示，如果不含雨水，则用”.”表示。
> 
> 现在，约翰想知道他的土地中形成了多少片池塘。
> 
> 每组相连的积水单元格集合可以看作是一片池塘。
> 
> 每个单元格视为与其上、下、左、右、左上、右上、左下、右下八个邻近单元格相连。
> 
> 请你输出共有多少片池塘，即矩阵中共有多少片相连的”W”块。
> 
> **输入格式**
> 第一行包含两个整数 N 和 M。
> 
> 接下来 N 行，每行包含 M 个字符，字符为”W”或”.”，用以表示矩形土地的积水状况，字符之间没有空格。
> 
> **输出格式**  
> 输出一个整数，表示池塘数目。
> 
> **数据范围**  
> 1≤N,M≤1000  
> 
> **输入样例**  
> 10 12  
> W........WW.  
> .WWW.....WWW  
> ....WW...WW.  
> .........WW.  
> .........W..  
> ..W......W..  
> .W.W.....WW.  
> W.W.W.....W.  
> .W.W......W.  
> ..W.......W.  
> **输出样例**  
> 3

## 题目描述
给出一个`n*m`的二维矩阵，其中有的格子是水池，有的格子是土地，只要当前水池周围八个角度任意一个也是水池则表示这两个格子是连通的，求 **总共有多少连通块**。

在示例1中总共有3个连通块，如下图所示：
![](/assets/d69cb8d6-3c7d-45e3-93fe-791eca1f3665.png)

## 题目分析
一般而言，我们会使用BFS来解决Flood Fill问题。BFS是基于迭代的算法，所以**搜索层数很深的情况下**也不存在**递归栈不够用**的问题。

本题是一个标准的模板型题目，所以思路也很简单：**遍历所有的点，如果该点是水池且未搜索过，则说明找到一个新的联通块，则进入BFS环节，在BFS中把与该点连通的水池全部标记为已搜索过，表示他们与目前的点属于一个连通块。**

## Code
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

#define x first
#define y second

typedef pair<int, int> PII;

const int N = 1010, M = N * N;
int n, m;
bool st[N][N];
PII q[M]; // 数组存储所有的坐标
char g[N][N];

// 在BFS中把与该点连通的所有点都标记为搜索过，意为他们组成一个连通块
void bfs(int x, int y) {
    // BFS的使用队列 这里用数组来模拟。
    int hh = 0, tt = 0;
    q[0] = {x, y};
    st[x][y] = true;
    while (hh <= tt) {
        PII t = q[hh ++ ];
        // 八连通 由{x, y}扩散到其他八个方向。
        for (int i = t.x - 1; i <= t.x + 1; i ++ ) {
            for (int j = t.y - 1; j <= t.y + 1; j ++ ) {
                if (i == t.x and j == t.y) continue;
                // 判断新坐标是否在合法范围
                if (i < 0 or i >= n or j < 0 or j >= m) continue;
                // 判断新坐标是否被遍历过 或者 不是水池
                if (st[i][j] or g[i][j] == '.') continue;
                st[i][j] = true;
                q[++ tt] = {i, j};
            }
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i ++ ) 
        for (int j = 0; j < m; j ++ )
            cin >> g[i][j];
    int cnt = 0; // 记录连通块的数量
    for (int i = 0; i < n; i ++ )
        for (int j = 0; j < m; j ++ ) {
            // 遇到未搜索的水池 则连通块++ 进入BFS
            if (!st[i][j] and g[i][j] == 'W') {
                cnt += 1;
                bfs(i, j);
            }
        }
    cout << cnt << endl;
    return 0;
}
```

# [AcWing 1098. 城堡问题](https://www.acwing.com/problem/content/1100/)

> ![图1](/assets/db66b7c5-5322-47dd-8688-ecb9dd963420.png)
> 
> 图1是一个城堡的地形图。
> 
> 请你编写一个程序，计算城堡一共有多少房间，最大的房间有多大。
>
> 城堡被分割成 m∗n个方格区域，每个方格区域可以有0~4面墙。
>
> 注意：墙体厚度忽略不计。
>
> **输入格式**  
> 第一行包含两个整数 m 和 n，分别表示城堡南北方向的长度和东西方向的长度。
>
> 接下来 m 行，每行包含 n 个整数，每个整数都表示平面图对应位置的方块的墙的特征。
>
> 每个方块中墙的特征由数字 P 来描述，我们用1表示西墙，2表示北墙，4表示东墙，8表示南墙，P 为该方块包含墙的数字之和。
>
> 例如，如果一个方块的 P 为3，则 3 = 1 + 2，该方块包含西墙和北墙。
> 
> 城堡的内墙被计算两次，方块(1,1)的南墙同时也是方块(2,1)的北墙。
> 
> 输入的数据保证城堡至少有两个房间。
> 
> **输出格式**  
> 共两行，第一行输出房间总数，第二行输出最大房间的面积（方块数）。
> 
> **数据范围**  
> 1≤m,n≤50,  
> 0≤P≤15   
> 
> **输入样例**  
> 4 7   
> 11 6 11 6 3 10 6   
> 7 9 6 13 5 15 5   
> 1 10 12 7 13 7 5   
> 13 11 10 8 10 12 13   
> 
> **输出样例**  
> 5  
> 9

## 题目描述
给出一个四连通的矩阵，所有**连通的格子共同组成一个房间**，每个格子的面积为`1`，求**房间总数 和 最大的房间面积**。

## 题目分析
本题和上题非常相似，不同之处是我们**需要统计最多 有多少个格子是连通的**。

而本题最难的地方在于**输入的数据就挺奇怪的**。

首先要明白输入的格式：输入数据是 **每个格子（而不是格子上的边）**，而每个格子的四周是否有墙，通过该元素值的二进制位判断。

比如：第一个格子是11，他的二进制位是1011，说明有三面是有墙的，一面没有墙，那么没有墙的那一面就可以扩展到其他格子。
![](/assets/813dba98-99a7-4f26-963c-547c8627e0ad.png)

## Code
```CPP
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

typedef pair<int, int> PII;

#define x first
#define y second
const int N = 60, M = N * N;
int n, m;
PII q[M];
int g[N][N];
bool st[N][N];

// 返回当前房间的面积
int bfs(int sx, int sy) {
    int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    st[sx][sy] = true;
    int area = 0;
    while (hh <= tt) {
        PII t = q[hh ++ ]; // 取出队头元素
        area ++ ;
        // 四连通扩散
        for (int i = 0; i < 4; i ++ ) {
            int a = t.x + dx[i], b = t.y + dy[i];
            if (a < 0 or a >= n or b < 0 or b >= m) continue; // 判断越界
            if (st[a][b]) continue; // 判断是否重复遍历
            // 本题关键 从当前格子权重的二进制位中取出当前第i位
            // 不是1表示没有墙 如果没有墙才能扩散到{a, b}
            if (g[t.x][t.y] >> i & 1) continue;
            st[a][b] = true;
            q[++ tt] = {a, b};
        }
    }
    return area;
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i ++ ) 
        for (int j = 0; j < m; j ++ ) 
            cin >> g[i][j];
    int area = 0, cnt = 0;
    for (int i = 0; i < n; i ++ ) 
        for (int j = 0; j < m; j ++ ) {
            if (!st[i][j]) {
                cnt ++ ; // 计算房间数
                area = max(area, bfs(i, j)); // 维护房间最大面积
            }
        }
    cout << cnt << endl;
    cout << area << endl;
    return 0;
}
```

# [AcWing 1106. 山峰和山谷](https://www.acwing.com/activity/content/problem/content/1470/)
> 
> FGD小朋友特别喜欢爬山，在爬山的时候他就在研究山峰和山谷。
> 
> 为了能够对旅程有一个安排，他想知道山峰和山谷的数量。
> 
> 给定一个地图，为FGD想要旅行的区域，地图被分为 n×n 的网格，每个格子 (i,j) 的高度 w(i,j) 是给定的。
> 
> 若两个格子有公共顶点，那么它们就是相邻的格子，如与 (i,j) 相邻的格子有(i−1,j−1),(i−1,j),(i−1,j+1),(i,j−1),(i,j+1),(i+1,j−1),(i+1,j),(i+1,j+1)。
> 
> 我们定义一个格子的集合 S 为山峰（山谷）当且仅当：
> 
> 1. S 的所有格子都有相同的高度。
> 2. S 的所有格子都连通。
> 3. 对于 s 属于 S，与 s 相邻的 s′ 不属于 S，都有 ws>ws′（山峰），或者 ws<ws′（山谷）。
> 4. 如果周围不存在相邻区域，则同时将其视为山峰和山谷。
> 
> 你的任务是，对于给定的地图，求出山峰和山谷的数量，如果所有格子都有相同的高度，那么整个地图即是山峰，又是山谷。
> 
> **输入格式**  
> 第一行包含一个正整数 n，表示地图的大小。
> 
> 接下来一个 n×n 的矩阵，表示地图上每个格子的高度 w。
> 
> **输出格式**  
> 共一行，包含两个整数，表示山峰和山谷的数量。
> 
> **数据范围**  
> 1≤n≤1000,  
> 0≤w≤109  
> 
> **输入样例1**  
> 5  
> 8 8 8 7 7  
> 7 7 8 8 7  
> 7 7 7 7 7  
> 7 8 8 7 8  
> 7 8 8 8 8  
> 
> **输出样例1**  
> 2 1  
> 
> **输入样例2**  
> 5  
> 5 7 8 3 1  
> 5 5 7 6 6  
> 6 6 6 2 8  
> 5 7 2 5 8  
> 7 1 0 1 7   
> 
> **输出样例2**  
> 3 3

## 题目描述
给出一个`n*n`的矩阵，定义**元素相同的相邻坐标（八连通）组成的区域为山峰或者山谷**，如果与该区域相邻的元素都比它小，则该区域为山峰，如果都比它大则是山谷。

此外，如果不存在相邻区域，那么整片区域即是山峰又是山谷。

示例1的含义如下图所示，我们把示例1划分为三个区域，其中**2号区域与1号和3号区域相邻 并且2号区域所有元素都比它相邻元素小**，所以2号区域是山谷，而1号和3号区域的**所有元素都比它周围元素大**，所以它俩都是山峰，共有2个山峰和1一个山谷。（注意这里的关键词是**所有**，只有所有元素都比当前区域大或者小，才能认为当前区域是山峰或者山谷。）
![](/assets/c3fa9021-37dd-46d6-8159-d40d7c50ca27.png)

## 题目分析
从题目描述中可以看出来，本题仍然是个连通性的问题，因此也符合我们本文的主题Flood Fill算法。

与前两题明显不同的是，本题的**连通区域**有两种属性：山峰或者山谷。这也是本题的难点，我们要在**确定连通性的同时，把该区域和相邻区域的高低关系判断出来**。

我们可以**在进入BFS之前设定两个标记，一个判断相邻区域是否所有元素都比它大，一个判断相邻区域是否所有元素都比它小，这样最后就可以根据结果，判断当前区域是山峰还是山谷**。

## Code
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

#define x first
#define y second

typedef pair<int, int > PII;
const int N = 1010, M = N * N;

int n, g[N][N];
PII q[M];
bool st[N][N];

void bfs(int sx, int sy, bool& low, bool& large) {
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    st[sx][sy] = true;
    while (hh <= tt) 
    {
        PII t = q[hh ++ ];
        for (int i = t.x - 1; i <= t.x + 1; i ++ ) {
            for (int j = t.y - 1; j <= t.y + 1; j ++ ) {
                if (i < 0 or i >= n or j < 0 or j >= n) continue;
                if (i == t.x and j == t.y) continue;
                // 如果相邻区域与当前坐标元素不同 说明不属于相同区域
                if (g[i][j] != g[t.x][t.y]) {
                    // 判断 相邻区域 和 当前区域的 高低
                    if (g[i][j] > g[t.x][t.y]) large = true;
                    else low = true;
                // 如果当前区域和相邻区域是相同元素 则判断是否搜索过
                } else if (!st[i][j]) {
                    // 如果没搜索过加入队列
                    st[i][j] = true;
                    q[++ tt] = {i, j};
                }
            }
        } 
    }
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i ++ )   
        for (int j = 0; j < n; j ++ )
            cin >> g[i][j];
    
    int low_cnt = 0, large_cnt = 0; // 统计山峰和山谷
    for (int i = 0; i < n; i ++ ) 
        for (int j = 0; j < n; j ++ ) {
            if (!st[i][j]) {
                // low标记相邻元素是否 都 比当前区域{i, j}低的
                // large标记相邻元素是否 都 比当前区域{i, j}高
                bool low = false, large = false;
                bfs(i, j, low, large);
                // 如果相邻区域都不比当前区域低 说明当前区域是山谷，反之是山峰
                if (!low) low_cnt ++ ;
                // 注意这里不能用 else  因为本题说明了
                // 一片区域即可以是山峰 也可以是山谷 
                if (!large) large_cnt ++ ;
            }
        }
    cout << large_cnt << ' ' << low_cnt << endl;
    return 0;
}
```

# 模板
观察上面三道题，我们可以发现代码的整体结构是非常相似的，因此我们可以整理出来一个该类型题目的通用代码结构，我们姑且叫它模板。

> **适用问题**：给出一个矩阵，矩阵的部分坐标可以连通，求连通部分的个数、大小等属性。

```cpp 伪代码
PII q[M];
st[N][N]; // 判重
void bfs(int sx, int sy) {
    // 初始化BFS队列中的第一个元素
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    st[sx][sy] = true;
    for (扩散到相邻点{a, b}) {
        // 如果超出了边界 则过滤掉
        if (a < 0 or a >= n or b < 0 or b >= 0) continue;
        // 根据题目进行过滤
        // 如果没有搜索过 则进入主体逻辑
        if (!st[a][b]) {
            // do something ... 
            // 扩散到的点 标记为已搜索 并放入队列
            st[a][b] = true;
            q[++ tt] = {a, b}
        }
    }
}

int main() {
    for (矩阵横坐标 i)
        for (矩阵综坐标 j)
            if (!st[i][j]) {
                bfs(i, j);
                // 根据条件判断 连通块的属性。
                // 如：个数、最大面积等。
                cnt ++ ; // 以最常见的个数为例
            }
}
```

OK，有了模板之后，我们去LeetCode找点类似题目操作一下，感受一下秒杀的感觉~~

（为了节省篇幅，LeetCode部分不再放原题了，可以直接从标题链接跳过去。）

# [LeetCode 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/submissions/)
本题实在是太经典了，无数个面试中会遇到，无数个题单中会遇到，同时也是搜索问题的入门必会基础题。

## 题目描述
与“池塘计数”这题基本相同，不过本题是个四连通的题目吗，是标准的模板题，我们可以用该题目先检验一下模板。

## Code
```cpp
typedef pair<int, int> PII;
#define x first
#define y second
class Solution {
public:
    const static int N = 310, M = N * N;
    PII q[M];
    bool st[N][N];
    int n, m;
    int numIslands(vector<vector<char>>& nums) {
        n = nums.size(), m = nums[0].size();
        int res = 0;
        for (int i = 0; i < n; i ++ ) {
            for (int j = 0; j < m; j ++ ) {
                if (!st[i][j] and nums[i][j] == '1') {
                    bfs(i, j, nums);
                    res ++ ;
                }
            }
        }
        return res;
    }
    void bfs(int sx, int sy, vector<vector<char>>& nums) {
        int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
        int hh = 0, tt = 0;
        q[0] = {sx, sy};
        st[sx][sy] = true;
        while (hh <= tt) {
            PII t = q[ hh ++ ];
            for (int i = 0; i < 4; i ++ )  {
                int a = t.x + dx[i];
                int b = t.y + dy[i];
                if (a < 0 or a >= n or b < 0 or b >= m) continue;
                if (nums[a][b] == '0') continue;
                if (!st[a][b]) {
                    st[a][b] = true;
                    q[ ++ tt] = {a, b};
                }
            }
        }
    }
};
```


# [LeetCode 733. 图像渲染](https://leetcode.cn/problems/flood-fill/)
简单的模板题，甚至题目名就叫flood-fill。

## 题目描述
给出一个二维矩阵nums，和三个整数`sr、sc、color`，其中{sr,sc} 表示源点，要求从源点开始扩散，把**所有与源点连通并且权重与源点相同的点改成color（包括源点）**。

## 题目分析
本题就没有啥好分析的了，可以说是完完全全的套在了我们模板上。

## Code
```cpp
#define x first
#define y second
typedef pair<int, int> PII;
class Solution {
public:
     const static int N = 50, M = N * N;
     PII q[M];
     bool st[N][N];
    vector<vector<int>> floodFill(vector<vector<int>>& nums, int sr, int sc, int color) {
        int n = nums.size(), m = nums[0].size();
        int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
        int hh = 0, tt = 0;
        q[0] = {sr, sc};
        st[sr][sc] = true;
        while (hh <= tt) {
            PII t = q[hh ++ ];
            for (int i = 0; i < 4; i ++ ) {
                int a = t.x + dx[i], b = t.y + dy[i];
                if (a < 0 or a >= n or b < 0 or b >= m) continue;
                // 过滤掉与源点权重不同的点
                if (nums[a][b] != nums[sr][sc]) continue;
                if (!st[a][b]) {
                    // 将符合条件的点染色成color
                    nums[a][b] = color;
                    q[++ tt] = {a, b};
                    st[a][b] = true;
                }
            }
        }
        nums[sr][sc] = color;
        return nums;
    }
};
```

# [LeetCode 130. 被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)
同样属于是标准的Flood Fill问题。

## 题目描述
给出一个矩阵，矩阵中有两种类型的元素：“X”和“O”。像下围棋一样，当“O”被“X”完全包围的时候，就要把所有的“O”全部变成“X”。其中**在矩阵边缘的“O”永远不可能被完全包围**。

## 题目分析
与前面的题目不同，本题**如果遍历所有的棋子，那么就需要判断那个棋子所在区域是否能连接边界**，非常的麻烦。

因此，我们换一个思路，**既然题目中说明，与边界连接的“O”不会被完全包围，那么我们只需要从四个边界的“O”向内进行搜索，把所有与边界相连的“O”找出来，那么剩下的坐标就应该全部都是“X”**。

本题的重点是**从边界进行搜索**。

## Code
```cpp
typedef pair<int, int> PII;
#define x first
#define y second
class Solution {
public:
    const static int N = 210, M = N * N;
    bool st[N][N];
    int n, m;
    void solve(vector<vector<char>>& nums) {
        n = nums.size(), m = nums[0].size();
        // 从左右边界上的“O”进行扩展
        for (int i = 0; i < n; i ++ ) {
            if (nums[i][0] == 'O') bfs(i, 0, nums);
            if (nums[i][m - 1] == 'O') bfs(i, m - 1, nums);
        }
        // 从上下边界上的“O”进行扩展
        for (int i = 0; i < m; i ++ ) {
            if (nums[0][i] == 'O') bfs(0, i, nums);
            if (nums[n - 1][i] == 'O') bfs(n - 1, i, nums);
        }
        for (int i = 0; i < n; i ++ ) {
            for (int j = 0; j < m; j ++ ) {
                if (nums[i][j] == '#') nums[i][j] = 'O';
                else nums[i][j] = 'X';
            }
        }
    }
    // BFS 把扩展到的“O”全部标记上“#” 
    // 这样我们只要判断坐标是不是“#” 就知道它是“O”还是“X”
    void bfs(int sx, int sy, vector<vector<char>>& nums) {
        int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
        PII q[M];
        int hh = 0, tt = 0;
        q[0] = {sx, sy};
        nums[sx][sy] = '#';
        st[sx][sy] = true;
        while (hh <= tt) {
            PII t = q[hh ++ ];
            for (int i = 0; i < 4; i ++ ) {
                int a = t.x + dx[i], b = t.y + dy[i];
                if (a < 0 or a >= n or b < 0 or b >= m) continue;
                if (!st[a][b] and nums[a][b] == 'O') {
                    nums[a][b] = '#';
                    st[a][b] = true;
                    q[++ tt] = {a, b};
                }
            }
        }
    }
};
```

# [LeetCode 529. 扫雷游戏](https://leetcode.cn/problems/minesweeper/)
典型的阅读理解题目，如果没有真的玩过“扫雷”这个游戏，建议去找一个玩两盘，不然还真挺难理解它。

## 题目描述
给出一个二维矩阵，每个矩阵的格子有两种标记，其中“M”代表地雷，“E”代表方块，给出一个点击坐标，问**返回点击之后的矩阵**。遵循“扫雷”游戏的规则：
1. 如果八连通的角度上都没有“地雷”，则当前位置就是一块“空地”，当点击到空地的时候要把所有相连接的“空地”全部打开。
2. 如果点击到的坐标 周围有地雷的时候，那么就把该坐标 标记为周围“地雷”的数量（八连通）。
3. 如果点击到地雷，直接爆炸，游戏结束。

## 题目分析
首先，我们要看一下本题和我们本文模板的关系，注意到游戏规则中的第一条，即，**当点击到一块空地的时候，要把和它连通的所有空地都找出来**，所以，我们就可以用模板来完成这一步啦。

然而，其中又有需要标记数字的格子，那我们怎么区分出来呢？很简单，**对于每一个格子我们都判断一下，它是不是需要填数字的格子，只有它不需要填数字，我们才能判断它需不需要根据连通性被带出来**。

因此，我们可以构建如下的算法步骤：
1. 把源点（点击的坐标）放到队列里面；
2. 对于队列中的每个点`x`，首先记录它八个方向上的相邻坐标的地雷个数；
3. 如果有地雷，就把当前点`x`变成地雷个数；
4. 如果没有地雷，则说明该点是个空点，那我们再套模板，找到“空地”连通区域。

## Code
```cpp
typedef pair<int, int> PII;
#define x first
#define y second
class Solution {
public:
    const static int N = 60, M = N * N;
    PII q[M];
    bool st[N][N];
    int n, m;
    vector<vector<char>> updateBoard(vector<vector<char>>& nums, vector<int>& click) {
        n = nums.size(), m = nums[0].size();
        int sx = click[0], sy = click[1];
        // 点到地雷 直接爆炸 不需要BFS
        if (nums[sx][sy] == 'M') nums[sx][sy] = 'X';
        else bfs(nums, sx, sy);
        return nums;
    }
    void bfs(vector<vector<char>>& nums, int sx, int sy) {
        int hh = 0, tt = 0;
        q[0] = {sx, sy};
        st[sx][sy] = true;
        while (hh <= tt) {
            PII t = q[hh ++ ];
            int cnt = 0;
            // 先统计当前点周围的地雷个数。
            for (int i = t.x - 1; i <= t.x + 1; i ++ ) {
                for (int j = t.y - 1; j <= t.y + 1; j ++ ) {
                    if (i == t.x and j == t.y) continue;
                    if (i < 0 or i >= n or j < 0 or j >= m) continue;
                    if (nums[i][j] == 'M') cnt += 1;
                }
            }
            // 如果有地雷 则该点变成地雷数量
            if (cnt > 0) nums[t.x][t.y] = cnt + '0';
            // 如果没有地雷 我们就套模板 找连通区域
            else {
                // 把当前区域变成空地 这里为啥不用判断当前区域是不是地雷？
                // 因为点击的点若是地雷不会进入BFS，
                // 而在搜索过程中，如果扩散到的点是地雷，则该点不需要进行搜索。
                nums[t.x][t.y] = 'B';
                for (int i = t.x - 1; i <= t.x + 1; i ++ ) {
                    for (int j = t.y - 1; j <= t.y + 1; j ++ ) {
                        // cout << i << ' ' << j << ' ' << endl;
                        if (i == t.x and j == t.y) continue;
                        if (i < 0 or i >= n or j < 0 or j >= m) continue;
                        // 注意这里 只有空地才可以扩展
                        if (!st[i][j] and nums[i][j] == 'E') {
                            
                            st[i][j] = true;
                            q[ ++ tt] = {i, j};
                        }
                    }
                }
            }
        }
    }
};
```

# 后记
本文共有三部分内容，前置知识，一个模板，以及七道题，本来想补充到十个题，但是感觉这七个做完之后，对于相似类型的题目，已经可以有一种基本掌握了，读者如果觉得掌握的不好，还可以继续在LeeCode上搜索“深度优先搜索”这个标签，其中有相当一部分类似题目可供练习。

**END**
