---
title: LeetCode 691. 贴纸拼词
math: true
date: 2022-05-14 
comment: true
tags: 
- 状态压缩
- LeetCode
categories:
- 算法
- 动态规划
---
# LeetCode 691. 贴纸拼词

:::info  
**参考视频：https://www.acwing.com/video/2516/**
:::

> 我们有 n 种不同的贴纸。每个贴纸上都有一个小写的英文单词。
>
> 您想要拼写出给定的字符串 target ，方法是从收集的贴纸中切割单个字母并重新排列它们。如果你愿意，你可以多次使用每个贴纸，每个贴纸的数量是无限的。
>
> 返回你需要拼出 target  的最小贴纸数量。如果任务不可能，则返回 -1 。
> 
> 注意：在所有的测试用例中，所有的单词都是从 1000 个最常见的美国英语单词中随机选择的，并且 target  被选择为两个随机单词的连接。
>
>示例 1：  
>输入： stickers = ["with","example","science"], target = "thehat"  
>输出：3  
>解释：  
>我们可以使用 2 个 "with" 贴纸，和 1 个 "example" 贴纸。  
>把贴纸上的字母剪下来并重新排列后，就可以形成目标 “thehat“ 了。  
>此外，这是形成目标字符串所需的最小贴纸数量。
>
>示例 2:  
>输入：stickers = ["notice","possible"], target = "basicbasic"  
>输出：-1  
>解释：我们不能通过剪切给定贴纸的字母来形成目标“basicbasic”。
>
>提示:
>
>n == stickers.length  
>1 <= n <= 50  
>1 <= stickers[i].length <= 10  
>1 <= target <= 15  
>stickers[i]  和  target  由小写英文单词组成

## 题意及分析

### 题意

本题的题意是比较好理解的。

给出一些单词，以及一个目标单词，用**最少的**给出单词，拼出这个目标单词，规则是可以任意使用每个给出单词中的字母，并且每个给出单词也是可以无限使用的。（参考示例 1）

### 思路

大体上的思路肯定非常好想到，但是本题的难点在于实现。

看数据范围，target 最大才 15 个字母，stickers 最大才 50 个单词，因此在**数据范围的提示**下很容易想到状态压缩暴力搜索。

大体思路：**枚举 stickers 中每个单词，枚举单词中的每个字母，把字母填充到 target，每次都更新 得到当前状态 使用了多少单词，这样当我们 枚举 target 到最后状态的时候，我们也就知道 得到最后状态 使用了多少单词。**

具体地：

1. 将 target 的二进制位中的每一位是否为 1, 表示为是否填充，也就是说当 target 中的二进制位全是 1 时候，我们认为已经成功匹配了 target。**比如：当前 target 的长度是 4，那么，当 target 长度的二进制位都是 1，也就是 1111 的时候，我们认为 target 已经找到**，target 的二进制位初始化为 0。

2. 通过填充每个单词中的每个字母来更新状态。比如：示例 1 中的单词"with"中的字母"t"匹配到了 target 中的字母"t"，我们就认为 target 中的“t”的位置已经被填充了（即，那个位置的的二进制位变成 1）。

3. 状态定义，使用**f(state)表示当前状态 所需要的最小单词数量。**

4. 累了，提莫的，随便写写不应该写这么详细，也不知道该咋写了，直接看代码注释吧。

## Code

```cpp
const int N = 1 << 15;
int f[N], g[N][26];

class Solution {
public:
    int n;
    string target;
    vector<string> strs;
    // target最大15. 也就是最多使用15个单词，每个单词出一个字母。
    // 所以极限值定为20。
    const int INF = 20;

    // 将 字符c 填充到状态 state
    int fill(int state, char c) {
        // v 表示状态state中添加了一个字母c 之后的状态
        // 为什么需要g？ 因为可能 不同的单词中有相同的字母
        // 遇到这种情况，可以直接返回，也就是下一步的剪枝。
        // 比如：当前状态1101遇到了字母a，
        // 而在枚举下一个单词时候1101又遇到了a，
        // 如果此时已经更新(g[1101][a]不等于初始值-1) 则直接返回即可。
        auto& v = g[state][c - 'a'];
        if (v != -1) return v; // 剪枝。
        v = state;
        // 枚举 target的每一位。
        // （注意target的最大值也才15，所以虽然fill操作频繁 但这一步并不慢）
        for (int i = 0; i < n; i ++ ) {
            // state >> i & 1 表示把state的二进制位的第i位拿出来
            // 因此，这个if 表示如果状态state的第i位不是1（或者说是0）
            // 那么就判断，当前字符c是不是和它一样，如果一样就填进去。
            if (!(state >> i & 1) && target[i] == c) {
                // 把v的第i位变成1，v变成新的状态
                v += 1 << i;
                break; // 已经把c填充到state了 所以直接break就行
            }
        }
        return v;
    }
    // 爆搜
    int dfs(int state) {
        // v 表示当前状态下需要添加的单词数量
        auto& v = f[state];
        // 剪枝。如果v已经被计算过则直接返回
        if (v != -1) return v;
        // 如果状态为全1,说明搜索完毕，不需要添加任何单词了
        if (state == (1 << n) - 1) return v = 0;
        // 把v置为较大的数，如果最后v仍然是INF，说明v从没真正有效更新过
        v = INF;
        // 枚举每个贴纸，也就是每个单词
        for (auto& str: strs) {
            int cur = state; // cur 临时保存当前状态。
            // 用单词的每个字母去填充当前状态。
            for (auto c: str) cur = fill(cur, c);
            // 填充完 如果和 填充前 不一样，说明状态有更新
            // 并且因为 目标是 最少使用的贴纸数量，所以这里用min
            // cout << state << ' ';
            if (cur != state) {
                v = min(v, dfs(cur) + 1);
                // cout << v << ' ';
            }
        }
        return v;
    }

    int minStickers(vector<string>& stickers, string _target) {
        memset(f, -1, sizeof f);
        memset(g, -1, sizeof g);
        target = _target;
        strs = stickers;
        n = target.size();
        int res = dfs(0);
        // 如果所有的贴纸都不能满足目标，则返回-1；
        if (res == INF) res = -1;
        return res;
    }
};
```

## 总结

状态压缩问题与位运算强相关，所以位运算方面的东西至少要了解常用操作，本题才好理解，比如：抠出第 i 位数等。

关于本题还有两个问题想谈一下，是我卡了一会才想明白的：

1. 是如何确定 target 被完全搜索出来的呢？（或者说是什么时候返回-1 的呢？或者说是什么时候 res 能等于 INF 呢？）

   答：注意注释中有一段是“状态全为 1，表示搜索完毕”那块，在它的下面我们每次 dfs 都把 v 变成了 INF，也就是说，只有最后进行“不明显回溯”的时候，v 的值才会被真的更新。而什么时候触发“不明显回溯”呢？就是当状态全为 1 的时候，我们才把 v 变成 0 开始回溯。

   这也保证了，只有 target 全部被搜索出来，min 的比较才有意义，不然全部都是 min(INF, INF)。

   只有当状态全为 1，开始回溯了，才会变成`min(INF, 1),min(INF, 2)....`这种比较。

   关于“回溯”、“不明显回溯”如果不太明白，或者上这段解释不懂，我建议先看[这篇文章](https://mp.weixin.qq.com/s/agL4aE7-GtGaB8YFz_X8TA)，否则，即便我个人觉得上面这段解释已经比较清楚了，您还是可能看不懂。

2. 为什么在填充字符c到state的时候，也就是fill函数，当找到第一个符合条件的空位就填进去呢？不用考虑后面其他符合条件的吗？

    答：不用的，我们的爆搜dfs函数，每次都搜索了所有的单词，也就是当前单词，下一次搜索还会重复搜到，所以如果后面仍有符合条件的，还会被填充进去的，不会漏掉。

还想写啥来着，我忘记了，不磨叽了，就这样吧~~

**END**

