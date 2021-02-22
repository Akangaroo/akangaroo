# Swagger

## 作用

1. 接口的文档在线自动生成。
2. 功能测试。

## 依赖

```xml
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
     </dependency>
```

## 配置

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    /**
     * apiInfo，增加API相关的信息
     * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现
     * @return
     */
    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()// 选择那些路径和api会生成document
                //配置要扫描接口的方式
                //basePackage("cn.edu.springboot")
                //any();任何
                //none();不扫描
                //withClassAnnotation
                //withMethodAnnotation
                .apis(RequestHandlerSelectors.basePackage("cn.edu.springboot"))
                //对所有路径进行监控
                .paths(PathSelectors.any())
                .build();
    }

    //配置Swagger信息 = apiInfo
    private ApiInfo apiInfo(){
        //作者信息
        Contact contact = new Contact("AKangaroo", "https://www.baidu.com", "Kangaroo_yw@163.com");

        return new ApiInfo("Spring Boot中使用Swagger2构建RESTful APIs",
                "更多请关注http://www.baidu.com",
                "1.0",
                "https://www.baidu.com",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList<VendorExtension>());

    }
}
```

- 访问http://localhost:8080/swagger-ui.html。

## 其它功能

- 生产环境使用，发布的时候不使用Swagger。

```java
//设置要显示的swagger环境
Profiles profiles = Profiles.of("dev");
//获取项目的环境
//判断是否处在环境中
boolean flag = environment.acceptsProfiles(profiles);

enable(flag);
```

- 分组

  配置多个Docket

  ```java
  return new Docket(DocumentationType.SWAGGER_2).groupName("hehe");
  ```

## Controller

- 遇到的坑：
  - dateType="Integer"没有效果
  - 2.9.2版本的bug

```java
@Controller
@Api("用来测试APi注解的控制器")
public class HelloController {


    @ResponseBody
    @RequestMapping(value = "/test/{id}",method = RequestMethod.GET)
    @ApiOperation(value = "根据id返回信息",notes = "test：根据id返回信息")
    @ApiImplicitParam(paramType = "path",name = "id", value = "用户编号", required = true, dataType = "int")
    public String test(@PathVariable("id") Integer id){
        if(id == 1){
            return "hehe";
        } else{
            return null;
        }
    }

    @ResponseBody
    @RequestMapping(value = "/test02",method = RequestMethod.GET)
    @ApiOperation(value = "根据id获得密码",notes = "")
    @ApiImplicitParams({@ApiImplicitParam(name = "id",
                                            value = "用户编号",
                                            required = true,
                                            dataType = "int"),
                        @ApiImplicitParam(name = "password"
                                            ,value = "用户密码"
                                            ,required = true
                                            ,dataType = "String")
            })
    public String test02(@RequestParam Integer id, @RequestParam String password){
        if(id == 1 && !StringUtils.isEmpty(password)){
            return "hello";
        } else{
            return null;
        }
    }
}
```

