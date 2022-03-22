[TOC]

## leetcode 周赛

### 第 274 场

#### Problem D - 相同元素间隔之和

题目链接：[相同元素的间隔之和](https://leetcode-cn.com/problems/intervals-between-identical-elements/)

参考题解：[前缀和+哈希](https://leetcode-cn.com/problems/intervals-between-identical-elements/solution/qian-zhui-he-ha-xi-by-gnomeshgh_plus-hsfd/)

总结：一开始写的思路和参考题解里的思路一样：

1、使用哈希表存放每个元素所对应的下标，键是每个出现的元素，值是这个元素出现的下标，使用数组进行存放。
2、从前往后遍历，找到每个值对应出现的所有下标，根据这些下标求距离。

这样子每次都会有非常多的重复计算，导致超时。后面也是知道了有前缀和这种算法。

```cpp
class Solution {
public:
    vector<long long> getDistances(vector<int>& arr) {
        int n = arr.size();
        vector<long long> pre(n, 0);
        vector<long long> post(n, 0);
        // pair.first 表示该元素的前一个元素的下标。second 表示该元素当前一共有多少个。
        unordered_map<int, pair<int, int>> hash; 

        // 求前缀和。
        // 注意，pre[i] 表示 arr[i] 之前所有等于 arr[i] 的元素到 arr[i] 的距离。post 同理
        for (int i = 0; i < n; i++)
        {
            int temp = arr[i];
            if (hash.find(temp) != hash.end())
            {
                int j = hash[temp].first;
                int cnt = hash[temp].second;
                pre[i] = pre[j] + abs(i - j) * cnt;
            }
            hash[temp].first = i;
            hash[temp].second++;
        }

        hash.clear();
        for (int i = n - 1; i >= 0; i--)
        {
            int temp = arr[i];
            if (hash.find(temp) != hash.end())
            {
                int j = hash[temp].first;
                int cnt = hash[temp].second;
                post[i] = post[j] + abs(i - j) * cnt; 
            }
            hash[temp].first = i;
            hash[temp].second++;
        }

        vector<long long> ans(n, 0);
        for (int i = 0; i < n; i++)
        {
            ans[i] = pre[i] + post[i];
        }
        return ans;
    }
};
```

### 第 275 场

#### Problem B - 最少交换次数来组合所有的 1 II

题目链接：[最少交换次数来组合所有的 1 II](https://leetcode-cn.com/problems/minimum-swaps-to-group-all-1s-together-ii/)

标签：滑动窗口

思路

1. 数出一共有多少个 1
2. 使用滑动窗口，以 1 的总数为窗口大小
3. 计算窗口内 0 的个数，最少的即为需要交换的次数

使用定长滑动窗口

```cpp
class Solution {
public:
    int minSwaps(vector<int>& nums) {
        int windMaxSize = 0;
        for (auto it : nums)
        {
            if (it == 1)
            {
                windMaxSize++;
            }
        }
        if (windMaxSize == 0)
        {
            return 0;
        }

        int ans = INT_MAX;
        int left = 0;
        int right = 0;
        int zeroCnt = 0;
        
        while (left < nums.size())
        {
            // 扩张
            while (1)
            {
                int n = nums[right % nums.size()];
                right++;
                if (n == 0)
                {
                    zeroCnt++;
                }
                
                // 满足窗口大小条件
                if (right - left == windMaxSize)
                {
                    ans = min(ans, zeroCnt);
                    break;
                }
            }

            // 收缩
            int n = nums[left];
            left++;
            if (n == 0)
            {
                zeroCnt--;
            }
        }

        return ans;
    }
};
```

####  Problem C - 统计追加字母可以获得的单词数

题目链接：[统计追加字母可以获得的单词数](https://leetcode-cn.com/problems/count-words-obtained-after-adding-a-letter/)

- **标签：哈希表**

超时。时间复杂度为O(n^2)

```cpp
class Solution {
public:
    int wordCount(vector<string>& startWords, vector<string>& targetWords) {
        int ans = 0;
        for (int i = 0; i < targetWords.size(); i++)
        {
            string targetStr = targetWords[i];
            for (int j = 0; j < startWords.size(); j++)
            {
                string startStr = startWords[j];
                if (targetStr.size() <= startStr.size())
                {
                    continue;
                }
                
                if (targetStr.size() - startStr.size() != 1)
                {
                    continue;
                }
                
                if (check(startStr, targetStr))
                {
                    ans++;
                    break;
                }
            }
        }
        
        return ans;
    }
    
    bool check(string str1, string str2)
    {
        int hash[256] = {0};
        for (int i = 0; i < str2.size(); i++)
        {
            hash[str2[i]]++;
        }
        
        for (int i = 0; i < str1.size(); i++)
        {
            hash[str1[i]]--;
            if (hash[str1[i]] < 0)
            {
                return false;
            }
        }
        
        return true;
    }
};
```

总结：串匹配，一般选择 hash 表保存字符串，然后再循环遍历一次即可。注意不要用 set，要用 unordered_set。set 底层是红黑树，find 效率没有 hash 表高。

```cpp
class Solution {
public:
    int wordCount(vector<string>& startWords, vector<string>& targetWords) {
        unordered_set<string> patterns;
        // unordered_map<string, int> patterns; // 也可以用 map 来存储
        for (int i = 0; i < startWords.size(); i++)
        {
            vector<int> vis(26, 0);
            string str = startWords[i];
            for (int j = 0; j < str.size(); j++)
            {
                vis[str[j] - 'a'] = 1;
            }

            for (char k = 'a'; k <= 'z'; k++)
            {
                if (vis[k - 'a'])
                {
                    continue;
                }

                string temp = str + k;
                sort(temp.begin(), temp.end());
                // patterns[temp] = 1;
                patterns.insert(temp);
            }
        }

        int ans = 0;
        for (int i = 0; i < targetWords.size(); i++)
        {
            string temp = targetWords[i];
            sort(temp.begin(), temp.end());
            if (patterns.find(temp) != patterns.end())
            {
                ans++;
            }
            // if (patterns[temp])
            // {
            //     ans++;
            // }
        }

        return ans;
    }
};
```

#### Problem D - 全部开花的最早一天

题目链接：[全部开花的最早一天](https://leetcode-cn.com/problems/earliest-possible-day-of-full-bloom/)

- **标签：贪心**

很显然，不论以任何顺序种下，播种花的总时间总为 sum{plantTime}

因此短板取决于生长花费的时间，让长的慢的尽早开始生长。

尽量让生长慢的先种，生长快的后种即可。

- **证明**

p<sub>1</sub> 和 p<sub>2</sub> ，生长所需天数为 g<sub>1</sub>，g<sub>2</sub> 

设  g<sub>1</sub> >= g<sub>2</sub> 

先 1 后 2 时的最晚开花时间:
$$
time_1 = max(p_1 + g_1, p_1 + p_2 + g_2)
$$
先 2 后 1 时的最晚开花时间:
$$
time_2 = max(p_2 + g_2, p_1 + p_2 + g_1)
$$

因为: g<sub>1</sub> >=  g<sub>2</sub>

所以: time<sub>2</sub> =  p<sub>1</sub>  +  p<sub>2</sub>  +  g<sub>1</sub> 

所以: p<sub>1</sub> + p<sub>2</sub> +  g<sub>1</sub>  >= p<sub>1</sub> + p<sub>2</sub> +  g<sub>2</sub> 

又因为: p<sub>1</sub> +  p<sub>2</sub> +  g<sub>1</sub> >  p<sub>1</sub> +  g<sub>1</sub>

所以: time<sub>2</sub> > time<sub>2</sub>

所以: time<sub>1</sub> 是我们要求的值。


```cpp
bool cmp(const pair<int, int>& a, const pair<int, int>& b)
{
    return a.second > b.second;
}

class Solution {
public:
    int earliestFullBloom(vector<int>& plantTime, vector<int>& growTime) {
        int n = plantTime.size();
        vector<pair<int, int>> arr(n);
        for (int i = 0; i < n; i++)
        {
            arr[i].first = plantTime[i];
            arr[i].second = growTime[i];
        }

        // grow time 从大到小排序
        sort(arr.begin(), arr.end(), cmp);

        int ans = INT_MIN;
        int sum = 0;
        for (int i = 0; i < n; i++)
        {
            sum += arr[i].first;
            ans = max(ans, sum + arr[i].second);
        }

        return ans;
    }
};
```

### 第 276 场

排名（1832 / 5243）

#### Problem C - 解决智力问题

题目链接：[解决智力问题](https://leetcode-cn.com/problems/solving-questions-with-brainpower/)

一开始想到是动态规划，但是写不出状态转移方程。后面思考用回溯法去做，但是超时了。不知道回溯法能不能通过剪枝去优化。

- 回溯法

```c++
class Solution {
public:
    long long ans = 0;

    long long mostPoints(vector<vector<int>>& questions) {
        int n = questions.size();
        if (n <= 1)
        {
            return questions[0][0];
        }
        vector<int> vis(n, 0); // 0 为没访问过
        dfs(0, n - 1, questions, vis, 0);
        return ans;
    }

    void dfs(int start, int end, vector<vector<int>>& questions, vector<int> vis, long long res)
    {
        ans = max(ans, res);
        if (start > end)
        {
            return;
        }

        for (int i = start; i <= end; i++)
        {
            if (!vis[i])
            {
                vis[i] = 1;
                dfs(i + questions[i][1] + 1, end, questions, vis, res + questions[i][0]);
                vis[i] = 0;
            }
        }
    }
};
```

后面看了评论后突然想起居然还有备忘录这东西。

- 备忘录 + 递归

```c++
class Solution {
public:
    vector<long long> memo;

    long long mostPoints(vector<vector<int>>& questions) {
        int n = questions.size();
        memo.resize(n, 0);
        return dfs(questions, 0, n);
    }

     // dfs 函数作用是从第 i 个问题开始最高能得到的分数
    long long dfs(vector<vector<int>>& questions, int i, const int n)
    {
        if (i >= n)
        {
            return 0;
        }
        if (memo[i] > 0)
        {
            return memo[i];
        }

        long long doScore = questions[i][0] + dfs(questions, i + questions[i][1] + 1, n);
        long long notDoScore = dfs(questions, i + 1, n);
        long long ans = max(doScore, notDoScore);
        memo[i] = ans; // 备忘录记录从第 i 个问题开始最高能得到的分数

        return ans;
    }
};
```

看了题解后还有动态规划

- 动态规划

动态规划记住最重要的一点，大问题分解成子问题去解决，是一种从下往上的算法（递归则从上往下）。这里状态方程其实很好写出来。第 i 道题无非是做或者不做。但是，这里和我们往常的思维不一样。因为 dp[i] 的结果需要后面的计算才能得出，所以如果我们从前往后遍历计算就相当于从上往下递归了。虽然这中写法也行，题解也有人正向 dp 来解题，但理解起来没有反向 dp 容易。

```c++
class Solution {
public:
    long long mostPoints(vector<vector<int>>& questions) {
        int n = questions.size();
        vector<long long> dp(n, 0); // dp[i] 表示从第 i 道题开始能获得的最大分数
        dp[n - 1] = questions[n - 1][0]; // 这里边界注意初始化。
        for (int i = n - 2; i >= 0; i--)
        {
            int next = i + questions[i][1] + 1;
            // 第 i 道题，做或不做
            long long doQuest = questions[i][0] + (next >= n ? 0 : dp[next]);
            long long notDoQuest = dp[i + 1];
            dp[i] = max(doQuest, notDoQuest);
        }

        return dp[0];
    }
};
```

- 总结

这种其实很容易想到动态规划，状态方程其实后面发现也很容易写出来。但是之前做的都是正向 dp，这种反向 dp 还是第一次见。还有就是拆成子问题，从下往上这种思维还是有所欠缺。

#### Problem D - 同时运行 N 台电脑的最长时间

状态：no pass

题目链接：[同时运行 N 台电脑的最长时间](https://leetcode-cn.com/problems/maximum-running-time-of-n-computers/)

假设 n 台电脑同时运行的最长分钟为 t。所以 n * t <= sum(总电池的容量)。所以 t <= sum / n。

所以大于平均值 t 的，那些电池给一个电脑从头用到尾，所以不考虑这个电脑也不考虑这个电池。问题的规模就缩小到了 n - 1。

继续求解 n - 1 的子问题。当最大的都不够平均值（子问题的平均值），那么答案就是剩下的所有电池混用能够维持的最大时间。

~~后续补充，今晚晚了，明天还要工作~~

### 第 277 场(待补充)

排名（1784 / 5059）

这周的周赛前三道相对于来说比较简单。第四题没有做出来。

#### Problem D -  基于陈述统计最多好人数

题目链接：[基于陈述统计最多好人数](https://leetcode-cn.com/problems/maximum-good-people-based-on-statements/)

完全没有头绪（后续补充）

### 第 278 场

排名（2290 / 4643）

- [x] `3 分` - [将找到的值乘以 2](https://leetcode-cn.com/problems/keep-multiplying-found-values-by-two/)
- [x] `4 分` - [分组得分最高的所有下标](https://leetcode-cn.com/problems/all-divisions-with-the-highest-score-of-a-binary-array/)
- [ ] `5 分` - [查找给定哈希值的子串](https://leetcode-cn.com/problems/find-substring-with-given-hash-value/)
- [ ] `6 分` - [字符串分组](https://leetcode-cn.com/problems/groups-of-strings/)

#### 分组得分最高的所有下标

方法一：前缀和

遍历过程中，维护前缀中 0 的个数和后缀中 1 的个数。

时间复杂度 O(N)。

空间复杂度 O(N)。

一开始就能想到用前缀和后缀和来做，但是还不是很熟练。写了好一会才做出来。看了别人写的题解后发现好简单，都不用额外开辟空间。

```cpp
class Solution {
public:
    vector<int> maxScoreIndices(vector<int>& nums) {
        int n = nums.size();
        vector<int> pre(n + 1, 0);
        vector<int> post(n + 1, 0);

        // 前缀和
        for (int i = 0; i < n; i++) {
            if (nums[i] == 0) {
                pre[i + 1] = pre[i] + 1;
            } else {
                pre[i + 1] = pre[i];
            }
        }

        // 后缀和
        for (int i = n; i > 0; i--) {
            if (nums[i - 1] == 1) {
                post[i - 1] = post[i] + 1;
            } else {
                post[i - 1] = post[i];
            }
        }

        map<int, vector<int>> m;
        int maxx = -1;
        for (int i = 0; i <= n; i++) {
            int score = pre[i] + post[i];
            maxx = max(score, maxx);
            m[score].push_back(i);
        }
        
        return m[maxx];
    }
};

```

#### 查找给定哈希值的子串

超时。想复杂了。一开始用滑动窗口，其实这里都不需要。

- 滑动窗口

```cpp
class Solution {
public:
    string subStrHash(string s, int power, int modulo, int k, int hashValue) {
        int right = 0;
        int left = 0;
        int n = s.size();
        string sub = "";
        while (right < n) {
            sub = s.substr(left, right - left + 1);
            right++;

            if (right - left == k) {
                if (hashValue == hash(sub, power, modulo)) {
                    return sub;
                }
            }

            while (right - left == k) {
                left++;
            }
        }

        return "";
    }

    int hash(string sub, int power, int modulo) {
        int k = sub.size();
        long long sum = 0;
        long long p = 1;
        for (int i = 0; i < k; i++) {
            int val = sub[i] - 'a' + 1;
            val %= modulo;
            p %= modulo;
            sum += val * p;
            sum %= modulo;
            p *= power;
        }
        return sum % modulo;
    }
};
```

- for 循环就完事了

只需要事先把 p 求出来即可。而不是每次都要重新计算 p，没有想到在 for 循环中顺便计算 p 也很耗时。（一直以为只有嵌套循环算法时间复杂度高了才会超时）

```c++
class Solution {
public:
    string subStrHash(string s, int power, int modulo, int k, int hashValue) {
        int n = s.size();
        vector<long long> pows(n, 1);
        for (int i = 1; i < n; i++) {
            pows[i] = pows[i-1] * power % modulo;
        }

        for (int i = 0; i <= n - k; i++) {
            string subStr = s.substr(i, k);
            if (hash(subStr, pows, modulo) == hashValue) {
                return subStr;
            }
        }

        return "";
    }

    int hash(string sub, const vector<long long>& pows, int modulo) {
        long long sum = 0;
        for (int i = 0; i <  sub.size(); i++) {
            sum += (sub[i] - 'a' + 1) * pows[i];
            sum %= modulo;
        }
        return sum % modulo;
    }
};
```

### 第 279 场

排名（1958 / 4132）

- [x] `3 分` - [对奇偶下标分别排序](https://leetcode-cn.com/problems/sort-even-and-odd-indices-independently/)
- [x] `4 分` - [重排数字的最小值](https://leetcode-cn.com/problems/smallest-value-of-the-rearranged-number/)
- [ ] `5 分` - [设计位集](https://leetcode-cn.com/problems/design-bitset/)
- [ ] `6 分` - [移除所有载有违禁货物车厢所需的最少时间](https://leetcode-cn.com/problems/minimum-time-to-remove-all-cars-containing-illegal-goods/)

#### 重排数字的最小值

方法一：贪心

- 零永远是零。
- 对于负数，将所有数字从大到小排列可以得到最小值。
- 对于正数，我们本应该将所有数字从小到大排列，但因为限制不允许有先导零，所以还要在排序后，将从左到右的第一个非零值移到最前面。

自己写的方法是把大数转换成数组后在排序。忽略了 string 也可以排序数字大小。导致了很多多余的操作（虽然代码思路都差不多）。

#### 设计位集

想到之前 csdn 发过的 bitmap 文章，而且回过头发现之前写的文章的代码还有错误的地方（后续修改）。

这道题想复杂了，想用二进制位运算去解，其实不需要。就一个单纯的 int 数组即可。需要注意的是 filp 操作，因为翻转一次时间复杂度是 `O(size)`，其中最多可能有 1e5 次，会超时。所以加入一个标志位，来表示当前的状态是否翻转了。具体需要注意的事项都写在注释里了。

```CPP
class Bitset {
private:
    vector<int> bucket;
    int cntOne;
    bool isFlip; // 翻转偶数次，相当于没翻转

public:
    Bitset(int size) {
        bucket.resize(size, 0);
        cntOne = 0;
        isFlip = false;
    }
    
    void fix(int idx) {
        if (!isFlip) {
            if (bucket[idx] == 0) {
                bucket[idx] = 1;
                cntOne++;
            }
        } else {
            if (bucket[idx] == 1) {
                bucket[idx] = 0;
                cntOne++;
            }
        }
    }
    
    void unfix(int idx) {
        if (!isFlip) {
            if (bucket[idx] == 1) {
                bucket[idx] = 0;
                cntOne--;
            }
        } else {
            if (bucket[idx] == 0) {
                bucket[idx] = 1;
                cntOne--;
            }
        }
    }
    
    void flip() {
        isFlip = !isFlip;
        cntOne = bucket.size() - cntOne; // 注意翻转后需要修改 cntOne
    }
    
    bool all() {
        return cntOne == bucket.size();
    }
    
    bool one() {
        return cntOne > 0;
    }
    
    int count() {
        return cntOne;
    }
    
    string toString() {
        string res = "";
        for (int i = 0; i < bucket.size(); i++) {
            int temp;
            if (isFlip) {
                if (bucket[i] == 0) { // 注意这里用一个中间变量去接，而不能在原数组操作
                    temp = 1;
                } else {
                    temp = 0;
                }
            } else {
                temp = bucket[i];
            }

            res += to_string(temp);
        }

        return res;
    }
};
```

### 第 280 场

排名（1488 / 5833）

- [x] `3 分` - [得到 0 的操作数](https://leetcode-cn.com/problems/count-operations-to-obtain-zero/)
- [x] `4 分` - [使数组变成交替数组的最少操作数](https://leetcode-cn.com/problems/minimum-operations-to-make-the-array-alternating/)
- [x] `5 分` - [拿出最少数目的魔法豆](https://leetcode-cn.com/problems/removing-minimum-number-of-magic-beans/)
- [ ] `6 分` - [数组的最大与和](https://leetcode-cn.com/problems/maximum-and-sum-of-array/)

### 第 282 场

排名（3998 / 7164）

- [x] `3 分` - [统计包含给定前缀的字符串](https://leetcode-cn.com/problems/counting-words-with-a-given-prefix/)
- [x] `4 分` - [使两字符串互为字母异位词的最少步骤数](https://leetcode-cn.com/problems/minimum-number-of-steps-to-make-two-strings-anagram-ii/)
- [ ] `5 分` - [完成旅途的最少时间](https://leetcode-cn.com/problems/minimum-time-to-complete-trips/)
- [ ] `6 分` - [完成比赛的最少时间](https://leetcode-cn.com/problems/minimum-time-to-finish-the-race/)

#### 完成旅途的最少时间

开始想到的是暴力，很显然，直接超时了（我们要思考怎么**去减少时间复杂度**，而不是对某些特殊情况做个 if 处理就完事了，这里做题的时候就陷入了这个误区，下次注意）。

暴力不过后，后面想着怎么保存之前计算的状态，没调试出来。

**参考答案**

方法一：二分答案
经典二分答案题：已知总时间 T，我们可以很容易地求出所有车一共运行了多少次。将这一次数表示为一个 T 的函数 f(T)，显然 f(T) 随 T 的增大单调递增。这一单调性是二分答案的基础。

时间复杂度 O(Nlog C)。其中 C = min(time) * totalTrips 为答案的上界。
空间复杂度 O(1)。

```c++
class Solution {
public:
    long long minimumTime(vector<int>& time, int totalTrips) {
        long long start = 1;
        int minn = getMin(time);
        long long end = static_cast<long long>(minn) * static_cast<long long>(totalTrips);

        while (start + 1 < end) {
            long long mid = start + (end - start) / 2;
            if (check(time, mid, totalTrips)) {
                end = mid;
            } else {
                start = mid;
            }
        }

        if (check(time, start, totalTrips)) {
            return start;
        }
        return end;
    }

    long long getMin(const vector<int>& time) {
        int minn = INT_MAX;
        for (auto t : time) {
            minn = min(minn, t);
        }
        return minn;
    }

    bool check(const vector<int>& time, long long t, long long totalTrips) {
        long long sum = 0;
        for (int i = 0; i < time.size(); i++) {
            sum += t / time[i];
        }
        return sum >= totalTrips;
    }
};
```

### 第 283 场

排名（4918 / 7817）

- [x] `3 分` - [Excel 表中某个范围内的单元格](https://leetcode-cn.com/problems/cells-in-a-range-on-an-excel-sheet/)
- [ ] `4 分` - [向数组中追加 K 个整数](https://leetcode-cn.com/problems/append-k-integers-with-minimal-sum/)
- [ ] `4 分` - [根据描述创建二叉树](https://leetcode-cn.com/problems/create-binary-tree-from-descriptions/)
- [ ] `6 分` - [替换数组中的非互质数](https://leetcode-cn.com/problems/replace-non-coprime-numbers-in-array/)

卡在第二题了，第三题其实很容易，但是没有做。所以比赛时候，被卡住了可以看看后面的题目。

#### 向数组中追加 K 个整数

第二题怎么说呢，看了答案后很简单，之前在做题的，没有想到等差数列求和。就暴力加。而且没加明白，导致超时。

- 暴力超时

```cpp
class Solution {
public:
    long long minimalKSum(vector<int>& nums, int k) {
        unordered_set<int> bucket;
        for (auto it : nums) {
            bucket.insert(it);
        }

        long long ans = 0;

        int start = 1;
        int end =  1000000000;
        for (int i = start; i <= end; i++) {
            if (k == 0) {
                break;
            }
            if (bucket.find(i) == bucket.end()) { // no found
                ans += i;
                k--;
            }
        }
        
        return ans;
    }
};
```

- 贪心

等差数列求和

```cpp
class Solution {
public:
    long long minimalKSum(vector<int>& nums, int k) {
        nums.push_back(0);
        nums.push_back(2e9);
        sort(nums.begin(), nums.end()); // 加入哨兵后排序

        long long ans = 0;
        for (int i = 1; i < nums.size(); i++) {
            long long len = nums[i] - nums[i-1] - 1; // 可以填充的数字个数
            if (len <= 0) {
                continue;
            }

            if (len >= k) {
                 // 等差数列求和。这里需要注意乘积是 k，而不是 len
                ans += ((long long)(nums[i-1]+1) + (long long)(nums[i-1]+k)) * k / 2;
                break;
            }

            ans += ((long long)nums[i-1] + (long long)nums[i]) * len / 2;
            k -= len;
        }

        return ans;
    }
};
```

### 第 284 场

排名（4086 / 8483）

- [x] `3 分` - [找出数组中的所有 K 近邻下标](https://leetcode-cn.com/problems/find-all-k-distant-indices-in-an-array/)
- [x] `4 分` - [统计可以提取的工件](https://leetcode-cn.com/problems/count-artifacts-that-can-be-extracted/)
- [ ] `5 分` - [K 次操作后最大化顶端元素](https://leetcode-cn.com/problems/maximize-the-topmost-element-after-k-moves/)
- [ ] `6 分` - [得到要求路径的最小带权子图](https://leetcode-cn.com/problems/minimum-weighted-subgraph-with-the-required-paths/)

这周周赛前三题还是挺简单，第四题还是没看。第三道一直在修改测试用例，差几个用例过不了。当时做题的时候没有好好分清楚情况就开始动手了。分类讨论情况比较多的题目还是得需要在纸上分好情况再动手。

####  K 次操作后最大化顶端元素

- k=0：直接输出 nums[0]；

- n = 1：若 kk 为奇数则输出 -1，否则输出 nums[0]；

- 1≤ k ≤n：有两种小情况，取其中的最大值即可。
  - 可以首先将栈顶的 (k−1) 个元素弹出，最后一步操作加入这些元素中的最大值；
  - 可以将栈顶的 k 个元素弹出，露出 nums[k]。

- k > n：一定能取到所有数中的最大值。

  首先将所有数弹出，然后还剩 (k - n) 步操作，分析如下。

  - 若 (k - n) 是奇数，则反复将最大值加入弹出栈即可。
  - 若 (k - n) 是偶数，先将一个非最大值加入栈，然后反复将最大值加入弹出栈即可。

### 第 285 场

排名（3081 / 7501）

- [x] `3 分` - [统计数组中峰和谷的数量](https://leetcode-cn.com/problems/count-hills-and-valleys-in-an-array/)
- [x] `4 分` - [统计道路上的碰撞次数](https://leetcode-cn.com/problems/count-collisions-on-a-road/)
- [ ] `5 分` - [射箭比赛中的最大得分](https://leetcode-cn.com/problems/maximum-points-in-an-archery-competition/)
- [ ] `6 分` - [由单个字符重复的最长子字符串](https://leetcode-cn.com/problems/longest-substring-of-one-repeating-character/)

一二题做出来了，但是花较久的时间，第三题看着完全没有头绪，后面看解答还是通过枚举法来解决（枚举法已经出现过好多次了，这个得练习一下才行）。第四题用暴力法去解，当然肯定超时，现阶段第四题还是不要想了，先把前三道做了先吧。
