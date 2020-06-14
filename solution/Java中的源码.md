### 1. HashMap 中计算初始容量

```java
    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 刚好大于 cap 的 2 的整数幂。x=2^n，满足:2^(n-1)<c<2^n
    // int n = cap - 1; 是因为避免 cap 原来就是 2^n 的情况
    // n |= n >>> 1; 等 5 步操作，是将 n 最高位的1，铺满其之后的所有低位
    // 为什么是 5 次？因为32位整型数，最多只要5次！最后一次 n >>> 16 是将高16位与低16位或
    // 之后 n 如果不越界，则 n + 1 会变成 2^n 形式，满足语义
    // 示例： 
    // cap=13               二进制表示为： 00000000 00000000 00000000 00001101，计算之后，结果应该为 16
    // int n = cap - 1;     二进制表示为： 00000000 00000000 00000000 00001100
    // n |= n >>> 1;        二进制表示为： 00000000 00000000 00000000 00001110
    // n |= n >>> 2;        二进制表示为： 00000000 00000000 00000000 00001111
    // n |= n >>> 4;        二进制表示为： 00000000 00000000 00000000 00001111
    // n |= n >>> 8;        二进制表示为： 00000000 00000000 00000000 00001111
    // n |= n >>> 16;       二进制表示为： 00000000 00000000 00000000 00001111
    // 最终：n 没有越界，返回 16
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
