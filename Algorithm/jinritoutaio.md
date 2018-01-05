####1.给定一个N*N的数组，顺时针回旋打印出来。

这道题我个人比较喜欢把它看成，一个一个的“口”字套在一起，每次退去一个”口“字，打印逻辑就是打印四条边，top->right->bottom->left，留意四个角的元素不要重复打印即可。

```
/**
 * 顺时针从外到里打印矩阵
 *
 * @param array Input a N*N Array
 */
public static void printMatrixClockWisely(@NotNull int[][] array) {
    int n = array.length;
    int currentRow = 0;
    int currentColumn = 0;
    int loopCount = n / 2;

    //当n时奇数时，最后剩下一个元素，需要多打印一次
    if (n % 2 != 0) {
        loopCount++;
    }

    while (loopCount > 0) {
        //当前外圈的上边一行
        for (int i = currentRow; i < n - currentColumn; i++) {
            System.out.print(array[currentRow][i] + " ");
        }

        //当前外圈的右边一列，排除掉右上角一个重复元素
        for (int i = currentRow + 1; i < n - currentRow; i++) {
            System.out.print(array[i][n - 1 - currentColumn] + " ");
        }

        //当前外圈的底边一行，排除掉右下角一个重复元素
        for (int i = n - 1 - 1 - currentColumn; i >= currentColumn; i--) {
            System.out.print(array[n - 1 - currentRow][i] + " ");
        }

        //当前外圈的左边边一列，排除掉左下角和左上角两个重复元素
        for (int i = n - 1 - 1 - currentRow; i > currentRow; i--) {
            System.out.print(array[i][currentColumn] + " ");
        }

        //下次循环从下一行、下一列开始
        currentRow++;
        currentColumn++;
        loopCount--;
    }
}
```


####2.给定一个单链表，特征，奇数节点升序，偶数节点降序，要求进行排序。
例如：->1->10->2->9->3->8



###3.给数组插入元素
```
/**
 * @param array    插入的数组
 * @param position 插入位置
 * @param len      当前元素个数
 * @param cap      数组的最大容量
 * @param value    待插入的值
 * @return 插入成功，则返回插入了value的数组，否则会抛出异常
 */
public static int[] insert(int[] array, int position, int len, int cap, int value) {
    if (position < 0 || position > len) {
        throw new IllegalArgumentException("position必须大于0且小于等于len");
    }

    //如果已经满了，扩容至2倍
    if (len == cap) {
        int[] newArray = new int[cap << 2];
        System.arraycopy(array, 0, newArray, 0, cap);
        array = newArray;
    }

    //把position位置及后面的元素往后移动
    for (int i = len; i > position; i--) {
        array[i] = array[i - 1];
    }
    array[position] = value;
    return array;
}
```

这道题目比较简单，考察的是顺序表的基本操作，但个人感觉这道题目有点不切实际，理论上来讲，len和cap是不应该作为参数传递进来的，所以这个我并没有去考虑这两个参数的合法性。
