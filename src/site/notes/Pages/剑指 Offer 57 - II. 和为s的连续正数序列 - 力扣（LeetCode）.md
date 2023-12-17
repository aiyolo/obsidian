---
uid: 
aliases: null
created: 2022-06-09 11:56:00
updated: 2023-03-09 11:24:23
tags:
  - leetcode
  - 连续序列
source: https://leetcode.cn/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/
title: 剑指 Offer 57 - II. 和为 s 的连续正数序列 - 力扣（LeetCode）
dg-publish: true
---

# 剑指 Offer 57 - II. 和为 s 的连续正数序列 - 力扣（LeetCode）

[剑指 Offer 57 - II. 和为s的连续正数序列 - 力扣（LeetCode）](https://leetcode.cn/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

---

示例 1：

输入：target = 9  
输出：``[[2,3,4],[4,5\|2,3,4],[4,5]]``  
示例 2：

输入：target = 15  
输出：``[[1,2,3,4,5],[4,5,6],[7,8\|1,2,3,4,5],[4,5,6],[7,8]]``

---

解题思路

- 假设连续整数序列为 a, a+1, …, a+k，他们的和为 $T$，那么

$$
2a = \frac{2T}{k+1}-k
$$

由于 a 是正数，所以有 $a>0$，因此

$$
\frac{2T}{k+1}-k>0 \Rightarrow k<\sqrt(2T)
$$

另外由于 a 是整数，那么要保证 $\frac{2T}{k+1}$ 为整数，$\frac{\frac{2T}{k+1}-k}{2}$ 为整数

```cpp
class Solution {
public:
    vector<vector<int>> findContinuousSequence(int target) {
        vector<vector<int>> ans;
        for(int i=sqrt(2*target);i>=1; i--){ //放缩，i就是上面的k，数组最少两个，所以i>=1
            vector<int> tmp;
            if((2*target)%(i+1)==0){
                if(((2*target)/(i+1)-i)%2==0){
                    int a = ((2*target)/(i+1)-i)/2;
                    //if(a<=0) continue;// 可以去掉 因为利用了 2a>0 来求k的范围                    
                    for(int j=0; j<=i; j++){
                        tmp.push_back(a+j);
                    }
                    ans.push_back(tmp);
                }
            }
        }
        return ans;
    }
};

```

---

> I destroy my enemies when I make them my friends.  
> — <cite>Abraham Lincoln</cite>
