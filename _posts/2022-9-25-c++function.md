---
layout: post
title: "function"
date: 2022-09-25
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

## function

```C++
int rotatedDigits(int n) {
        vector<int> digits;
        while(n) {
            digits.push_back(n % 10);
            n /= 10;
        }
        reverse(digits.begin(), digits.end());

        memset(mem, -1, sizeof(mem));
        function<int(int, bool, bool)> dfs = [&](int pos, bool bound, bool diff) -> int {
            if(pos == digits.size())
                return diff;
            if(mem[pos][bound][diff] != -1)
                return mem[pos][bound][diff];
            
            int ret = 0;
            for(int i = 0; i <= (bound ? digits[pos] : 9); i++) {
                if(check[i] != -1)
                    ret += dfs(pos + 1, bound && i == digits[pos], diff || (check[i] == 1));
            }

            return mem[pos][bound][diff] = ret;
        };

        int ret = dfs(0, true, false);
        return ret;
}
```

- `reverse` 可以对数组、字符串、vector容器中的元素进行翻转操作
- `memset` 将数字以单个字节逐个拷贝的方式放到指定的内存中去（没太理解）
- `lambda function` 也可以称之为匿名 function，你可以向 lambda function传入参数，lambda function 也可以返回参数，它和 function 唯一的区别就是没有名字，但是因为要用深度遍历，所以这里给该匿名函数绑定了一个函数指针（没查到这个到底是什么用法，但是感觉应该是这个功能），总结一下写法：
  
  ```c++
  function<return type(params)> name = []() -> {}
  ```

- 这里 return 的时候先赋值是一个挺有意思的写法