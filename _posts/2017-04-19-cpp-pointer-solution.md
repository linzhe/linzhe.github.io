---
title: c中指针的巧妙使用
date: 2017-04-19 22:28:58
tags: 
    - 指针
    - 链表
categories:
    - leetcode
---

# 19. Remove Nth Node From End of List

题目中要求从一个链表中删除倒数第n个节点，主要考查两个方面的知识，双指针与链表删除操作。
比较巧妙的是指针的指针last的使用，很值得学习，顺便复习一下指针的知识。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if (!head) {
            return head;
        }
        ListNode* prev = head;
        ListNode** last = &head;
        for (int i=1; i<n; ++i) {
            prev = prev->next;
        }
        while (prev->next != NULL) {
            prev = prev->next;
            last = &((*last)->next);
        }
        *last = (*last)->next;
        return head;
    }
};
```

