1. ThreadLocal本身不存储数据
2. ThreadLocal的数据保存在Thread.threadLocals中，类型为ThreadLocalMap
3. 多个ThreadLocal对应Thread中的同一个Map