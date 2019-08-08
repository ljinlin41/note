1. 安装redis
    ```shell
    sudo apt update
    sudo apt install build-essential tcl
    cd ~
    mkdir document/
    cd document/
    curl -O http://download.redis.io/redis-stable.tar.gz
    tar zxvf redis-stable.tar.gz
    cd redis-stable/
    make
    make test
    sudo make install
    cp ~/document/redis-stable/src/redis-trib.rb /usr/local/bin
    ```

2. 开始集群搭建  
    1. 修改配置文件
        ```shell
        # 设置配置文件目录
        sudo mkdir -p /etc/redis/redis-cluster/7000
        cd /etc/redis/
        # 创建一份配置文件，Cluster架构-3主3从
        sudo cp redis.conf redis-cluster/7000/redis.conf
        # 修改配置文件
        sudo vim redis-cluster/7000/redis.conf
        ```  
        
        配置文件修改如下部分  
        ```shell
        bind 0.0.0.0 # 外部访问redis
        daemonize    yes  # redis后台运行
        pidfile  /var/run/redis_7000.pid  #需要修改为 reids_{port}.pid 的形式
        port  7000  #端口
        cluster-enabled  yes #开启集群
        cluster-config-file  7000/nodes_7000.conf #集群的配置文件 nodes_{port}.conf的形式
        cluster-node-timeout  5000 #超时时间 5s够了
        appendonly  yes #开启AOF日志
        ```
        
        将配置文件复制5份，并修改每份文件的端口号
        ```shell
        # 创建5份配置文件路径
        sudo mkdir -p /etc/redis/redis-cluster/7001
        sudo mkdir -p /etc/redis/redis-cluster/7002
        sudo mkdir -p /etc/redis/redis-cluster/7003
        sudo mkdir -p /etc/redis/redis-cluster/7004
        sudo mkdir -p /etc/redis/redis-cluster/7005
        # 复制5份配置文件
        cd /etc/redis/
        sudo cp redis-cluster/7000/redis.conf redis-cluster/7001/
        sudo cp redis-cluster/7000/redis.conf redis-cluster/7002/
        sudo cp redis-cluster/7000/redis.conf redis-cluster/7003/
        sudo cp redis-cluster/7000/redis.conf redis-cluster/7004/
        sudo cp redis-cluster/7000/redis.conf redis-cluster/7005/
        # 修改端口号
        sudo sed -i "s/7000/7001/g" /etc/redis/redis-cluster/7001/redis.conf
        sudo sed -i "s/7000/7002/g" /etc/redis/redis-cluster/7002/redis.conf
        sudo sed -i "s/7000/7003/g" /etc/redis/redis-cluster/7003/redis.conf
        sudo sed -i "s/7000/7004/g" /etc/redis/redis-cluster/7004/redis.conf
        sudo sed -i "s/7000/7005/g" /etc/redis/redis-cluster/7005/redis.conf
        ```
    
    2. 启动Redis
        ```shell
        cd /etc/redis/redis-cluster/
        sudo redis-server 7000/redis.conf
        sudo redis-server 7001/redis.conf
        sudo redis-server 7002/redis.conf
        sudo redis-server 7003/redis.conf
        sudo redis-server 7004/redis.conf
        sudo redis-server 7005/redis.conf
        ```
        
    3. 创建集群
        ```shell
        sudo redis-cli --cluster create 192.168.137.25:7000 192.168.137.25:7001 192.168.137.25:7002 192.168.137.25:7003 192.168.137.25:7004 192.168.137.25:7005 --cluster-replicas 1
        # 等一会 然后输入
        yes
        ```
        
    4. 检查集群状态
        ```shell
        redis-cli --cluster check 127.0.0.1:7000  #填写任意节点即可 会带出所有的
        ```
        
3. 连接redis集群
    ```
    redis-cli -c -h 127.0.0.1 -p 7000
    127.0.0.1:7000> set hi redis
    -> Redirected to slot [16140] located at 127.0.0.1:7002
    OK
    127.0.0.1:7002> get hi
    "redis"
    ```

4. 重启集群异常解决  

    1. Node has slots in importing state异常解决  
        redis-cli客户端登录7004、7006端口，分别执行cluster setslot 1180 stable命令

    
