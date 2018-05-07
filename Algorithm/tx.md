### 1.反转单链表

#### 方法一：3指针法
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

#### 方法二：将后面的节点逐一插入第一个节点后后面，最后将第一个节点放到链表尾部。

```
public static Node revertLinkList(Node head) {
    //两个以上节点才考虑反转
    if (head == null && head.next == null) {
        return head;
    }
    Node p;
    Node q;
    p = head.next;
    while (p.next != null) {
        q = p.next;
        p.next = q.next;
        q.next = head.next;
        head.next = q;
    }

    p.next = head;//相当于成环
    head = p.next.next;//新head变为原head的next
    p.next.next = null;//断掉环
    return head;
}
```

#### 方法三：递归进行，这种情况需要一个全局变量来记录新链表的头指针。
```
Node newHead;
public Node revertLinkList(Node head) {

    if (head == null){
        return head;
    }

    if ((head.next == null)) {
        newHead = head;
        return head;
    }

    Node newTail = revertLinkList(head.next);
    newTail.next = head;
    head.next = null;
    return head; //输出参数head为新链表的尾指针
}
```
#### 方法四：用一个数组来保存节点，再根据索引重新组装成一个新的链表，此方法比较耗内存；

