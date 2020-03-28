```java
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    // 刚好大于c的2的整数幂。x=2^n，满足:2^(n-1)<c<2^n
    private static final int tableSizeFor(int c) {
        int n = c - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
