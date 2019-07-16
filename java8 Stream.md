1. 通过集合接口来生成Stream  
    1. collect.stream() // 为集合创建串行流  
    2. collect.parallelStream // 为集合创建并行流，相当于多线程操作，注意：使用线程安全的集合  

2. Stream操作  
    1. Stream的操作分为两种，中间操作(intermediate operation)，最终操作(terminal operation)  
        1. 中间操作返回一个中间流，以供后续操作使用  
        2. 最终操作产生结果  
        
3. 中间操作(intermediate operation)  
    1. filter()   
        1. 对流进行过滤，只留下满足条件的元素，通常使用lambda表达式  
    2. map()
        1. 对流中的元素进行映射，将一个元素转换为另一个元素，可以理解为与forEach()类似，但是forEach()为终端操作  
            
        
        
4. 终端操作(terminal operation)
    1. collect()


5. collect()  
    1. collect()是一个终端操作，将流转换为各种结果  
    2. collect()提供两个重载方法  
        1. <R, A> R collect(Collector<? super T, A, R> collector)  
        2. <R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)  
    
    3. 第一个重载方法接受Collector类作为参数，Collector类可以自定义，也可以使用Collectors工具类  
    
    
6. Collectors工具类
    1. 转换为集合
        1. Collectors.toList()
        2. Collectors.toSet()
        3. Collectors.toMap()
        4. Collectors.toCollection()
    2. 数据统计
        1. 求平均值
            1. Collectors.averagingInt()
            2. Collectors.averagingLong()
            3. Collectors.averagingDouble()
        2. Collectors.counting() // 统计Stream元素个数
        3. Collectors.maxBy() // 指定条件下，Stream最大元素
        4. Collectors.minBy() // 指定条件下，Stream最小元素
        5. 求和
            1. Collectors.summingInt()
            2. Collectors.summingLong()
            3. Collectors.summingDouble()
        6. 求多个常规聚合值，包括count，min，max，sum，average
            1. Collectors.summarizingInt()
            2. Collectors.summarizingLong()
            3. Collectors.summarizingDouble()
        7. Collectors.reducing() // reduce操作
        
    3. 数据分组
        1. Collectors.partitioningBy() // Map<Boolean, T>, 将Stream按照规则分为true和false两部分 
        2. Collectors.groupingBy() // Map<T, R>, 与partitioningBy()的不同的在于Map的key的类型可以为T, 所以可以将数据分为多个组
        
    4. 字符串
        1. Collectors.joining() // 将数据转换为字符串，传递3个参数，分隔符，前缀，后缀
        
    5. 组合Collector
            