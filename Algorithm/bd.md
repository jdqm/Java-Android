### 1.用O(1)的时间复杂度删除单链表的指定的节点。

删除一个节点容易，可以要在O(1)的时间内删除就要变通一下，思路是删除指定节点的下一个节点，把下一个节点的指放在要删除的节点上，从结果上看，也相当于删除了指定的节点那，但是不能删除最后一个节点。

```
class Node {
    int value;
    Node next;
}

/**
 * 在O（1）的时间复杂内删除指定节点，不能删除最后一个节点
 *
 * @param node 要删除的节点
 */
public static void deleteNode(Node node) {
    if (node.next == null) {
        throw new IllegalArgumentException("不能删除最后一个节点");
    }
    node.value = node.next.value;
    node.next = node.next.next;
}
```