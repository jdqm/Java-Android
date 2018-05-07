![collection.png](http://upload-images.jianshu.io/upload_images/3631399-e677d55ed3cdf036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![map.png](http://upload-images.jianshu.io/upload_images/3631399-e436e5797306140a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.ArrayList
>Resizable-array implementation of the <tt>List</tt> interface.  Implements all optional list operations, and permits all elements, including null.  In addition to implementing the List interface,this class provides methods to manipulate the size of the array that is used internally to store the list.  (This class is roughly equivalent to Vector, except that it is unsynchronized.

- 底层实现是Object数组
- 默认大小：10
- 扩容机制

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```  
首先扩大当前大小的一半，若还是比需要的大小小，则取需求的大小，若需求大小大于MAX_ARRAY_SIZE，则取最大整形值。其中这个MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8，主要是有一些虚拟机会在数组中存储一些头字段，所以数组的大小超过这个值有可能会触发OutOfMemeryError。使用System.arraycopy()进行数据拷贝。

-优缺点：基本上就是顺序表（数组的优点），可根据索引直接访问，查找快，但是添加删除可能涉及到数据的迁移，这将影响其性能，所以在经常增添、删除的情景下就不适合使用。

## 2.LinkedList

>Doubly-linked list implementation of the List and Deque interfaces. Implements all optional list operations, and permits all elements (including null).

- 底层的数据结构是双向链表
```
class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```
- 优缺点：就是链表的特性，增删高效，查找低效，双向链表在查找的时候根据查找位置距离头尾指针的距离来选择从那边搜索，将时间复杂度从O(n)变成O(n/2)，比较典型的空间换时间。

## 3.HashMap

>Hash table based implementation of the Map interface. This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it isunsynchronizedandpermits nulls.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.

- 底层数据结构一个存储Node<K,V>[] table数组；
- hash的计算方式

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
最终的hash值是：hashCode的高16位不变，第16位修改为高16位与低16位的异或。


索引计算
（n-1）& hash

- 扩容：扩容为原来的两倍，需要重新计算原来元素的位置，原来的元素位置要么在原位置，要么在原来的位置+原来的大小，例如原来是16的容量，索引是5，修改后可能是5或者，16+5。到底是哪个取决于hash的下一位是0还是1。

### 1.画出HashMap的存储结构图。


### 2.HashMap在什么情况下会扩容？


### 3.hash值是如何计算的，为什么要这样计算？


### 4.HashMap是如何解决冲突的？还有哪些解决冲突的方法？


### 5.put和get的过程是如何工作的？get的时间复杂度？


### 6.HashMap和Hashtable有何关联、区别？


