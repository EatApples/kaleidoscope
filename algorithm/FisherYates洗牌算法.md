### 来自 wiki
@see https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle
```
-- To shuffle an array a of n elements (indices 0..n-1):
for i from n−1 downto 1 do
     j ← random integer such that 0 ≤ j ≤ i
     exchange a[j] and a[i]
```

### 来自 JDK
@see Collections.shuffle(list, rnd)
```java
// Shuffle array
for (int i=size; i>1; i--)
  swap(arr, i-1, rnd.nextInt(i));
```
这两个表达的意思是一样的！
