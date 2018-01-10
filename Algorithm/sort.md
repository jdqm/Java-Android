###1.冒泡排序
假设有n个元素，排序趟数是n-1，需要注意一点优化点是，若果一趟排序下来没有交换发生，则说明已经排好续了。
**时间复杂度: O(n*n)**

```
public int[] bubbleSort(int[] array) {
    if (array == null) {
        return array;
    }

    boolean flag = true;
    for (int i = 0; i < (array.length - 1) & flag; i++) {
        System.out.println("排序");
        flag = false;
        //每次把剩下的最大值沉入底部
        for (int j = 0; j < array.length - 1 - i; j++) {
            //当新一轮比较没有交互发生，说明已经排好了
            if (array[j] > array[j + 1]) {
                int tmp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = tmp;
                flag = true;
            }
        }
    }
    return array;
}

public int[] bubbleSort(int[] array) {
    if (array == null) {
        return array;
    }
    boolean flag = true;
    for (int i = 0; i < array.length - 1 & flag; i++) {
        System.out.println("排序");
        flag = false;
        //每次把剩下的最小的往上浮动
        for (int j = array.length - 1; j > i; j--) {
            //当新一轮比较没有交互发生，说明已经排好了
            if (array[j] < array[j - 1]) {
                int tmp = array[j];
                array[j] = array[j - 1];
                array[j - 1] = tmp;
                flag = true;
            }
        }
    }
    return array;
}
```

###2.快速排序
