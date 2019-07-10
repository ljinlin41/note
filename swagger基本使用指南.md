1. Maven依赖  

    ```
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.6.1</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.6.1</version>
    </dependency>
    ```
2. SpringBoot配置  
    1. 新建Swagger2.java类，请确保放在能被SpringBoot扫描到的位置  
    
    ```
    @Configuration
    @EnableSwagger2
    public class Swagger2 {
    
        @Bean
        public Docket createRestApi() {
            return new Docket(DocumentationType.SWAGGER_2).apiInfo(this.apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("controller包"))
                .paths(PathSelectors.any())
                .build();
        }
    
        private ApiInfo apiInfo() {
            return new ApiInfoBuilder().title("springboot利用swagger构建api文档")
                .description("项目描述，代码地址 https://github.com/ljinlin41")
                .termsOfServiceUrl("https://github.com/ljinlin41")
                .version("1.0")
                .build();
        }
    
    }
    ```
3. API Web地址    
    1. 此时启动SpringBoot项目，浏览<http://localhost:8080/swagger-ui.html>，可以发现已经有了项目的API信息。但这些信息是swagger自动配置的，如果需要定制更详细的信息，可以使用注解。  
      
4. 使用注解    
    1. @Api  
        1. 功能:描述controller类  
        2. 注解位置:类  
        3. 常用注解属性    
            1. tags = "" // 描述此controller类  
    2. @ApiOperation  
        1. 功能:描述一个方法或者一个API接口  
        2. 注解位置:方法  
        3. 常用注解属性  
            1. value = "" // 描述方法  
            2. notes = "" // 描述方法详细信息  
    3. @ApiImplicitParam  
        1. 功能:描述方法或接口参数  
        2. 注解位置: 方法  
        3. 注解属性  
            1. name = "" // 方法或接口的形参, 注意要与方法的参数名称相同  
            2. value = "" // 对参数的描述  
            3. paramType = "" // 参数传递方式，此属性的可选值 ["header", "query", "path", "body", "form"]  
                1. header，使用@RequestHeader获取的参数  
                2. query，使用@RequestParam获取的参数，常用于GET请求  
                3. path，使用@PathVariable获取的参数  
                4. body，使用@RequestBody获取的参数，常用于POST请求，对象参数  
            4. dataType = "" // 参数类型，例如 string, int, ArrayList, POJO类  
    4. @ApiImplicitParams  
        1. 功能:汇集多个参数  
        2. 注解位置: 方法  
        3. 注解属性  
            1. @ApiImplicitParam组成的列表  
            
5. 项目代码地址  
    <https://github.com/ljinlin41/demo-swagger>
    