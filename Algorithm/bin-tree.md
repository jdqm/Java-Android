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

```
####2.二叉树的递归遍历

```
/**
 * 递归先序遍历二叉树
 *
 * @param root 根节点
 */
public static void preOrder(Node root) {
    if (root != null) {
        visitTreeNode(root);
        if (root.leftChild != null) {
            preOrder(root.leftChild);
        }
        if (root.rightChild != null) {
            preOrder(root.rightChild);
        }
    }
}

/**
 * 递归中序遍历二叉树
 *
 * @param root 根节点
 */
public static void inOrder(Node root) {
    if (root != null) {
        if (root.leftChild != null) {
            inOrder(root.leftChild);
        }
        visitTreeNode(root);
        if (root.rightChild != null) {
            inOrder(root.rightChild);
        }
    }
}

/**
 * 递归后序遍历二叉树
 *
 * @param root 根节点
 */
public static void postOrder(Node root) {;
    if (root != null) {
        if (root.leftChild != null) {
            postOrder(root.leftChild);
        }
        if (root.rightChild != null) {
            postOrder(root.rightChild);
        }
        visitTreeNode(root);
    }
}
```
####3.二叉树的非递归遍历
```
/**
 * 非递归先序遍历，访问根节点，入栈根节点，访问左子树，出栈访问右子树
 *
 * @param root
 */
public static void notRecursionPreOrder(Node root) {
    System.out.print("非递归前序遍历：");
    Stack<Node> stack = new Stack<>();
    while (root != null || !stack.isEmpty()) {
        if (root != null) {
            visitTreeNode(root);
            stack.push(root);
            root = root.leftChild;
        } else {
            root = stack.pop();
            root = root.rightChild;
        }
    }
}
```
```

/**
 * 非递归中序遍历，访问根节点，入栈根节点，访问左子树，出栈访问右子树
 *
 * @param root
 */
public static void notRecursionInOrder(Node root) {
    System.out.print("非递归中序遍历：");
    Stack<Node> stack = new Stack<>();
    while (root != null || !stack.isEmpty()) {
        if (root != null) {
            stack.push(root);
            root = root.leftChild;
        } else {
            root = stack.pop();
            visitTreeNode(root);
            root = root.rightChild;
        }
    }
}
```
```
/**
 * 非递归后续遍历
 * <p>
 * 定义两个引用：cur引用当前栈顶元素，per引用上次出栈元素:
 * (1)遇到任意节点先入栈，将cur指向当前栈顶元素
 * (2)若cur的左右孩子都是null，或者其左右孩子都输出完了(per == cur.leftChild || per == cur.rightChild)，则输出cur，并将其出栈，per引用出栈元素
 * (3)如果（2）的条件不满足，则分别将其右孩子、左孩子入栈，注意修改cur，继续步骤（2）
 * (4)直到栈空结束遍历
 *
 * @param root
 */
public static void notRecursionPostOrder(Node root) {
    System.out.print("非递归后续遍历：");
    Stack<Node> stack = new Stack<>();
    Node per = null, cur;
    stack.push(root);

    while (!stack.isEmpty()) {
        cur = stack.peek();
        //左右孩子为null，或者最后孩子都输出完了，此时可以输出根节点
        if ((cur.leftChild == null && cur.rightChild == null) || (per != null && (per == cur.leftChild || per == cur.rightChild))) {
            visitTreeNode(cur);
            per = stack.pop(); //per指向上一个出栈元素
        } else {
            //将右孩子入栈
            if (cur.rightChild != null) {
                stack.push(cur.rightChild);
            }

            if (cur.leftChild != null) {
                stack.push(cur.leftChild);
            }
        }
    }
}
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