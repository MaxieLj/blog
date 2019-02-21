---
title: 算法-求第k个排序
date: 2018-01-16 23:57:21
tags: 算法
categories: 算法
toc: true
---

## 题目
### 英文
The set [1,2,3,...,n] contains a total of n! unique permutations.

By listing and labeling all of the permutations in order, we get the following sequence for n = 3:

"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the kth permutation sequence.

Note:

Given n will be between 1 and 9 inclusive.
Given k will be between 1 and n! inclusive.
Example 1:

Input: n = 3, k = 3
Output: "213"
Example 2:

Input: n = 4, k = 9
Output: "2314"

### 中文
给出集合 [1,2,3,…,n]，其所有元素共有 n! 种排列。

按大小顺序列出所有排列情况，并一一标记，当 n = 3 时, 所有排列如下：

"123"
"132"
"213"
"231"
"312"
"321"
给定 n 和 k，返回第 k 个排列。

说明：

给定 n 的范围是 [1, 9]。
给定 k 的范围是[1,  n!]。
示例 1:

输入: n = 3, k = 3
输出: "213"
示例 2:

输入: n = 4, k = 9
输出: "2314"
## 思路

- 排列的每一个位置都有一个固定的进位（如果当前数固定，可以有多少种排列），当k达到前进位是，则改位置加1。
- 从最左（进位最大）开始，依次用当前进位对k-1做除法（（k-1）/当前进位)，如果结果不为0，则用`结果`+当前能够使用的最小值，获得当前位置的值。
- 用余数当做k,按照第二步的思路，求其余位置的值。

##代码
```c
char* getPermutation(int n, int k) {
    char* result = (char*)malloc((n + 1) * sizeof(char));
    bool* flag = (bool*)calloc(n, sizeof(bool));
    int temp[10] = { 0 };
    temp[n] = 1;
    temp[n - 1] = 1;
    int i = 0, t = 0, idx = 0, j = 0;
    for(i = n - 2; i > 0; i--)
        temp[i] = temp[i + 1] * (n - i);
    for(i = 1; i <= n; i++) {
        t = (k - 1) / temp[i];
        idx = t;
        for(j = 1; j <= n; j++) {
            //第j小未使用的数，j为上述计算得来。
            if(!flag[j]) {
                t--;
                if(t == -1)
                    break;
            }
        }
        result[i - 1] = '0' + j;
        flag[j] = true;
        k -= idx * temp[i];
    }
    result[n] = '\0';
    free(flag);
    return result;
}
```