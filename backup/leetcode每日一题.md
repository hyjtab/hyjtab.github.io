# 每日一题（零一背包与bitset）
> Leetcode 3181. 执行操作可获得的最大总奖励 II
> 给你一个整数数组 rewardValues，长度为 n，代表奖励的值。
> 最初，你的总奖励 x 为 0，所有下标都是 未标记 的。你可以执行以下操作任意次 ：
> 从区间 [0, n - 1] 中选择一个 未标记 的下标 i。
> 如果 rewardValues[i] 大于 你当前的总奖励 x，则将 rewardValues[i] 加到 x 上（即 x = x + rewardValues[i]），并标记下标 i。
> 以整数形式返回执行最优操作能够获得的 最大总奖励。

今天是一道hard题，第一眼就感觉很像动态规划，但是没有坚持这个思路，去试了暴力和贪心，发现都不能做出来，于是去翻题解，官方题解使用了bitset优化空间和动态规划结合的做法。是一种基于类零一背包的优化做法。
## 动态规划
于是顺着链接，翻看代码随想录，捡了捡思路，首先是动态规划五部曲：
>1.确定dp数组（dp table）以及下标的含义
>2.确定递推公式
>3.dp数组如何初始化
>4.确定遍历顺序
>5.举例推导dp数组
零一背包一般是指在有限个具有自身价值和重量的物品中选取尽可能多价值的物品，同时不超过背包的总限重的问题。
参照编程随想录的思路，首先确定数组形状（维数）以及各个下标的含义，对于零一背包，这个应该是最好解决的问题，两个参数也就确定了两个维度。

随后是数组内容的含义，这一部分我觉得是需要更加细致的解释的。数组内容的含义就是每个子问题的答案，而子问题是什么也就成了重点，多维变量确定子问题，我们应该先考虑其中一维的问题，而其中比较直接的子问题是关于背包容量的子问题，即背包最大容量为i时，能够拥有的最大价值是多少，于是开始考虑另一维。由于前面的分析，我们在每一个单元保存的都是价值之和，价值这一维度就不能“你是你，我是我了”，因此我们将价值转为物品编号，对于编号j，是指从所有编号小于j的物品中选取最多价值的情况。随后也就可以确定我们的递推公式。

$dp[i][j] = max(dp[j-1][i],dp[j][i-w[j]]+v[j])$

然后是如何初始化，这就要考虑等式左右两边的递推时的下标变化情况。我们可以明显看出ij均为从小到大，因此只需要正确初始化每个index为0的行列即可。随后是遍历顺序，由于边界情况都被正确初始化，所以推导以任意一维为主均可正确运行。随后便是打印验证环节。

除此之外，数组下标和内容的含义还可以变化，令j为价值，而内容为能否实现这样的递归顺序。那么我们的递归公式将变成：
$dp[i][j] = dp[i-1][j-v[i]] || dp[i-1][j]$，
递归起点便只有00和j=0全为true，答案即为内容为true的最大的j。

随后需要注意的是，零一背包问题均可以进行降维(空间上)：01背包问题的第一种做法中，我们设置的二维数组可以用一维表示，因为我们只需要考虑从所有编号小于n的物品中选取最多价值的情况即可,递归起点为dp[0]=0。
$dp[i]=max(dp[i-1],dp[i-w[j]]+v[i])$

## bitset优化压缩
这题的思路应该倾向于第二种，但是不做任何改变依旧会超时。答案做的优化是，将数组压缩成为1维数组，随后转为bitset。由于题目特性，只有v大于当前价值和j时才能够纳入背包，因此当前价值和满足v<= j <2v，因此，对于每一个v，都需要在j∈[v , 2v)区间内做这样的操作：dp[j] = dp [j-v] || dp[j]。再次压缩为bitmap，即f左移size-j再右移size-j再左移v，前两次抵消的目的是清除大于v位的比特，随后进行或运算，实现我们的动归算法。

最后答案从最大可能性2m-1开始搜寻，最大者即为答案。

答案来自[灵茶山艾府](https://leetcode.cn/problems/maximum-total-reward-using-operations-ii/solutions/2805413/bitset-you-hua-0-1-bei-bao-by-endlessche-m1xn/)这位大佬。

```cpp
class Solution {
public:
    int maxTotalReward(vector<int>& rewardValues) {
        ranges::sort(rewardValues);
        rewardValues.erase(unique(rewardValues.begin(), rewardValues.end()), rewardValues.end());
        bitset<100000> f{1};
        for (int v : rewardValues) {
            int shift = f.size() - v;
            // 左移 shift 再右移 shift，把所有 >= v 的比特位置 0
            // f |= f << shift >> shift << v;
            f |= f << shift >> (shift - v); // 简化上式
        }
        for (int i = rewardValues.back() * 2 - 1; ; i--) {
            if (f.test(i)) {
                return i;
            }
        }
    }
};
```
bitset语法部分：
初始化
![leetcodenotes1](https://github.com/user-attachments/assets/cd9ab729-1af0-4d09-bde8-598dc3b5025b)
size
![leetcodenotes](https://github.com/user-attachments/assets/2c1c2528-fb9c-4245-a289-d71fdd647cf5)
test
![leetcodenotes2](https://github.com/user-attachments/assets/6545b956-40e7-4e69-b881-e59e37423931)