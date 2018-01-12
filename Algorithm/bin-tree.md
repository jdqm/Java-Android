####1.二叉树的递归创建
```
static class Node {
    String value;
    Node leftChild;
    Node rightChild;
}

public static Node createBinTree() {
    Scanner scanner = new Scanner(System.in);
    String value = scanner.next();
    Node root;
    if ("#".equals(value)) {
        root = null;
    } else {
        root = new Node();
        root.value = value;
        root.leftChild = createBinTree();
        root.rightChild = createBinTree();
    }
    return root;
}

public static void visitTreeNode(Node node) {
    if (node != null) {
        System.out.print(node.value + " ");
    }
}

/**
 * 递归先序遍历二叉树
 *
 * @param root 根节点
 */
public static void preOrder(Node root) {
    if (root == null) {
        return;
    }
    visitTreeNode(root);
    preOrder(root.leftChild);
    preOrder(root.rightChild);
}
```
####2.二叉树的递归遍历

```
/**
 * 递归中序遍历二叉树
 *
 * @param root 根节点
 */
public static void inOrder(Node root) {
    if (root == null) {
        return;
    }
    inOrder(root.leftChild);
    visitTreeNode(root);
    inOrder(root.rightChild);
}

/**
 * 递归中序遍历二叉树
 *
 * @param root 根节点
 */
public static void postOrder(Node root) {
    if (root == null) {
        return;
    }
    postOrder(root.leftChild);
    postOrder(root.rightChild);
    visitTreeNode(root);
}
```
####3.二叉树的非递归遍历
```
```



####4.按层级遍历
```
/**
 * 按层级从上到下，从左到右遍历
 *
 * @param root
 */
public static void levelOrder(Node root) {
    System.out.print("按层级打印:");
    if (root == null) {
        return;
    }
    Queue<Node> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        Node node = queue.poll();
        visitTreeNode(node);
        if (node.leftChild != null) {
            queue.offer(node.leftChild);
        }
        if (node.rightChild != null) {
            queue.offer(node.rightChild);
        }
    }
}
```