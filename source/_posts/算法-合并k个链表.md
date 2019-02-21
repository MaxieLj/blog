---
title: 合并k个链表
tags: 算法
categories: 算法
toc: true
date: 2018-01-10 16:09:18
---

## 题目
合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

## 分析
- 如果理由`最后选择`进行合并链表，这是时间复杂度是`kn`如果 `k` 很大的时候时间复杂度简直爆炸，还不如直接合并。
- 如果使用归并排序 时间复杂度是`nlogk`。
- 如果直接两两暴力合并，时间复杂度是`n(k^2+k)/2`
- 所以当k很大的时候，最优解应该是并归。

## 代码
```c
struct ListNode* merge(struct ListNode* l1, struct ListNode* l2)
{
    struct ListNode* l3 = NULL;
    struct ListNode* p = NULL;
    struct ListNode* tmp = NULL;
    if(!l1 && !l2) {
        return l3;
    }
    if(l1 && !l2) {
        return l1;
    }
    if(!l1 && l2)
    {
        return l2;
    }
    //设置头结点
    if(l1->val < l2->val) {
        l3 = l1;
        l1 = l1->next;
    } else {
        l3 = l2;
        l2 = l2->next;
    }    
    p = l3;
    while(l1 && l2)
    {
        if(l1->val < l2->val)
        {
            tmp = l1->next;
            p->next = l1;
            l1 = tmp;
        } else {
            tmp = l2->next;
            p->next = l2;
            l2 = tmp;
        }
        p = p->next;
    }
    
    if(l1) {
        p->next = l1;
    }
    if(l2)
    {
        p->next = l2;
    }
    return l3;
}

/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* mergeKLists(struct ListNode** lists, int listsSize) 
{
    if(listsSize == 0) {
        return 0;
    }
    if(listsSize == 1) {
        return (*lists);
    }
    struct ListNode* ret = (*lists);
    for(int i = 1 ; i < listsSize; i++)
    {
        lists++;
        ret = merge((*lists), ret);
    }
    return ret;   
}

```

## 后记
尝试寻找更优解法。