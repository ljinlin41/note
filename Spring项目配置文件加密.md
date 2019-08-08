1. 导入依赖  
    ```
    <dependency>
        <groupId>com.github.ulisesbocchio</groupId>
        <artifactId>jasypt-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
    ```
2. 代码示例
    ```java
    package cn.ljlin233.hsjryredisdemo;
    
    import org.jasypt.encryption.StringEncryptor;
    import org.jasypt.util.text.BasicTextEncryptor;
    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import lombok.extern.slf4j.Slf4j;
    
    /**
     * @author lvjinlin@yunrong.cn
     * @version V2.1
     * @date 2019/7/29 14:33
     * @since 2.1.0
     */
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @Slf4j
    public class JasyptTest {
    
        private BasicTextEncryptor encryptor = new BasicTextEncryptor();
    
        private String depassword;
    
        @Before
        public void setSaltKey() {
            String salt = "Oat4jkamo$dK";
            log.info("加盐: " + salt);
            encryptor.setPassword(salt);
        }
    
        @Test
        public void encrypt() {
            String password = "123456";
            log.info("<=========  开始加密  ========>");
            log.info("密码明文: " + password);
            this.depassword = encryptor.encrypt(password);
            log.info("加密后的密码: " + depassword);
            log.info("<=========  加密成功  ========>");
        }
    
        @Test
        public void decrypt() {
            this.depassword = "j43LGrseBYHm4h/FQlptJA==";
            log.info("<=========  开始解密  ========>");
            log.info("未解密的密码：" + this.depassword);
            String getPassword = encryptor.decrypt(this.depassword);
            log.info("解密后的密码: " + getPassword);
            log.info("<=========  解密成功 ========>");
        }
    
    }

    ```