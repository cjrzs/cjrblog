---
title: 看完必会的单调队列优化DP攻略
math: true
date: 2022-07-22
comment: true
tags:
- 单调队列
- 前缀和
- 二分
- AcWing算法提高课
- LeetCode
categories:
- 算法
- 动态规划
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
> 前置知识：[滑动窗口](https://cjrzs.github.io/%E7%9C%8B%E5%AE%8C%E5%BF%85%E4%BC%9A%E7%9A%84%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E5%85%A5%E9%97%A8%E6%94%BB%E7%95%A5/), [前缀和](https://cjrzs.github.io/%E5%AE%9E%E7%94%A8%E7%9A%84%E7%AE%97%E6%B3%95%E5%B0%8F%E6%8A%80%E5%B7%A7--%E5%89%8D%E7%BC%80%E5%92%8C/)。这两个基本技巧是本文涉及题目的基础，应优先掌握。

# 单调队列优化
在动态规划的世界中，有那么一个小部落，这个部落里的题目都具有某些相同的特征：**在状态转移的部分，上一状态的取值范围是一个给定大小的窗口，而我们要维护窗口中元素的某种极值作为上一状态，此时就可以使用单调队列来维护滑动窗口的极值**。

尤其值得注意的一点是，这类的题目中通常伴随着“**连续**”、“**子序列**”等关键词。

# [AcWing 135. 最大子序和](https://www.acwing.com/problem/content/137/)
> 输入一个长度为 n 的整数序列，从中找出一段长度不超过 m 的连续子序列，使得子序列中所有数的和最大。
> 
> 注意： 子序列的长度至少是 1。
> 
> **输入格式**  
> 第一行输入两个整数 n,m。
> 
> 第二行输入 n 个数，代表长度为 n 的整数序列。
> 
> 同一行数之间用空格隔开。
> 
> **输出格式**  
> 输出一个整数，代表该序列的最大子序和。
> 
> **数据范围**  
> 1≤n,m≤300000   
>
> **输入样例**  
> 6 4  
> 1 -3 5 1 -2 3  
> **输出样例**  
> 7


## 题目描述
**在长度为n的一个整数序列中找出一段不超过m长度的子序列**，这里需要注意的是子序列意味着不可以跳着选，长度为m的子序列必须是连续的。而最后输出的是**子序列中所有元素和最大**的子序列。

## 题目示例
在长度为6的序列找出长度**不超过4**的子序列，使得**该子序列的和**最大。  

我们看下给出示例中的子序列都有哪些呢？
```
[1],[1,-3],[1,-3,5],[1,-3,5,1]
[-3],[-3,5],[-3,5,1],[-3,5,1,-2]
[5],[5,1],[5,1,-2],[5,1,-2,3]
[1],[1,-2],[1,-2,3]
[-2],[-2,3]
[3]
```

所以最后输出 **和最大的子序列的和**，也就是`[5,1,-2,3]=7`。

## 题目分析

1. 如果有对**滑动窗口**这种技巧有所了解的话，可以很快的发现，诶？这不就是**长度最长为m的滑动窗口求极值问题**吗？我们只要**对每个窗口，都求出该窗口内所有元素的和，然后实时更新窗口和的最大值，当枚举完所有的元素，最后的那个最大值就是答案**。

2. 那么，**如何快速求出一个窗口内所有元素的和呢？** 关于这一点其实也有一个小技巧，即**前缀和**，它的作用就是在O(1)的时间内求出某个区间内所有元素的和。

3. 我们知道求区间和的公式是`sum[r] - sum[l-1]`，也就是说，想让区间和越大，那么只需要让队列（窗口）头部元素最小就可以了。

## Code
```cpp
#include <iostream>

using namespace std;
const int N = 300010;
int n, m;
int s[N]; // 前缀和数组
int q[N]; // 单调队列求滑动窗口

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i ++ ) {
        cin >> s[i];
        s[i] += s[i - 1]; // 计算前缀和
    }
    // 在数组模拟队列的方法里，我们用hh表示队列头，用tt表示队列尾
    // 一般来讲队列尾tt要初始化为-1，这里初始化为0，是因为此时我们
    // 默认队列中已经有一个元素，就是0，对应前缀和的第一个元素s[0]=0
    int hh = 0, tt = 0; 
    int res = -0x3f3f3f;
    // 滑动窗口模板，窗口中队头保存当前的最小值
    for (int i = 1; i <= n; i ++ ) {
        while (hh <= tt && i - q[hh] > m) hh ++ ; // 维护窗口大小不超过m
        res = max(res, s[i] - s[q[hh]]); // 实时维护最大的子序列和
        // 单调队列要维护最小值，那么当前元素比队尾还小，说明队尾元素没有价值了，出队即可。
        while (hh <= tt && s[i] <= s[q[tt]]) tt -- ; 
        q[++ tt] = i; // 把当前元素入队，注意这里队列中保存的是下标
    }
    cout << res << endl;
    return 0;
} 
```


# [AcWing 1087. 修剪草坪](https://www.acwing.com/problem/content/1089/)
> 在一年前赢得了小镇的最佳草坪比赛后，FJ 变得很懒，再也没有修剪过草坪。
> 
> 现在，新一轮的最佳草坪比赛又开始了，FJ 希望能够再次夺冠。
> 
> 然而，FJ 的草坪非常脏乱，因此，FJ 只能够让他的奶牛来完成这项工作。
> 
> FJ 有 N 只排成一排的奶牛，编号为 1 到 N。
> 
> 每只奶牛的效率是不同的，奶牛 i 的效率为 Ei。
> 
> 编号相邻的奶牛们很熟悉，如果 FJ 安排超过 K 只编号连续的奶牛，那么这些奶牛就会罢工去开派对。
> 
> 因此，现在 FJ 需要你的帮助，找到最合理的安排方案并计算 FJ 可以得到的最大效率。
> 
> 注意，方案需满足不能包含超过 K 只编号连续的奶牛。
> 
> **输入格式**   
> 第一行：空格隔开的两个整数 N 和 K；
> 
> 第二到 N+1 行：第 i+1 行有一个整数 Ei。
> 
> **输出格式**  
> 共一行，包含一个数值，表示 FJ 可以得到的最大的效率值。
> 
> **数据范围**   
> 1≤N≤10^5,  
> 0≤Ei≤10^9
> 
> **输入样例**  
> 5 2  
> 1  
> 2  
> 3  
> 4  
> 5
> 
> **输出样例**  
> 12  
> 
> **样例解释**  
> FJ 有 5 只奶牛，效率分别为 1、2、3、4、5。
> 
> FJ 希望选取的奶牛效率总和最大，但是他不能选取超过 2 只连续的奶牛。
> 
> 因此可以选择第三只以外的其他奶牛，总的效率为 1 + 2 + 4 + 5 = 12。

## 题目描述
同上题类似，给出一个长度为n的序列s，要从中选择出一些元素使得最后这些**被选择元素的和 最大**，然而给出了一个规则，**不能选择连续超过m个元素**。

## 题目分析
如果我们不知道文章题目而是实际中遇到这题，看到题目中有**安排方案**这种关键字，也是首先应该考虑是否可以**动态规划**或者**DFS**，看本题的**数据范围**，DFS可能会超时，所以我们优先选择动态规划。

其实大家也能感觉到，本题和上一个题目非常类似，但仍有一些区别，本题的目标是**从整个序列的所有元素中选出一些元素的最大和，而窗口大小m变成了一个选择过程中的限制条件**，而上一题**窗口大小m**则是**最终结果的限制条件**。

### 状态定义
`f(i)`表示从以`i`为右端点的序列中选择，符合题目条件 并且 序列中 所有元素 和最大 的方案 的元素和。

### 状态转移
与其他动态规划问题相同，我们根据最后一个没有选择的元素对集合进行划分。根据题目范围我们知道每个元素都是正整数，也就是说最佳的方案中**如果选择了超过m个元素，那么我们不选其中的一个，使选择的元素断开，一定不会使结果变坏**。所以状态转移中，我们就枚举最后一个“**不选的元素 j**”。

如此，**最后一个 “不选择的元素 j”** 将整个序列分成了两部分：
1. 第一部分是**在以 `j-1` 为右端点的元素中做选择**，我们神奇的发现这一部分其实就是状态的定义f(j-1)。
2. 第二部分是`j~i`这一段的区间和，而对于区间和，我们可以使用**前缀和**快速求出。

![](/assets/1a79fd18-330a-4285-b185-08ee23ecadc6.png)


最后，我们把两部分相加就得到了状态转移方程：`f[i]=max(f[j-1] + s[i] - s[j])`。

我们把公式整理一下，因为前缀和s[i]是已知的，所以可以提出来，那么公式变成：`f[i]=s[i]+max(f[j-1] - s[j])`。

而我们想让`f[j-1]-s[j]`是最大值，**只需要让`s[j]`是最小值即可**。

要维护`s[j]`为最小值，注意j的取值范围`0 <= i-j <= m`，也就是说这是`j~i`是一个最大为m的窗口，所以我们可以如上题一般**使用单调队列来维护这个大小为m的窗口中的最小值**。

## Code
```cpp
#include <iostream>

using namespace std;

const int N = 100010;
// 单个元素的值为10^9，求前缀和会爆int，所以用long long来存。
typedef long long LL ; 
int n, m;
int q[N];
int hh, tt;
LL s[N], f[N]; // s数组存储前缀和

// g函数快速求出分析中的公式
LL g(int i) {
    return f[i - 1] - s[i];
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ) {
        scanf("%lld", &s[i]);
        s[i] += s[i - 1]; // 求前缀和
    }
    hh = 0, tt = 0; // 此处 单调队列的队头队尾与上题相同。
    // 枚举前缀和数组 滑动窗口模板 
    // 队头存储的是使得当前窗口中的g(q[hh])最大的元素下标
    for (int i = 1; i <= n; i ++ ) {
        if (q[hh] < i - m) hh ++ ;
        // 分析中的公式
        f[i] = max(f[i - 1], g(q[hh]) + s[i]);
        // 维护这个单调递增序列
        while (hh <= tt && g(q[tt]) <= g(i)) tt --;
        q[++ tt] = i;
    }
    cout << f[n] << endl;
    return 0;
}
```

# [AcWing 1089. 烽火传递](https://www.acwing.com/problem/content/1091/)
> 烽火台是重要的军事防御设施，一般建在交通要道或险要处。
> 
> 一旦有军情发生，则白天用浓烟，晚上有火光传递军情。
> 
> 在某两个城市之间有 n 座烽火台，每个烽火台发出信号都有一定的代价。
> 
> 为了使情报准确传递，在连续 m 个烽火台中至少要有一个发出信号。
> 
> 现在输入 n,m 和每个烽火台的代价，请计算在两城市之间准确传递情报所需花费的总代价最少为多少。
> 
> **输入格式**  
> 第一行是两个整数 n,m，具体含义见题目描述；
> 
> 第二行 n 个整数表示每个烽火台的代价 ai。
> 
> **输出格式**  
> 输出仅一个整数，表示最小代价。
> 
> **数据范围**  
> 1≤m≤n≤2×105,  
> 0≤ai≤1000
> 
> **输入样例**  
> 5 3  
> 1 2 5 6 2  
> 
> **输出样例**  
> 4

## 题目描述
给出一个长度为n的序列，选出一些元素，使得最后这些被选择元素之和最小，选择的要求是连续m个元素中至少要选择一个。

本题和上一题整好相反，上一题是**所有被选择元素和最大，不能连续选择m个**，而本题是**所有被选择元素和最小，连续m个元素中至少选择一个**。

## 题目分析
因为和上题类似，所以我们可以使用相同的动态规划思路。

### 状态定义
`f(i)`表示从以`i`为结尾的序列中选择，符合条件（`m`个元素中必选一个）并且选择元素`i`的 方案中，最优方案的 元素和。

### 状态转移
与上题类似，本题的状态转移中，我们枚举 i 之前 最后被选择的元素j，以此 将整个序列分为两部分：
1. 一部分是`1~j`，这部分在 以`j`为结尾的序列中考虑，并且选择`j`的最优方案，我们神奇的发现这部分正好可以套在状态定义中，也就是`f(j)`。
2. 另一部分是`j~i`，这段长度最长为`m`的序列中选择出的`i`的权重。

最后，我们把两部分加起来就是`f[i]=min(f[j] + w[i])`。

在这个公式里，`i`的权重`w[i]`是已经固定的，那么想求到最小值就**需要让`f[j]`越小越好。**

而我们知道题目要求m个元素必须选择一个，所以i和j之间的距离就是`1~m`，即`1 <= i - j <= m`这样。

也就是我们要快速找到的`f[j]`是**最大长度为m的窗口内的最小值**， 所以就可以**使用单调队列，在最大为m的窗口中维护一个最小值**。

## Code
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 200010;
int n, m;
int f[N], w[N], q[N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ) {
        cin >> w[i];
    }
    int hh = 0, tt = 0;
    f[0] = 0;
    for (int i = 1; i <= n; i ++ ) {
        if (hh <= tt and q[hh] < i - m) hh ++ ;
        // 分析中的公式 队头存储的是最小的f[j]
        f[i] = f[q[hh]] + w[i]; 
        // 维护 递减队列单调性
        while (hh <= tt and f[q[tt]] >= f[i]) tt -- ;
        q[++ tt] = i;
    }
    int res = 1e9;
    for (int i = n - m + 1; i <= n; i ++ ) {
        res = min(res, f[i]);
    }
    cout << res << endl;
    return 0;
}
```

# [AcWing 1090. 绿色通道](https://www.acwing.com/problem/content/1092/)

> 高二数学《绿色通道》总共有 n 道题目要抄，编号 1,2,…,n，抄第 i 题要花 ai 分钟。
> 
> 小 Y 决定只用不超过 t 分钟抄这个，因此必然有空着的题。
> 
> 每道题要么不写，要么抄完，不能写一半。
> 
> 下标连续的一些空题称为一个空题段，它的长度就是所包含的题目数。
> 
> 这样应付自然会引起马老师的愤怒，最长的空题段越长，马老师越生气。
> 
> 现在，小 Y 想知道他在这 t 分钟内写哪些题，才能够尽量减轻马老师的怒火。
> 
> 由于小 Y 很聪明，你只要告诉他最长的空题段至少有多长就可以了，不需输出方案。
> 
> **输入格式**  
> 第一行为两个整数 n,t。
> 
> 第二行为 n 个整数，依次为 a1,a2,…,an。
> 
> **输出格式**  
> 输出一个整数，表示最长的空题段至少有多长。
> 
> **数据范围**  
> 0<n≤5×10^4,  
> 0<ai≤3000,  
> 0<t≤10^8  
>
> **输入样例**  
> 17 11  
> 6 4 5 2 5 3 4 5 2 3 4 5 2 3 6 3 5
> 
> **输出样例**  
> 3

## 题目描述
（为什么小Y同学要抄作业呢？他自己写不行吗？大家可不要学习他。）

给出长度为`n`的序列，序列中每个元素都有一个权重`w[i]`，再给出一个正整数`m`，求出一个方案**使得所选元素的权重总和不超过m，并且 所选元素 的 间距 越小越好**，目标是输出**选中方案的最大间距和**。

## 题目分析
本题初步看上去可能没有什么头绪，但是细品一下，我们可以注意到题目中的重要的关键信息：所求目标为**所有方案中 间距和 最短的 那个方案中最长的 间距和 是多少**。

这种**最长的。。。中找最短的。。。**或者**最大的。。。中找最小的**或者**最慢的。。。中找最快的。。。**等关键字，正是提示我们题目**可能 具有二段性**，因此我们应该优先考虑是否可以进行**二分**。

而在本题中，我们可以在题目中发现一个明显的信息：**题目的最终答案，也就是 最小的间距 一定可以使 所选的元素权重和 满足m**

在此基础上，**如果我们把最终答案增大，也就是空题变多，那么所选元素总和就会变得小于m；而我们把这个答案变小，也就是选择元素多了，显然最后会大于m**。

而小于m说明还有时间能抄更多的作业，大于m显然是不满足题目要求的，所以只有等于m才是正好的。

因此，我们可以以**空题段为0**作为二分的下限，而**空题段为所有元素**作为二分的上限，假设二分后的结果（最大间距）是`x`。

这样，本题就变形成了**在序列中选择一些元素，最长的选择间隔是x（换句话说x+1个间隔中必选一个元素），使选择的 元素和 最小**。这样一来，本题变得和上一道题一模一样。

而我们在求出最小的 元素和 之后，只需要在与m进行比较，看看这个 最小的元素和（也就是花费的时间）是否达到了`m`，如果没达到`m`，说明最大间距还可以减小；如果超过了说明需要增加最大间距。

## Code
```cpp
#include <iostream>
#include <cstring>

using namespace std;
const int N = 50010;
int n, m;
int f[N], w[N], q[N];

// 单调队列优化DP 同上题
bool check(int x) {
    int hh = 0, tt = 0;
    for (int i = 1; i <= n; i ++ ) {
        if (hh <= tt and q[hh] < i - x) hh ++ ;
        f[i] = f[q[hh]] + w[i];
        while (hh <= tt and f[q[tt]] >= f[i]) tt --;
        q[++ tt] = i;
    }
    int res = 1e9;
    // 只需要找最后一个间隔就可以啦。
    for (int i = n - x + 1; i <= n; i ++ ) res = min(res, f[i]);
    return res <= m;
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ) cin >> w[i];
    // 二分
    int l = 0, r = n;
    while (l < r) {
        int mid = l + r >> 1;
        if (check(mid + 1)) r = mid;
        else l = mid + 1;
    }
    cout << l << endl;
    return 0;
}
```

# 模板
到此为止，我们已经写了四道类似的题目，它们的共同特点是都使用了**单调队列**，因此我想可以总结出一个模板，来应对类似题目。

> 该类题目，一般给出一个长度为`n`的序列，给出一个窗口大小`m`，根据窗口大小`m`求方案。同时，伴随着**连续**、**子序列**等关键词。

## Code
```cpp 伪代码
int f[N], q[N], w[N];

// 注意这里默认了队列中已经有了第一个元素，即q[0] = 0;
int hh = 0, tt = 0; 
// 枚举序列 要注意是否需要先做前缀和 因为单调队列一般会求区间
// 而如果需要求区间和 多半需要提前计算前缀和 
for (int i = 1; i <= n; i ++ ) {
    // 确保窗口范围在给定的大小m内 
    // 比较符号 根据枚举顺序确定 一般正向枚举就是小于号
    if (hh <= tt and i - m < q[hh]) hh ++ ;
    // 状态转移 其中极值在队列头 也就是 q[hh] 
    ...
    // 确保队列单调性
    while (hh <= tt and {队尾q[tt] 与 当前元素i 比较}) tt --;
    q[ ++ tt ] = i // 把当前元素放到队列，注意这里放的是下标
}
// 根据状态的定义 判断最后的目标值。
// 如果是对整个序列求和之类的整体操作，那么多半可能是f[n]。
// 如果是选择最佳方案之类的 则可能需要再在整个序列上找一下答案。
```

## 基本参数
**f 数组**: 表示状态定义。在状态定义中，一般是**以当前元素 `i` 为结尾的序列，同时满足题目条件，并且选择当前元素 i**，

**q 数组**： 用来配合`hh`和`tt`指针来模拟单调队列，值得注意的是：在模板中我们默认了队列拥有第一个元素`0`，并且在枚举的时候，也是从`1`开始枚举的（而不是`0`），因此，我们读入数据的时候最好也从下标`1`开始读入。

**w 数组**：一般会存储题目给出的长度为`n`的序列，这里值得注意的是：单调队列的作用是**在某个区间内维护极值**，而关于区间的操作，我们就有可能会涉及到**求区间和**，而在`O(1)`的时间内求区间和，离不开**前缀和**的加持，因此，我们必须**判断 w数组 是否需要变成 前缀和 数组**。

## 状态转移
在介绍`f 数组`部分，我们已经介绍过**状态的定义**，而之所以要如此定义，是因为在**状态转移**的时候，可以方便的根据上一次的状态把序列分为两部分，一部分是**已知状态**，即，这部分可以**直接通过状态定义求出来**，我们只需要专注在另一部分的逻辑即可。

而在另一部分的逻辑中，我们通过状态的含义，**来找到需要保持单调的部分**，具体可以参考一下上面的第二个和第三个题目，都有标记出需要进行单调的部分。

# [LeetCode 1425. 带限制的子序列和](https://leetcode.cn/submissions/detail/340062842/)
> 给你一个整数数组 `nums` 和一个整数 `k` ，请你返回 非空 子序列元素和的最大值，子序列需要满足：子序列中每两个 相邻 的整数 `nums[i]` 和 `nums[j]` ，它们在原数组中的下标 `i` 和 `j` 满足 `i < j` 且 `j - i <= k` 。
> 
> 数组的子序列定义为：将数组中的若干个数字删除（可以删除 0 个数字），剩下的数字按照原本的顺序排布。
> 
>  
> 
> 示例 1：
> 
> 输入：nums = [10,2,-10,5,20], k = 2  
> 输出：37  
> 解释：子序列为 [10, 2, 5, 20] 。  
>
> 示例 2：
> 
> 输入：nums = [-1,-2,-3], k = 1  
> 输出：-1  
> 解释：子序列必须是非空的，所以我们选择最大的数字。
> 
> 示例 3：  
> 
> 输入：nums = [10,-2,-10,-5,20], k = 2  
> 输出：23  
> 解释：子序列为 [10, -2, -5, 20] 。  
>  
> 
> 提示:
> 
> 1 <= k <= nums.length <= 10^5  
> -10^4 <= nums[i] <= 10^4

## 题目描述
给出一个整数数组`nums`，和一个整数`k`，目标是找到**非空 子序列和 的最大值**，

这里的子序列含义是**保证顺序即可，但是不要求连续**。

在此基础上，还有一个规则：那就是**相邻两个元素的 下标之差 必须 小于等于 k**。

把这个规则换一句话说就是：**在k个间隔之内必须有一个元素被选择**。如此我们发现本题与前面的题目（烽火传递）竟然非常相似，只不过一个求最小值，本题求的是最大值。

还有一个很重要的细节，也需要注意到：**本题给出的序列中的元素可能是负数**，这也就是说，虽然本题求的是最大值，但是并不是选择的元素越多越好的。

## 套模板
我们使用上面总结出来的模板来解决这道题。过程中，我们需要关注这样几个关键点：

1. 首先是判断窗口大小的部分，我们因为是从前往后枚举元素，所以不用动，直接就使用小于号即可。

2. 在状态转移的部分，只要使用窗口中的极值（最大值）加上当前元素的权重，也就得到了当前状态。
    
    这一转移过程与**烽火传递**这道题完全的相同，因此不在展开细节了。但是有一个很关键的点需要注意：**因为本题存在负数的原因，很可能出现 状态转移方程的前半部(也就是上一状态)分是负数，但是当前元素是正数，而当前元素与上一状态相加得到的 当前状态 仍然是负数 的这种情况**。
    
    此时就会错误，因为我们**最后的目标 子序列 是可以任意选择开始的，此时如果我们从当前的正数部分选择无疑是更优的，因为可以使当前状态为正数。**
    
    所以，**我们要判断下上一状态如果是负数，就让他变成0，以此来让当前元素 作为 当前状态**。
    
3. 在维护队列单调性的部分，因为我们需要维护的是队头元素最大，所以当出现**当前元素 比 队尾元素 更大时，说明队尾元素没用了，把它出队即可**。

4. 因为题目中存在负数，所以并不能保证`f[n]`一定是最后的答案，因此，我们遍历整个`f数组`找到最大值。

## Code
```cpp
class Solution {
public:
    const static int N = 100010;
    int f[N], w[N], q[N];
    int constrainedSubsetSum(vector<int>& nums, int k) {
        int n = nums.size();
        // 注意，为了完全的套上模板，我在这里把数组下标移动到从1开始。
        for (int i = 1; i <= n; i ++ ) w[i] = nums[i - 1];
        int hh = 0, tt = 0;
        for (int i = 1; i <= n; i ++ ) {
            // 关键点1
            while (hh <= tt and q[hh] < i - k) hh ++ ;  
            // 关键点2
            f[i] = max(f[q[hh]], 0) + w[i];
            // 关键点3
            while (hh <= tt and f[q[tt]] < f[i]) tt -- ;
            q[ ++ tt] = i;
        }
        // 关键点4
        int res = -1e9;
        for (int i = 1; i <= n; i++ ) res = max(res, f[i]);
        return res;
    }
};
```

# [LeetCode 1696. 跳跃游戏 VI](https://leetcode.cn/problems/jump-game-vi/)

> 给你一个下标从 `0` 开始的整数数组 `nums` 和一个整数 `k` 。
> 
> 一开始你在下标 `0` 处。每一步，你最多可以往前跳 `k` 步，但你不能跳出数组的边界。也就是说，你可以从下标 `i` 跳到 `[i + 1， min(n - 1, i + k)]` 包含 两个端点的任意位置。
> 
> 你的目标是到达数组最后一个位置（下标为 `n - 1` ），你的 得分 为经过的所有数字之和。
> 
> 请你返回你能得到的 最大得分 。
> 
>  
> 
> **示例 1**
> 
> 输入：nums = [1,-1,-2,4,-7,3], k = 2  
> 输出：7  
> 解释：你可以选择子序列 [1,-1,4,3] （上面加粗的数字），和为 7 。  
> **示例 2**
> 
> 输入：nums = [10,-5,-2,4,0,3], k = 3  
> 输出：17  
> 解释：你可以选择子序列 [10,4,3] （上面加粗数字），和为 17 。  
> 
> **示例 3**
> 
> 输入：nums = [1,-5,-20,4,-1,3,-6,-3], k = 2  
> 输出：0
>  
> 
> **提示**
> 
>  1 <= nums.length, k <= 105  
> -104 <= nums[i] <= 104

## 题目描述
给出一个序列`nums`和一个正数`k`，从下标 `0` 开始，向后遍历并选择元素并且尽量让所选元素和最大，注意元素并非全是正数，因此并不是选择越多越好。除此之外，还有选择的规则：每次只能在当前元素的后k个元素中做选择。换句话说就是：**k个间隔之内必须有一个元素被选择**。

## 套模板
从题目描述中我们可以发现，本题其实**和上一道题几乎一模一样**，因此，本题权当留给读者当作业（~~绝对不是我写的太累了~~），自己去试着从分析到代码，完整的做一遍。

仍然要提示的是，本题虽然与上题很相似，但是还有一个小的关键点需要注意。即，在上一个题目中，序列是可以随意进行选择的，但是本题**必须从第一个元素就开始选择，且第一个元素是必选的**（可以从示例看出来）。

因此，我们的应对之法就是**首先把f[0]初始化为第一个元素的值，那么剩下的序列就从1开始了，再参考上一题就可以了**。

## Code
```cpp
class Solution {
public:
    const static int N = 100010;
    int f[N], q[N];
    int maxResult(vector<int>& nums, int k) {
        int n = nums.size();
        int hh = 0, tt = 0;
        // 第一个元素默认必须选择 因此后面的序列下标直接从1开始
        // 所以我们没有像上题一样把nums的下标改成1~n。
        f[0] = nums[0]; 
        // 单调队列优化 与上题完全相同 注意最后一个元素是n-1。
        for (int i = 1; i < n; i ++ ) {
            while (hh <= tt and q[hh] < i - k) hh ++ ;
            f[i] = f[q[hh]] + nums[i];
            while (hh <= tt and f[q[tt]] <= f[i]) tt -- ;
            q[ ++ tt] = i;
        }
        return f[n - 1];
    }
};
```

**END**
