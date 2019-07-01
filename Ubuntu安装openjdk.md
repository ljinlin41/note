1. `sudo apt-get update`  
2. `sudo apt-get install openjdk-8-jdk`  
3. 添加环境变量  
   
    ```shell
    sudo vim ~/.bashrc
    在尾部加入
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${PATH}:${JAVA_HOME}/bin
    ```
4. 重启