1. 线程共享数据区
    1. java堆
    2. 方法区
        1. 运行时常量池
2. 线程私有数据区
    1. 程序计数器
    2. java虚拟机栈(俗称java栈)
        1. 栈帧
    3. 本地方法栈
3. 直接内存    

4. java堆
    1. 垃圾回收的主要区域
    
5. 方法区
    用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等。用于存放Class的相关信息，如类名，访问修饰符，常量池，字段描述，方法描述等
    
    
6. java内存溢出  
    1. java堆溢出  
    2. 虚拟机栈和本地方法栈溢出  
    3. 方法区和运行时常量池溢出
    4. 本机直接内存溢出    