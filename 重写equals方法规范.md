1. 方法逻辑顺序
    1. 使用 `==` 判断是否为自身
    2. 使用 `instanceof` 判断类型是否正确
    3. 对参数进行强制转换
    4. 比较两个对象中的值
2. 注意
    1. 重写equals时必须重写hashCode
    2. 不要将equals的参数类型从Object换为其他具体类型

