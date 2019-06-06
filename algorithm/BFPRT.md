### 排序问题的概率解释
对于给定的序列 A1，A2，A3，...，An，共 N 个元素（元素可以相同，也可以不相同，对结果没有影响），其全排列共有 N! 种可能。

其有序序列，不管是升序或者降序，都是只是其中一种可能的序列。使用比较的方式，每次比较能够确定2个元素的先后次序，也就是说，最多能减少 N!/2 个不符的序列。现在的问题是，需要多少次比较，才能得到这个有序序列？

我们得到 2^x>=N!，两边取对数，得 x >= log(N!)

```
其中：log(N!) = log(N*(N-1)*(N-2)*...N/2*...*3*2*1)
             > log(N*(N-1)*(N-2)*...N/2)
             > log(N/2*N/2*N/2*...N/2) （共 N/2 个 N/2 相乘）
             = N/2*log(N/2)
```
意味着比较排序的下界是 o(NlogN)

### 7 次比较
问题描述：给定 5 个不同的元素，最多比较 7 次，使其有序。

因为：2^7=128 > 5! = 120，故存在这样一种排序算法。

排序过程见扩展阅读。

### 快排
快排算法是由 Charles Antony Richard Hoare (Tony Hoare or C.A.R. Hoare, born January 11, 1934) 在 1960 年（注意，当时他只有26岁！）发明的，是世界上应用最广的排序算法。

Tony Hoare 于 1980 年获得图灵奖。并于 2000 年由英女王授予爵士爵位。

快排算法实现简单，原地排序（只需要很小的额外空间），平均 O(NlogN) 的时间复杂度。而且因为内循环短小，意味着能利用局部性原理（时间局部性与空间局部性），加快处理。

排序算法这么多，就快排敢叫" Quicksort"，一个字，“快”就完事了！

在快排中，切分元素的选取非常关键。快排的最好情况是，每次都能正好将元素对半分。平均而言，切分元素都能落在待排元素的中间。所谓成也萧何，败也萧何，不当的切分元素可能使快排的性能急速退化，最坏退化为平方级别。

一个简单的做法就是在排序前，将待排元素随机地打乱（洗牌算法就可以办到）。Tony Hoare 在发明快排的时候，就已经推荐了这种方式。

### 快速切分
注意到快排算法中的切分（PARTITION）过程有个副作用，即每次切分完后，用于切分的元素就已经在正确的位置。对于快速选择问题，即查找第 K 大元素，就可以使用切分过程来做。

```
PARTITION(Comparable[] a, int lo, int hi)
{
  int i = lo, j = hi+1;
  while (true)
  {
    while (less(a[++i], a[lo]))
      if (i == hi) break;

    while (less(a[lo], a[--j]))
      if (j == lo) break;

    if (i >= j) break;

    exch(a, i, j);
  }
  exch(a, lo, j);
  return j;
}
```

事实上，Tony Hoare 在发表的论文中，除了快排算法，还有快速选择算法 FIND：

```
FIND(Comparable[] a, int k)
{
  int lo = 0, hi = a.length - 1;
  while (hi > lo)
  {
    int j = PARTITION(a, lo, hi);
    if (j < k) lo = j + 1;
    else if (j > k) hi = j - 1;
    else return a[k];
  }
  return a[k];
}

```
FIND 算法的平均时间复杂度为线性O(N)。与快排算法一样，依赖于 PARTITION 的选择，最坏时退化为 O(KN)。

### BFPRT 算法
对于前面的查找第 K 大元素问题（find top K），还可以先构建 K 个元素的大顶堆，然后扫描剩余的 N-K 个元素，维护这个大顶堆，最终输出堆顶元素。时间复杂度为 O(NlogK)。

类似的，先构建 N 元素的小顶堆，然后类似堆排序，输出第 K 个元素，时间复杂度为 O(N+KlogN)。

BFPRT 算法能在线性时间查找第K大元素，俗称“中位数之中位数算法”。理论上已达到最优。

算法伪代码（从论文《Time Boundsfor Selection》中析出）：
```
PICK（为啥叫 PICK 呢？是为了和快排的 FIND 做区分）：从数组 S 中找出第 i 大元素，其中数组的大小为 n，1 <= i <= n

1. 选择：从数组 S 中选择一个元素 m，用作划分
(a) 将数组 S 切分成 n/c 列，每列 c 个元素，分别对每列元素排序
(b) 令 T =  从已排序的 n/c 个列中，取出每列的第 d 个元素，组成新的数组；令 m = 数组 T 的第 b 大元素。这个求解 m 的过程，当 n/c >1 时，可以递归使用 PICK 算法

2. 比较: 将 S 中与 m 大小关系未知的元素（之前排序后的某些元素与 m 已有全序关系，就不用再比较），与 m 比较，就能得出 m 所在数组的确定的位置，记作 j，1<= j <=n

3. 切分: 如果 m 刚好是数组 S 的第 i 大元素，即 i=j，则算法结束；如果 j > i，则将 S 中 >= m 的元素全部剔除，此时，S 中的元素全部 < m，从这个新的数组 S 中（大小为 j ）继续寻找第 i 大元素；如果 j < i，则将 S 中 <= m 的元素全部剔除，此时，S 中的元素全部 > m，从这个新的数组 S 中（大小为 n-j ）继续寻找第 i-j 大元素

若算法没有结束，则回到第1步。
```
具体的算法实现见扩展阅读。

为什么分组的时候选择 5 元素一个组，而不是其他的个数？
```
在论文中，作者证明了，这个划分的元素个数 c，必须 c>=5，否则 PICK 算法将不再是线性时间复杂度。

对于每列 c 个元素，在 PICK 算法中是需要排序的。而我们又在前文中提到了，对于比较的排序算法，比较次数 h(c) 满足： h(c) >= log(c!)。当 c 越大，比较次数也就越大，PICK 算法中常数系数 h(c)/c 的值也就越大。

由此可知，选择5个元素一组是合理的（排序只需要 7 次比较！）。
```

### 作者轶事
BFPRT 算法以 5 位作者的姓的首字母命名（类似的还有大家熟悉的 KMP 算法，RSA 算法等），他们分别是 Manuel Blum，Robert W. Floyd，Vaughan Pratt，Ron Rivest 与 Robert Tarjan。这五个人，个个大佬。

Manuel Blum，计算复杂性理论的主要奠基人之一，于 1995 年获得图灵奖。

Robert W. Floyd，发明了最短路径算法，且是堆排算法的发明者之一，于 1978 年获得图灵奖。

Vaughan Pratt，KMP 算法的发明者之一（其中的 P 就是他）。导师是著名的 Donald Ervin Knuth。

Ron Rivest，RSA 算法的发明者之一（其中的 R 就是他），于 2002 年获得图灵奖。

Robert Tarjan，发明了强连通分量算法，于 1986 年获得图灵奖。

由此推之，BFPRT 算法也可称“五佬星算法”。

### 扩展阅读
#### 1. 算法中对于用七次比较完成5个元素的排序
https://blog.csdn.net/chopstics/article/details/11929463
#### 2. Median of medians
https://en.wikipedia.org/wiki/Median_of_medians
#### 3. BFPRT
https://blog.csdn.net/hebtu666/article/details/83070107
#### 4. BFPRT算法论文：《Time Boundsfor Selection》
http://people.csail.mit.edu/rivest/pubs/BFPRT73.pdf
