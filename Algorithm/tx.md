1.反转单链表

方法一：3指针法
```
public Node revertLinkList(Node head) {
    //两个以上节点才考虑反转
    if (head == null || head.next == null) {
        return head;
    }
    Node p, q;
    p = head;
    q = head.next;
    head = q.next;
    p.next = null;
    q.next = p;

    while (head.next != null) {
        p = q;
        q = head;
        head = head.next;
        q.next = p;
    }
    head.next = q; //最后一个独立节点之前前面已经反转好的链表
    return head;
}
```

方法二：用一个数组来保存节点，再根据索引重新组装成一个新的链表，此方法比较耗内存；

方法三：