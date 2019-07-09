1. PO: persistant object。持久对象，是与数据库中的表相映射的Java对象，与数据库表一一对应。
2. VO: value object。值对象，通常用于业务层之间的数据传递，是抽象出的业务对象，可以与数据库表对应，也可以不对应，根据业务要求决定。
3. DTO: data transfer object。数据传输对象，有时仅仅需要数据库的几个字段，此时PO就不合适了，应使用DTO，可以理解为VO的一种。
4. BO: business object。业务对象，由Service层输出的封装业务逻辑的对象。
5. DO: Data Object。此对象与数据库表结构一一对应，通过DAO层向上传输数据源对象。
6. AO: ApplicationObject。应用对象，在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
7. POJO: plain ordinary Java object。简单无规则java对象，只有属性和属性对应的setter和getter方法，tostring()方法，前面提到的PO、VO、DTO都可以归为POJO。
8. DAO: data access object。数据访问对象，此对象用于访问数据库。通常和PO、DTO结合使用。
9. Query: 数据查询对象。各层接收上层的查询请求。注意超过2个参数的查询封装，禁止使用Map类来传输。
10. BIZ: 其名称就是商业的简写，也就是其对应的是业务层，此包里的对象通过调用DAO中的方法来完成业务层上的操作，其目的是封装对数据库的操作。