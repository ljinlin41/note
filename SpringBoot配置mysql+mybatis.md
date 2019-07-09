1. 配置数据库连接 application.properties
   
    ```
    spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test
    spring.datasource.username=root
    spring.datasource.password=8848
    ```

2. 导入依赖

    ```
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
        <version>4.0.0</version>
    </dependency>

    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
            <source>${jdk.version}</source>
            <target>${jdk.version}</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.6</version>
            <configuration>
            <configurationFile>
                <!-- 配置文件地址，下面有模板 -->
                ${basedir}/src/main/resources/generator/generatorConfig.xml
            </configurationFile>
            <overwrite>true</overwrite>
            <verbose>true</verbose>
            </configuration>
            <dependencies>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.15</version>
            </dependency>
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper</artifactId>
                <version>4.0.0</version>
            </dependency>
            </dependencies>
        </plugin>
    </plugins>
    ```

3. 配置文件 generatorConfig.xml

    ```
    <!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
            "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

    <generatorConfiguration>

        <!-- 导入外部配置属性，可以不用，直接设置 -->
        <properties resource="config.properties"/>

        <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>

            <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
                <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
                <property name="caseSensitive" value="true"/>
                <property name="lombok" value="Getter,Setter,ToString"/>
            </plugin>

            <!-- 配置数据库链接 -->
            <jdbcConnection driverClass="${jdbc.driverClass}"
                            connectionURL="${jdbc.url}"
                            userId="${jdbc.user}"
                            password="${jdbc.password}">
            </jdbcConnection>

            <!-- 配置 实体类 生成位置 -->
            <javaModelGenerator targetPackage="com.isea533.mybatis.model" 
                                targetProject="src/main/java"/>

            <!-- 配置 mapper.xml 生成位置 -->
            <sqlMapGenerator targetPackage="mapper" 
                            targetProject="src/main/resources"/>

            <!-- 配置 Mapper接口 生成位置 -->
            <javaClientGenerator targetPackage="com.isea533.mybatis.mapper" 
                                targetProject="src/main/java"
                                type="XMLMAPPER"/>

            <!-- 配置数据库表 -->
            <table tableName="user_info">
                <generatedKey column="id" sqlStatement="JDBC"/>
            </table>
        </context>
    </generatorConfiguration>
    ```

4. 代码生成命令
    `mvn mybatis-generator:generate` 

5. 启用注解 `@tk.mybatis.spring.annotation.MapperScan()`