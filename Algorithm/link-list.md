1.一个链表中包含环，请找出该链表的环的入口结点。

```
class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}

public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        ListNode slow = pHead;
        ListNode fast = pHead;
        while(fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if(fast == slow) {
                ListNode p = slow;
                ListNode q = pHead;
                while(p != q){
                    p = p.next;
                    q = q.next;
                }
                return p;
            }
        }
        return null;
    }
}
```
测试地址 https://www.nowcoder.com/practice/253d2c59ec3e4bc68da16833f79a38e4?tpId=13&tqId=11208&rp=3&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking