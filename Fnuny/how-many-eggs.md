##### Q. 一筐鸡蛋： 1个1个拿，正好拿完。 2个2个拿，还剩1个。 3个3个拿，正好拿完。 4个4个拿，还剩1个。 5个5个拿，还剩1个 6个6个拿，还剩3个。 7个7个拿，正好拿完。 8个8个拿，刚好拿完，问篮子至少几个鸡蛋？


1、3、7、9都可以拿完，说明为7和9的的公倍数，设为 63n，n为整数，由于除以2余1，所以n必定要是奇数

2、4、5、8只剩一个，说明是5和8的公倍数加1，设为 40m+1，m为整数；

所以有以下等式：

63n = 40m+1
m = 63n-1/40

当 n=7时，m为最小值11，63*7=441
