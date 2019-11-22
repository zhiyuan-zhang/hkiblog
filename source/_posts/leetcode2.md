---
title: 乐扣刷题2 两数相加
date: 2017/11/18
categories:
  - java
tags:
  - 
comments: true
abbrlink: 49197
img:
---


大概思路都在下面备注里

```java

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {

    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode a = new ListNode(0);
        ListNode p = l1, q = l2,curr = a;
        int carry = 0;

        while(p != null || q != null){
            int x = (p != null) ? p.val : 0;
            int y = (q != null) ? q.val : 0;
            int sum = carry + x + y;

            // 只能是0 或者 1
            carry = sum / 10;

            //读取个位数 加入next
            curr.next = new ListNode(sum % 10);

            // 激活下一位
            curr = curr.next;

            // 赋值两个链表
            if(p != null) p = p.next;
            if(q != null) q = q.next;
        }


        // 当最后大于1 但是while不会循环 所以需要给new一个listNode来补充 比如 11 99 110 用来最后添加这个1的
        if(carry > 0){
            curr.next = new ListNode(carry);
        }

        return a.next;

    }
}

```
