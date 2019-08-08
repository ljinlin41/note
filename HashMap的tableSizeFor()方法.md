0. 实现  

    ```
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
1. 作用：输入一个值，返回一个最接近的2的整数次幂的数。例如3->4, 6->8, 50->64。
2. 实现：通过最高位右移4次，将001x xxxx变为0011 1111，最后+1即可得到最接近的2的整数次幂。