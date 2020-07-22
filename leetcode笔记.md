[TOC]

## 贪心

### 最小差值

https://leetcode-cn.com/problems/smallest-range-ii/

我一开始理解错了，以为是A[i]加K或减K之后放进A里，然而题目是直接把运算后的结果替换A[i]。

先给A进行排序。要想让B的最大最小值的差值最小，应该让前面i个元素集体+K，剩余的元素集体-K。

则最大值一定在转折点附近或者末尾产生；最小值也一定在转折点附近或者开始产生。然后使用最大值减去最小值得到差值。

遍历每个i，选择差值最小的。

参考这个解法：

https://leetcode-cn.com/problems/smallest-range-ii/solution/tai-nan-liao-zhi-neng-hua-tu-ping-zhi-jue-by-user8/

```
int maxn = max(A[i]+K, A[A.size()-1]-K);
int minn = min(A[0]+K, A[i+1]-K);
```

完整代码：

```
class Solution {
public:
    int smallestRangeII(vector<int>& A, int K) {
        sort(A.begin(), A.end());
        if(K<0) K=-K;

        if(A.size()==1)
            return 0;
        int gap = A[A.size()-1]-A[0];

        for(int i=0; i<A.size()-1; i++){
            //将i及之前的+K，i之后的减K
            int maxn = max(A[i]+K, A[A.size()-1]-K);
            int minn = min(A[0]+K, A[i+1]-K);
            if(gap>maxn-minn)
                gap = maxn-minn;
        }
        return gap;
    }
};
```





### 坏了的计算器

https://leetcode-cn.com/problems/broken-calculator/

只经过简单运算（乘法、加法）让绝对值较小的数到绝对值较大的数时，乘法可能造成大量间隔，而这些间隔通常只有不断做减法才能达到，这就导致中途产生的状态异常的多，超时。 这时候不如让反向目标数做加法和除法，等目标数小于输入数时，直接减法就够了。

```
class Solution {
public:
    int brokenCalc(int X, int Y) {
        int ans = 0;
        while(Y>X){
            ans++;
            if(Y%2==1)
                Y++;
            else Y = Y/2;
        }
        return ans+X-Y;
    }
};
```







## 动态规划

### <font color =red>摆动序列</font>

https://leetcode-cn.com/problems/wiggle-subsequence/

说是贪心，其实考动态规划。摆动上涨只和上一个元素的摆动下降有关，同样，摆动下降只和上一个元素的摆动上涨有关。

https://leetcode-cn.com/problems/wiggle-subsequence/solution/tan-xin-si-lu-qing-xi-er-zheng-que-de-ti-jie-by-lg/

> - nums[i+1] > nums[i]
>   假设down[i]表示的最长摆动序列的最远末尾元素下标正好为i，遇到新的上升元素后，up[i+1] = down[i] + 1，这是因为up一定从down中产生（初始除外），并且down[i]此时最大。
>   假设down[i]表示的最长摆动序列的最远末尾元素下标小于i，设为j，那么nums[j:i]一定是递增的，因为若完全递减，最远元素下标等于i，若波动，那么down[i] > down[j]。由于nums[j:i]递增，down[j:i]一直等于down[j]，依然满足up[i+1] = down[i] + 1。
> - nums[i+1] < nums[i]，类似第一种情况
> - nums[i+1] == nums[i]，新的元素不能用于任何序列，保持不变



<img src="https://pic.leetcode-cn.com/dd09644d01ea873cfb14a3d538c7b6b49680f5d840e22f3eef6a5e07aec78db0-image.png" alt="image.png" style="zoom:85%;" />

```
public int wiggleMaxLength(int[] nums) {
    int down = 1, up = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] > nums[i - 1])
            up = down + 1;
        else if (nums[i] < nums[i - 1])
            down = up + 1;
    }
    return nums.length == 0 ? 0 : Math.max(down, up);
}
```

