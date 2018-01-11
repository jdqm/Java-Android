###1.冒泡排序
基本思想：每次比较交换相邻的元素，随着排序的进行，元素越来越少。假设有n个元素，排序趟数是n-1，需要注意一点优化点是，若果一趟排序下来没有交换发生，则说明已经排好续了。
**时间复杂度: O(n*n)**

```
public int[] bubbleSort(int[] array) {
    if (array == null) {
        return array;
    }

    boolean flag = true;
    for (int i = 0; i < (array.length - 1) && flag; i++) {
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
    for (int i = 0; i < array.length - 1 && flag; i++) {
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

2.选择排序

基本思想：每次从剩下的未排序元素元素中，找出最小的值，已拍好的尾部。由于交换次数变少，所以性能略优于bubbleSort，但是这种算法无法察觉到剩余元素是否已经排好序。
**时间复杂度：O(n*n)**

```
public int[] selectSort(int[] array) {
    if (array == null) {
        return array;
    }
    int length = array.length;
    for (int i = 0; i < length - 1; i++) {
        int k = i;
        for (int j = length - 1; j > i ; j--) {
            if (array[j] < array[k]) {
                k = j;
            }
        }
        if (i != k) {
            int tmp = array[i];
            array[i] = array[k];
            array[k] = tmp;
        }
    }
    return array;
}
```

3.插入排序
基本思想：将后面待排序的元素，逐个插入已拍好序的序列中。主要逻辑是前面排好序的要让出一个位置来，从已排序好的最后一个位置开始，一路往前让，当大于前面的元素就插入。（跟打牌一样,只能把后面的往前插）
**时间复杂度：O(n*n)**，性能上优于冒泡排序和选择排序
```
public int[] insertSort(int[] array) {
    if (array == null) {
        return array;
    }
    int length = array.length;
    int j;
    for (int i = 1; i < length; i++) {
        int tmp = array[i]; //这个值要存起来，否自第一次让位置就把它给覆盖了
        for (j = i; j > 0 && tmp < array[j - 1]; j--) {
            array[j] = array[j - 1];
        }
        array[j] = tmp;
    }
    return array;
}
```
4.希尔排序

6.

7.快速排序

基本思想：
（1）取一个基准数，将大于基数数的放在它后面，将小于或者等于基准数的值放到它的前面；
（2）以基准数为界限（不包括基准数），在前后两个序列中各自取一个基准数，然后重复第一步，直到每个序列只有一个数字；
时间复杂度：O(n*lgN)
```
public static void quickSort(int[] array, int left, int right) {
    if (left >= right) {
        return;
    }
    int l = left, r = right;
    int base = array[l]; // 取当前序列的第0个为基准数，挖坑array[0]
    while (l < r) {

        //从后面往前找一个比基准数小的值来填坑
        while (r > l && array[r] >= base) {
            r --;
        }
        if (l < r) {
            array[l] = array[r]; //坑变为array[r]
            l++;
        }

        while (l < r && array[l] < base) {
            l++;
        }
        if (l < r) {
            array[r] = array[l];//坑变为array[l]
            r--;
        }

    }
    array[l] = base;
    //递归进行前后区间，直到最后每个区间只有一个数字（即 left == right）
    quickSort(array, left, l - 1); 
    quickSort(array, r + 1, right);
}
```
