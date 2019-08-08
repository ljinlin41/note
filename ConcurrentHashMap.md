1. ConcurrentHashMap分析
    1. tryPresize()
    2. transfer()
    3. putVal()
    4. addCount()
    5. sumCount()

```java
class ConcurrentHashMap
{
    /**
     * Tries to presize table to accommodate the given number of elements.
     *
     * @param size number of elements (doesn't need to be perfectly accurate)
     */
    private final void tryPresize(int size) {
        // 当size未达到MAXIMUM_CAPACITY时，扩容size。调用tableSizeFor()，
        // 此处size + (size >>> 1) + 1 == size*1.5+1，注意, size = table.length << 1
        // 传进来的size已经提前扩容了2倍
        // c == 2**n
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        // 判断条件，sizeCtl >=0 说明 table未初始化或处于正常状态，线程安全
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            // 如果使用put()方法，则会调用initTable()，初始化table[]。但是当直接调用putAll()方法时，
            // 不会调用initTable()方法，因此在这里判断table是否为null
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;
                // CAS设置sizeCtl为-1, 线程安全
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        // 二次判断，线程安全
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            // 设置table[]
                            table = nt;
                            // 将sc=sizeClt设置为容量的75%, n - (n >>> 2) == n*0.75
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        // 设置sizeCtl
                        sizeCtl = sc;
                    }
                }
            }
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            else if (tab == table) {
                int rs = resizeStamp(n);
                // sc < 0 说明正在扩容，因此多线程辅助扩容
                if (sc < 0) {
                    Node<K,V>[] nt;
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    // sc + 1，表示此线程正在辅助扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 否则开始新的扩容
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
    
    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     * 把数组中的节点复制到新的数组的相同位置，或者移动到扩张部分的相同位置
     * 在这里首先会计算一个步长，表示一个线程处理的数组长度，用来控制对CPU的使用，
     * 每个CPU最少处理16个长度的数组元素,也就是说，如果一个数组的长度只有16，那只有一个线程会对其进行扩容的复制移动操作
     * 扩容的时候会一直遍历，知道复制完所有节点，没处理一个节点的时候会在链表的头部设置一个fwd节点，这样其他线程就会跳过他，
     * 复制后在新数组中的链表不是绝对的反序的
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        // CPU的可用线程数，例如4核8线程，NCPU = 8
        // MIN_TRANSFER_STRIDE: 每个线程的处理的数组的长度，最小为16
        // n: 数组长度
        // 当NCPU <= 1，也就是单核单线程时，stride=数组长度，也就是只有1个线程来扩容
        // 当NCPU > 1时, 当stride<16时，直接将stride赋值为16。以8个线程的CPU举例，
        // 只有当 table.length > 16(最小处理数组长度)*8(线程数)*8 时，才能改变stride的值(16)
        // stride设置最小值，防止在容量很小的时候就使用大量的线程进行扩容
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
            
        // 当nextTab == null,  说明这是第一个进行扩容的线程   
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                // n << 1, 说明nextTab大小为原来的两倍 
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            // transferIndex为扩容复制过程中的桶首节点遍历索引
            // 从n开始，表示从后向前遍历
            transferIndex = n;
        }
        int nextn = nextTab.length;
         // ForwardingNode是Node节点的直接子类，是扩容过程中的特殊桶首节点
         // 该类中没有key,value,next，hash值为特定的-1
         // 同时具有一个Node和一个Node[]
         // 创建一个fwd节点，这个是用来控制并发的，当一个节点为空或已经被转移之后，就设置为fwd节点
         // 这是一个标志节点，指向nextTable 
         // 在扩容时才会出现的特殊节点，其key,value全部为null。并拥有nextTable指针引用新的table数组。
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        // 死循环, i 表示数组下标，bound 表示当前线程可以处理的当前桶区间最小下标
        // 例子，当 transferIndex = 64, stride = 16时，一次循环后
        // i = 63, bound = transferIndex - stride = 48
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                // 注意: table[]处理区间的分配方式为从后往前
                // --i >= bound 说明当前线程已经领取了要处理的table[]区间，并且还没有处理完毕，推出while循环进行处理
                // finishing 说明所有的区间都已经处理完毕
                if (--i >= bound || finishing)
                    advance = false;
                // transferIndex 说明所有的区间都已经处理完毕
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                // CAS 修改 transferIndex，即 length - 区间值，留下剩余的区间值供后面的线程使用
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            // 当扩容完成时
            // i<0 说明已经遍历完旧的数组tab
            // i>=n 下方赋值 i=n
            // nextn = nextTable.length，i+n>=nextn说明已经扩容完成 
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 扩容完成
                if (finishing) {
                    nextTable = null;
                    // 将nextTab赋值给table
                    table = nextTab;
                    // sizeCtl = 2n - 0.5n = 1.5n = 2*oldSizeCtl
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                // 线程退出 sizeCtl-1
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // 当有多个线程执行完毕退出时，保证只有最后一个退出的线程可以继续执行，其他线程直接return
                    // 第一个扩容的线程，执行transfer方法之前，会设置 sizeCtl = 
                    // (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2) 	
                    // 后续帮其扩容的线程，执行transfer方法之前，会设置 sizeCtl = sizeCtl+1
                    // 每一个退出transfer的方法的线程，退出之前，会设置 sizeCtl = sizeCtl-1
                    // 那么最后一个线程退出时：
                    // 必然有sc == (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)，
                    // 即 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT 
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    // 最后退出的线程要重新check下是否全部迁移完毕
                    i = n; // recheck before commit
                }
            }
            // 当扩容未完成时
          	// 当前数组中第i个元素为null，用CAS设置成特殊节点forwardingNode(可以理解成占位符)
          	// 表明这个节点已经处理过了
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 如果 f.hash=-1 的话说明该节点为 ForwardingNode,说明该节点已经处理过了
            // 表示已经被其他线程处理了，则直接往前遍历，并发扩容的核心  
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            // 真正的扩容操作，锁住当前节点扩容，防止 putVal 的时候向链表插入数据 
            else {
                // 锁住Node首节点 
                synchronized (f) {
                    // 二次判断，线程安全
                    if (tabAt(tab, i) == f) {
                        // ln低位桶，hn高位桶  
                        Node<K,V> ln, hn;
                        // hash>0, 说明为普通Node节点
                        // TreeBin 的 hash 是 -2
                        if (fh >= 0) {
                            // n = table.length
                            // 因为n都是2的n次方，类似于00010000，因此根据bit为1的位进行与操作，
                            // 将Node分为两类，一类bit指定位=0，一类bit指定位=1
                            // 0&1=0, 1&1=1 
                            // 当结果为0，将其放在低位；当结果为1，将其放在高位
                            // 这里高低位分配是有道理的，低位节点放在新链表的i位，高位节点放在新链表的i+n位，
                            // 这样对扩容后的新链表进行get操作时，hash&(n-1)仍能得到正确的位置，反之则不行  
                            int runBit = fh & n;
                            // lastRun为当前节点链表的末尾   
                            Node<K,V> lastRun = f;
                            // 将lastRun遍历到末尾，并设置runBit为末尾节点的runBit
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                // 当新的hash&n与runBit不同时，将新的hash&n作为runBit
                                // 并且将此节点作为lastRun。
                                // 这样做的目的是减少之后判断链表节点高低位的次数，因为当runBit相同时，
                                // 说明这些节点都会被放置同一高位或低位，因此当runBit不变时，说明后面的节点所属的位置相同  
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            // 设为低位桶
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            // 设为高位桶 
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 当桶位置为单节点时，此单节点会被放置高位或低位
                            // 当桶位置为链表时，此节点链表会被拆分为2个新链表  
                            // 从首节点开始遍历，到lastRun尾节点，尾节点不遍历  
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                // 获取hash, key, value
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                // 将原本的一个链表根据hash&n分为2个链表，构建新链表采用头插法
                                // 无法概括两个新链表相对旧链表的顺序，有很多可能，并不是一个正序，一个倒序  
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            // 将低位节点设置为新链表的i位
                            setTabAt(nextTab, i, ln);
                            // 将高位节点设置为新链表的i+n位
                            setTabAt(nextTab, i + n, hn);
                            // 将旧链表的i位设为ForwardingNode
                            setTabAt(tab, i, fwd);
                            // advance为true，所以当前线程可以重新领取扩容任务
                            advance = true;
                        }
                        // 树节点，红黑树迁移  
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            // 高低位树节点  
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            // 开始遍历
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                // 构造树节点
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                // 将构造好的节点放入低位树  
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                // 将构造好的节点放入高位树  
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            // 将低位树头赋值给ln, 并判断要不要转为链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            // 将高位树头赋值给hn, 并判断要不要转为链表
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            // 将低位节点设置为新链表的i位
                            setTabAt(nextTab, i, ln);
                            // 将高位节点设置为新链表的i+n位
                            setTabAt(nextTab, i + n, hn);
                            // 将旧链表的i位设为ForwardingNode
                            setTabAt(tab, i, fwd);
                            // advance为true，所以当前线程可以重新领取扩容任务
                            advance = true;
                        }
                    }
                }
            }
        }
    }
    
    
    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        // 得到节点Hash
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 第一次put先初始化table[]，懒加载
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // 如果put位置为空，通过CAS设置节点
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 如果put位置正在扩容，则当前线程加入辅助扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            // 如果put位置冲突，则锁住当前Node节点，线程安全
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        // hash>0，说明为Node节点
                        // 树节点的hash = -2
                        if (fh >= 0) {
                            binCount = 1;
                            // 遍历Node链表
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // hash值相同，覆盖value 
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // hash值不同，将节点加入链表末尾  
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // 冲突节点为树节点  
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    // 当链表链表数量>8时，转为红黑树  
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 增加Map计数  
        addCount(1L, binCount);
        return null;
    }
    
    
    
    /**
     * Adds to count, and if table is too small and not already
     * resizing, initiates transfer. If already resizing, helps
     * perform transfer if work is available.  Rechecks occupancy
     * after a transfer to see if another resize is already needed
     * because resizings are lagging additions.
     *
     * @param x the count to add
     * @param check if <0, don't check resize, if <= 1 only check if uncontended
     * check, 当在putVal()中调用时，x = 1, check为节点链表长度
     */
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        
        // 当计数盒子不是空
        // 当并发CAS修改baseCount失败时        
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            // 如果计数盒子是空（尚未出现并发）
            // 如果随机取余一个数组位置为空 或者
            // 修改这个槽位的变量失败（出现并发了）
            // 执行 fullAddCount 方法。并结束
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            // 获取Map中的元素个数
            s = sumCount();
        }
        // 检查是否需要扩容  
        // check为节点链表长度
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            // 当元素个数>sizeCtl进行扩容
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                // rs = Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1))
                // rs = n的最高非零位前面的0的个数 + 32768
                // rs作为table长度的标识符    
                int rs = resizeStamp(n);
                // sizeCtl<0，说明正在扩容中
                if (sc < 0) {
                    // 不需要辅助扩容  
                    // 如果 sc 的高16位左移后不等于 长度标识符（校验异常 sizeCtl 变化了）
                    // 如果 sc == 标识符 + 1 （扩容结束了，不再有线程进行扩容）（默认第一个线程设置 sc ==rs 左移 16 位 + 2，当第一个线程结束扩容了，就会将 sc 减一。这个时候，sc 就等于 rs + 1）
                    // sc == rs + 1为BUG，见https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8214427
                    // 如果 sc == 标识符 + 65535（帮助线程数已经达到最大）
                    // 如果 nextTable == null（结束扩容了）
                    // 如果 transferIndex <= 0 (转移状态变化了)
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    // 进入辅助扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 作为第一个扩容的线程
                // 设置 sc = rs长度标识符的低16位左移至高16位 + 2
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
    
    
    
    final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        // 当CounterCell[]为null时，说明没有并发竞争，直接返回baseCount
        // 当不为null时，说明存在并发，则统计CounterCell[] + baseCount的总数
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
}

```