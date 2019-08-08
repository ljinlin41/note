1. 准备编译环境  

    ```shell
    sudo apt-get install -y zip unzip build-essential libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libasound2-dev ccache gawk m4 libasound2-dev  libxrender-dev xorg-dev xutils-dev binutils libmotif-dev ant
    sudo apt-get install openjdk-7-jdk // 或者自己安装jdk7，oracle的jdk7也可以
    ```
    
2. 下载openjdk8源码

    ```shell
    wget https://download.java.net/openjdk/jdk8u40/ri/openjdk-8u40-src-b25-10_feb_2015.zip
    # 解压
    unzip openjdk-8.zip
    ```
    
3. 编译
    
    ```shell
    # 进入源码目录
    cd openjdk/
    # 环境设置
    export LANG=C
    export PATH=/usr/lib/jvm/java-7-openjdk-amd64/bin:${PATH}
    unset JAVA_HOME
    unset CLASSPATH
    # 开始编译
    sudo bash ./configure 
    sudo make all DISABLE_HOTSPOT_OS_VERSION_CHECK=OK ZIP_DEBUGINFO_FILES=0
    ```
    
4. 使用
    1. 编译完成后进入 ./build/机器名/jdk, 这里就是编译完成的jdk，将JAVA_HOME设置为此目录，运行java -version即可发现jdk是编译的版本      
    
    
5. 问题解决
    
    1. C++版本过高
    
        ```
        generation.hpp:425:17: warning: invalid suffix on literal; C++11 requires a space between literal and string macro [-Wliteral-suffix]
        ```
        原因: C++编译环境版本过高
        解决方案: 安装gcc-4.8, g++4.8, 并设为默认 
        
    2. 内核版本过高
    
        ```
        # 内核版本不支持
        sudo vim ./hotspot/make/linux/Makefile
        # 找到SUPPORTED_OS_VERSION这行，后面添加4%
        ```
        
    3. 忽略警告
    
        ```
        在./hotspot/make/linux/makefiles/gcc.make文件中找到WARNINGS_ARE_ERRORS = -Werro，注释该段或改成WARNINGS_ARE_ERRORS = -Wno-all。再编译就会忽略掉警告，直到编译完成。
        ```
    
    