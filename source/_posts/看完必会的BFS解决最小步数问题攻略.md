---
title: 看完必会的BFS解决最小步数问题攻略
math: true
date: 2022-08-06
comment: true
tags:
- AcWing算法提高课
- BFS
- 经典必会
- 最短路问题
- 最小步数问题
- 二进制

categories:
- 算法
- 搜索
--- 

# 前置知识
> [看完必会的搜索之超经典flood fill问题攻略](https://cjrzs.github.io/%E7%9C%8B%E5%AE%8C%E5%BF%85%E4%BC%9A%E7%9A%84%E6%90%9C%E7%B4%A2%E4%B9%8B%E8%B6%85%E7%BB%8F%E5%85%B8flood%20fill%E9%97%AE%E9%A2%98%E6%94%BB%E7%95%A5/)，[看完必会的BFS解决最短路问题攻略](https://cjrzs.github.io/%E7%9C%8B%E5%AE%8C%E5%BF%85%E4%BC%9A%E7%9A%84BFS%E8%A7%A3%E5%86%B3%E6%9C%80%E7%9F%AD%E8%B7%AF%E9%97%AE%E9%A2%98%E6%94%BB%E7%95%A5/)。本文所讲的题型与前置文章属于类似问题，因此需要先熟悉前置文章中**数组模拟队列**、**由坐标向其他方向扩散**等基础操作，**重复内容不会再讲**。

# 用BFS解决最小步数问题

在[前文](https://cjrzs.github.io/%E7%9C%8B%E5%AE%8C%E5%BF%85%E4%BC%9A%E7%9A%84BFS%E8%A7%A3%E5%86%B3%E6%9C%80%E7%9F%AD%E8%B7%AF%E9%97%AE%E9%A2%98%E6%94%BB%E7%95%A5/)中的**最短路模型**中，目的是**求一个点到另一个点的最短距离**。而在本文的**最小步数模型**中，目的是**求一个状态到另外一个状态的最小步数**。

我们依旧是从一个标准的模板题作为切入点。

# [AcWing 1107. 魔板](https://www.acwing.com/problem/content/1109/)

> Rubik 先生在发明了风靡全球的魔方之后，又发明了它的二维版本——魔板。
> 
> 这是一张有 8 个大小相同的格子的魔板：
> 
> 1 2 3 4
> 8 7 6 5
> 我们知道魔板的每一个方格都有一种颜色。
> 
> 这 8 种颜色用前 8 个正整数来表示。
> 
> 可以用颜色的序列来表示一种魔板状态，规定从魔板的左上角开始，沿顺时针方向依次取出整数，构成一个颜色序列。
> 
> 对于上图的魔板状态，我们用序列 (1,2,3,4,5,6,7,8) 来表示，这是基本状态。
> 
> 这里提供三种基本操作，分别用大写字母 A，B，C 来表示（可以通过这些操作改变魔板的状态）：
> 
> A：交换上下两行；
> B：将最右边的一列插入到最左边；
> C：魔板中央对的4个数作顺时针旋转。
> 
> 下面是对基本状态进行操作的示范：
> 
> A：
> 
> 8 7 6 5
> 1 2 3 4
> B：
> 
> 4 1 2 3
> 5 8 7 6
> C：
> 
> 1 7 2 4
> 8 6 3 5
> 对于每种可能的状态，这三种基本操作都可以使用。
> 
> 你要编程计算用最少的基本操作完成基本状态到特殊状态的转换，输出基本操作序列。
> 
> 注意：数据保证一定有解。
> 
> 输入格式
> 输入仅一行，包括 8 个整数，用空格分开，表示目标状态。
> 
> 输出格式
> 输出文件的第一行包括一个整数，表示最短操作序列的长度。
> 
> 如果操作序列的长度大于0，则在第二行输出字典序最小的操作序列。
> 
> 数据范围
> 输入数据中的所有数字均为 1 到 8 之间的整数。
> 
> 输入样例：
> 2 6 8 4 5 7 3 1
> 输出样例：
> 7
> BCABCCB

## 题目描述
给出一个`2 * 4`的矩阵，用数字`1 ~ 8`**顺序**标识，作为基本状态。并且有三种操作变换矩阵，给出一个目标状态，目标是求出**至少几次变换**能把矩阵从**基本状态**变到**目标状态**。

## 题目分析
如果我们把**三种状态的转换**，看做是**从当前坐标可以扩展到其他三个方向的坐标**。把**起始状态**看做是**起点**，把**目标状态**看做是**终点**，那么，该问题就变成了**给出一个坐标，每次可以向三个方向扩散，问 至少多少步可以走到终点**。

这样就变成了一个标准的**BFS求最短路问题**。

## Code
该题目虽然分析起来不难，但是实现可能就比较复杂了。

我们使用最简单的思路来完成。
1. 模拟题目的描述，用一个`2*4`的矩阵来存储数据；
2. 用字符串来存储状态，在操作的时候将字符串转换为矩阵，并在操作后还原成字符串；
3. 注意本题的输出格式中，要求**有状态变换的时候**，输出操作序列。

```cpp
#include <iostream>
#include <cstring>
#include <unordered_map>
#include <algorithm>
#include <queue>

using namespace std;

char g[2][4]; // 存储矩阵数据
unordered_map<string, int> dist; // BFS中存储每个状态的最小步数
// 存储 每个状态 是由 哪个状态经 过何种操作 转换而来的。
unordered_map<string, pair<string, char>> pre;

/*
代表状态的字符串 按照题目给出的顺序 变成2*4的矩阵 方便进行变换
例如：
12345678 -> 1 2 3 4
            8 7 6 5
*/ 
void set(string state) {
    for (int i = 0; i < 4; i ++ ) g[0][i] = state[i];
    for (int i = 7, j = 0; j < 4; i --, j ++ ) g[1][j] = state[i];
}

/*
把2*4的矩阵按照题目给出的顺序 转换成 代表状态的字符串
例如：
1 2 3 4 -> 12345678
8 7 6 5
*/
string get() {
    string res;
    for (int i = 0; i < 4; i ++ ) res += g[0][i];
    for (int i = 3; i >= 0; i -- ) res += g[1][i];
    return res;
}

/* 模拟第一个操作，交换矩阵上下两行 */
string move1(string state) {
    set(state);
    for (int i = 0; i < 4; i ++ ) swap(g[0][i], g[1][i]);
    return get();
}

/* 模拟第二个操作，将最右边的一列插入到最左边 */
string move2(string state) {
    set(state);
    int v0 = g[0][3], v1 = g[1][3];
    for (int i = 3; i >= 0; i -- ) {
        g[0][i] = g[0][i - 1];
        g[1][i] = g[1][i - 1];
    }
    g[0][0] = v0, g[1][0] = v1;
    return get();
}

/* 模拟第三个操作，魔板中央对的4个数作顺时针旋转。 */
string move3(string state) {
    set(state);
    int v = g[0][1];
    g[0][1] = g[1][1];
    g[1][1] = g[1][2];
    g[1][2] = g[0][2];
    g[0][2] = v;
    return get();
}

// 标准的BFS求最短路径 并且 打印出路径
int bfs(string start, string end) {
    if (start == end) return 0;
    queue<string> q;
    q.push(start);
    dist[start] = 0;
    while (!(q.empty())) {
        string t = q.front(); // 当前状态
        q.pop();
        string m[3]; // 当前状态 扩散到其它三个方向
        m[0] = move1(t);
        m[1] = move2(t);
        m[2] = move3(t);
        for (int i = 0; i < 3; i ++ ) {
            if (dist[m[i]] == 0) {
                dist[m[i]] = dist[t] + 1;
                // 记录下一个状态 是由当前状态t 执行哪个操作转换的
                pre[m[i]] = {t, 'A' + i}; 
                q.push(m[i]);
                if (m[i] == end) return dist[m[i]];
            }
        }
    }
    return -1;
}

int main() {
    string start, end;
    for (int i = 0; i < 8; i ++ ) {
        int x;
        cin >> x;
        end += char(x + '0');
    }
    for (int i = 1; i <= 8; i ++ ) start += char(i + '0');
    int step = bfs(start, end);
    cout << step << endl;
    // 倒推一遍 求出正向路径
    if (step) {
        string res;
        while (start != end) {
            res += pre[end].second;
            end = pre[end].first;
        }
        reverse(res.begin(), res.end());
        cout << res << endl;
    }
    return 0;
}
```

# [AcWing 1105. 移动玩具](https://www.acwing.com/problem/content/description/1107/)

> 在一个 4×4 的方框内摆放了若干个相同的玩具，某人想将这些玩具重新摆放成为他心中理想的状态，规定移动时只能将玩具向上下左右四个方向移动，并且移动的位置不能有玩具，请你用最少的移动次数将初始的玩具状态移动到目标状态。
> 
> **输入格式**  
> 前四行表示玩具的初始状态，每行 4 个数字 1 或 0，1 表示方格中放置了玩具，0 表示没有放置玩具。
> 
> 接着是一个空行。
> 
> 接下来四行表示玩具的目标状态，每行 4 个数字 1 或 0，意义同上。
> 
> **输出格式**  
> 输出一个整数，表示所需要的最少移动次数。
> 
> **输入样例**
>
>     1111
>     0000
>     1110
>     0010
>     
>     1010
>     0101
>     1010
>     0101
> **输出样例**
>
>     4

## 题目描述
给出一个`4*4`的01矩阵，其中**1可以向0移动，移动后原本的1变成0，移动到的0变成1，0不可以移动**。问给出指定的矩阵状态，需要移动多少次。

## 题目分析
从题目中可以看出来，本题同样是标准的最小步数模型，因此我们直接套用最短路模板就可以了。而本题的难点在于状态的转换，对于`4*4`的矩阵，怎样才能更方便的从一个状态转换到另外一个状态呢？

我们发现**矩阵的大小是固定的，也就是只有16位，并且除了0就是1**，因此，可以使用**二进制**的方式来存储矩阵状态。

## Code

本题的难点在于**二进制的熟练程度**，以及**二维坐标与一维坐标之间的转换**。

```cpp
#include <iostream>
#include <queue>
#include <unordered_map>

using namespace std;
const int N = 4;
int start, e;
unordered_map<int, int> dist;

int bfs() {
    queue<int> q;
    q.push(start);
    dist[start] = 0;
    int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
    while (q.size()) {
        int t = q.front();
        q.pop();
        if (t == e) return dist[t]; // 走到了结束状态
        for (int i = 0; i < N * N; i ++ ) {
            // 没有玩具（也就是 状态t 的 i位 为0） 不能移动
            if (!(t >> i & 1)) continue;
            // 把一维上的i 转换成二维的坐标{x,y}
            int x = i % N, y = i / N; 
            // 四连通的扩展
            for (int k = 0; k < 4; k ++ ) {
                int a = x + dx[k], b = y + dy[k];
                if (a < 0 or a >= N or b < 0 or b >= N) continue;
                // 把新扩展的坐标{a,b}转换到一维中的位置j
                int j = b * N + a;
                // 如果t的j位是玩具 则不能扩展
                if (t & (1 << j)) continue;
                // 使用临时的tn存储当前状态 避免操作原本的t
                // 因为t要扩展四次，不能在本次就更改
                int tn = t;
                tn ^= 1 << i; // 把t的第i位上的1变成 0
                tn ^= 1 << j; // 把t的第j位上的0变成 1
                // 本状态如果被扩展过就可以跳过了 减少搜索次数
                if (dist[tn]) continue; 
                dist[tn] = dist[t] + 1;
                q.push(tn);
            }
        }
    }
    return -1;
}

int main() {
    char c;
    for (int i = 0; i < N; i ++ )
        for (int j = 0; j < N; j ++ ) {
            cin >> c;
            if (c == '1') start |= 1 << i * N + j;
        }
    for (int i = 0; i < N; i ++ )
        for (int j = 0; j < N; j ++ ) {
            cin >> c;
            if (c == '1') e |= 1 << i * N + j;
        }
    // cout << start << ' ' << e << endl;
    cout << bfs() << endl;
    return 0;
}
```


# 模板：[LeetCode 433. 最小基因变化](https://leetcode.cn/problems/minimum-genetic-mutation/)

## 题目描述
由八个字符组成的字符串，其中每个字符都是[A,C,G,T]之一，给出一个初始字符串和一个目标字符串和一个判重数组，每次变化可以使字符串中的一个字符变为其他字符，但是只有在判重数组中存在的字符串才是合法变化，问从初始串到目标串至少需要几次变化。

## 题目分析
对比上面两题来说，本题更简单，更加的接近模板。以本题为基准，我们来看下一个最小步数的基本模板是什么样子。

## 模板Code
```cpp
class Solution {
public:
    int minMutation(string start, string end, vector<string>& bank) {
        // 用set存一下判重数组 加速判重
        unordered_set<string> S;
        for (auto& s: bank) S.insert(s);
        queue<string> q; // 初始化队列
        unordered_map<string, int> dist; // 初始化距离
        dist[start] = 0; // 初始化起始点 距离为0
        q.push(start); // 起始点入队
        char op[4] = {'A', 'C', 'G', 'T'};
        while (q.size()) { // 循环存取队列
            string t = q.front(); // 取出队头 作为当前处理的状态
            q.pop();
            // 找到目标状态了 返回结果
            if (t == end) return dist[t];
            // 根据题目要求进行状态转换
            // 本题的状态转换规则是 *当前状态的每个字符都可以用其他字符替换*
            for (int i = 0; i < t.size(); i ++ ) {
                string s = t; // 缓存当前状态 因为当前状态的字符i 需要进行三次变换
                for (char c: op) {
                    s[i] = c; // 变换字符i
                    // 合法判断 是否在判重数组 以及 是否已经遍历过该状态
                    if (S.count(s) and dist.count(s) == 0) {
                        // 模板 新状态由 *旧状态步数+1* 换而来转
                        dist[s] = dist[t] + 1; 
                        q.push(s); // 合法的新状态入放到队尾
                    }
                }
            }
        }
        return -1;
    }
};
```

# 总结

在上面三个例题中，我们可以发现在最小步数模型中，题目本身的思路并不是难点，最难的地方在于**根据题目的条件进行状态转换**。

比如，在第一个题目[AcWing 1107. 魔板](https://www.acwing.com/problem/content/1109/)中，题目明确给出了状态如何变换，我们要思考的点在于**把矩阵变换成比较方便表示状态的字符串**。

在第二个题目[AcWing 1105. 移动玩具](https://www.acwing.com/problem/content/description/1107/)中，我们观察矩阵的大小是固定的，且矩阵中只存在0和1，所以我们思考的点在于**使用二进制来表示状态**。

**END**
