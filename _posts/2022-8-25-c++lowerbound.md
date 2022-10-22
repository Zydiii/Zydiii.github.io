---
layout: post
title: "查找工具——lower_bound"
description: "数组神器"
date: 2022-08-25
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

## std::lower_bound(), std::upper_bound()

- `std::lower_bound()` 在指定区域内查找不小于目标值的第一个元素

- `std::upper_bound()` 在指定范围内查找大于目标值的第一个元素

```C++
// lower_bound/upper_bound example
#include <iostream>     // std::cout
#include <algorithm>    // std::lower_bound, std::upper_bound, std::sort
#include <vector>       // std::vector

int main () {
  int myints[] = {10,20,30,30,20,10,10,20};
  std::vector<int> v(myints,myints+8);           // 10 20 30 30 20 10 10 20

  std::sort (v.begin(), v.end());                // 10 10 10 20 20 20 30 30

  std::vector<int>::iterator low,up;
  low=std::lower_bound (v.begin(), v.end(), 20); //          ^
  up= std::upper_bound (v.begin(), v.end(), 20); //                   ^

  std::cout << "lower_bound at position " << (low- v.begin()) << '\n';
  std::cout << "upper_bound at position " << (up - v.begin()) << '\n';

  return 0;
}
```

- 在今天的每日一题中遇到了自定义 upper_bound 比较函数的写法，可以比较多维数组，他在找第一个满足比较函数的数据，比如这里我们要找第一个 jobs[i-1][0] < job[1] 的，这样写就比较方便而且快速，因为这个函数是用的二分查找

```C++
class Solution {
public:
    int jobScheduling(vector<int>& startTime, vector<int>& endTime, vector<int>& profit) {
        int n = startTime.size();
        vector<vector<int>> jobs(n);
        for(int i{0}; i < n; i++) {
            jobs[i] = {startTime[i], endTime[i], profit[i]};
        }
        sort(jobs.begin(), jobs.end(), [&](auto a, auto b){return a[1] < b[1];});

        vector<int> dp(n + 1);
        for(int i = 1; i <= n; i++) {
            int j = upper_bound(jobs.begin(), jobs.begin() + i - 1, jobs[i-1][0], [&](int st, vector<int> &job) {return st < job[1];}) - jobs.begin();
            std::cout << i << " " << j << std::endl;
            dp[i] = max(dp[i - 1], dp[j] + jobs[i - 1][2]);
        }

        return dp[n];
    }
};
```