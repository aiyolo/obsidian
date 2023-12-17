---
uid: 
aliases: null
created: 2022-08-08 17:37:11
updated: 2023-03-21 12:24:26
tags:
  - leetcode
  - review
  - medium
source: https://leetcode.cn/problems/569nqc/
title: 剑指 Offer II 035. 最小时间差 - 力扣（LeetCode）
dg-publish: true
---

# 剑指 Offer II 035. 最小时间差 - 力扣（LeetCode）

[剑指 Offer II 035. 最小时间差 - 力扣（LeetCode）](https://leetcode.cn/problems/569nqc/)

---

给定一个 24 小时制（小时: 分钟 **"HH:MM"**）的时间列表，找出列表中任意两个时间的最小时间差并以分钟数表示。

**示例 1：**

**输入：** timePoints = \["23:59","00:00"\]  
**输出：** 1

**示例 2：**

**输入：** timePoints = \["00:00","23:59","00:00"\]  
**输出：** 0

**提示：**

* `2 <= timePoints <= 2 * 104`
* `timePoints[i]` 格式为 **"HH:MM"**

注意：本题与主站 539 题相同： [https://leetcode-cn.com/problems/minimum-time-difference/](https://leetcode-cn.com/problems/minimum-time-difference/)

---

#review

- sort 函数里的比较函数，可以设成 lambda 表达式，或者自定义的函数
- lanbda 表达式不能隐式捕获 this 指针，所以如果要使用成员函数，则必须显示捕获 this 指针
- 自定义的函数需要设成 static，因为 sort 传给比较函数的只有两个参数，而自定义的比较函数多了一个 this 参数
- 静态函数不能调用非静态函数

```cpp
// 2023-03-21 12:20
class Solution {
public:
    bool cmp(string &s1, string &s2){
       return toMinutes(s1) < toMinutes(s2);
    }
    int toMinutes(string &s1){
        int h1 = stoi(s1.substr(0, 2));
        int m1 = stoi(s1.substr(3, 2));
        return h1*60 + m1;
    }
    int findMinDifference(vector<string>& timePoints) {
	    // this 不能被隐士捕获
	    // 也可以省略第三个参数，不用比较函数
	    // 不使用this捕获的话，要把cmp和toMinutes都换成静态函数
        sort(timePoints.begin(), timePoints.end(), [this](string &s1, string &s2){
            return cmp(s1, s2);
        }); 
        int ans = INT_MAX;
        for(int i=0; i<timePoints.size()-1; i++){
            int tmp = toMinutes(timePoints[i+1]) - toMinutes(timePoints[i]);
            ans = min(tmp, ans);
        }
        // 结果也可能是头-尾
        ans = min(ans, 1440+toMinutes(timePoints[0])-toMinutes(timePoints.back()));
        return ans;
    }
};

```

历史记录

```cpp
class Solution {
public:
    static bool cmp(string &s1, string &s2){
       return toMinutes(s1) < toMinutes(s2);
    }
    static int toMinutes(string &s1){
        int h1 = stoi(s1.substr(0, 2));
        int m1 = stoi(s1.substr(3, 2));
        return h1*60 + m1;
    }
    int findMinDifference(vector<string>& timePoints) {
        sort(timePoints.begin(), timePoints.end());
        int ans = INT_MAX;
        for(int i=0; i<timePoints.size()-1; i++){
            int tmp = toMinutes(timePoints[i+1]) - toMinutes(timePoints[i]);
            ans = min(tmp, ans);
        }
        ans = min(ans, 1440+toMinutes(timePoints[0])-toMinutes(timePoints.back()));
        return ans;
    }
};
```

---

> Don't watch the clock; do what it does. Keep going.  
> — <cite>Sam Levenson</cite>
