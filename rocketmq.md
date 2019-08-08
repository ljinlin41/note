0. 启动rocketMQ
    1. `start mqnamesrv.cmd`  
    2. `start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true`  
    

1. topic  
    1. 第一级消息类型
    
2. tag
    1. 第二级消息类型，是topic的二级分类
    
3. GroupName
    1. 具有相同角色的生产者组合或消费者组合，称为生产者组或消费者组。在消费者组中，可以实现消息消费的负载均衡和消息容错目标。
    
4. 4个组件
    1. nameServer，主要功能是为整个MQ集群提供服务协调与治理，具体就是记录维护Topic、Broker的信息，及监控Broker的运行状态。为client提供路由能力。
    2. broker，分为master，slave
    3. producer，Producer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。
    4. consumer，Consumer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。

    
    
    