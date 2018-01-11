###1.二分查找
####非递归方式
```
public static int binSearch(int[] array, int value) {
    int start = 0;
    int end = array.length - 1;
    int middle;
    while (start <= end) {
        middle = (end - start) / 2 + start;
        if (value == array[middle]) {
            return middle;
        } else if (value < array[middle]) {
            end = middle - 1;
        } else if (value > array[middle]) {
            start = middle + 1;
        }
    }
    return -1;
}
```
####递归实现
```
public static int binSearch(int[] array, int start, int end, int value) {
    if (end < start) {
        return -1;
    }
    int middle = (end - start) / 2 + start;
    if (value > array[middle]) {
        return binSearch(array, middle + 1, end, value);
    } else if (value < array[middle]) {
        return binSearch(array, start, middle - 1, value);
    }
    return middle;
}
```