```java
package com.fisec.vpn.util;


import java.util.List;

// Definition for singly-linked list.
class ListNode {
    int val;
    ListNode next;

    ListNode() {
    }

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}

class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
//        133    4782   5
//        321    2874   3195
        ListNode dummy = new ListNode(-1);
        // 虚拟头结点（构建新链表时的常用技巧）
        ListNode cur = dummy;
        int temp = 0;
        while (l1 != null || l2 != null) {
            ListNode listNode = new ListNode();
            int i = 0;
            int j = 0;
            if (l1 != null) {
                i=l1.val;
            }
            if (l2!=null){
                j=l2.val;
            }
            int res = i+j+temp;
            listNode.val =  res % 10;
            temp = res / 10;
            cur.next=listNode;
            cur=cur.next;
            if (l1 != null) {
                l1 = l1.next;
            }
            if (l2 != null) {
                l2 = l2.next;
            }
        }
        //处理最后的进位
        if (temp!=0){
            ListNode listNode=new ListNode(temp);
            cur.next=listNode;
        }
        return dummy.next;
    }
}




```
