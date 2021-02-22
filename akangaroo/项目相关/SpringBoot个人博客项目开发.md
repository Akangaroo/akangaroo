# SpringBoot个人博客项目开发

## 1. 环境的配置以及测试

1. 依赖导入
   - devtools：热部署
   - thymeleft：模板引擎
   - validation：JSR303校验
   - aop
   - web
   - mysql

- 导入静态页面

- application.yml配置文件编写

  ```yaml
  #applicaition.yml
  spring:
    # 模板引擎缓存关闭
    thymeleaf:
      cache: false
  
    profiles:
      active: dev
  
  ```

  ```yaml
  # 2. applicaition-dev.yml
  # DataSource Config
  spring:
    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/myblog?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
      username: root
      password: love22
  
  # 日志配置
  logging:
    level:
      root: info
      cn.edu: debug
    file:
      name: log/blog-dev.log
  ```

  ```yaml
  # 3. applicaition-prod.yml
  # DataSource Config
  spring:
    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/MyBlog?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
      username: root
      password: love22
  
  server:
    port: 80
  
  # 日志配置
  logging:
    level:
      root: warn
      cn.edu: debug
    file:
      name: log/blog-prod.log
  ```

- 测试是否能连接数据库，以及能否使用前端页面

  ```java
  @SpringBootTest
  class BlogApplicationTests {
  
      @Autowired
      DataSource dataSource;
  
      @Test
      void contextLoads() throws SQLException {
          System.out.println(dataSource.getClass());
          Connection connection = dataSource.getConnection();
          System.out.println(connection);
          connection.close();
      }
  
  }
  ```

  ```java
  @Controller
  public class TestController {
      @RequestMapping("/hello")
      @ResponseBody
      public String hello(){
          return "hello";
      }
  }
  ```

## 2. 异常处理

1. 异常界面

   放在error目录下

2. 自定义异常处理器

   ```java
   @ControllerAdvice
   public class ControllerExceptionHandler {
   
       private final Logger logger = LoggerFactory.getLogger(this.getClass());
   
       @ExceptionHandler(Exception.class)
       public ModelAndView exceptionHandler(HttpServletRequest request,Exception exception) throws Exception {
           logger.error("Request URL : {} , Exception : {}",request.getRequestURL(),exception);
   
           ModelAndView mv = new ModelAndView();
           mv.addObject("url",request.getRequestURL());
           mv.addObject("exception",exception);
           mv.setViewName("error/error");
           return mv;
       }
   }
   ```

3. 自定义异常

   ```java
   @ResponseStatus(HttpStatus.NOT_FOUND)
   public class NotFoundException extends RuntimeException{
   
       public NotFoundException() {
       }
   
       public NotFoundException(String message) {
           super(message);
       }
   
       public NotFoundException(String message, Throwable cause) {
           super(message, cause);
       }
   }
   ```

   因为异常处理器中拦截了所有异常，所以需要进行判断。

   

## 3. 日志处理

1. 记录日志的内容
   - 请求的url
   - 访问者ip
   - 调用方法classMethod
   - 参数args
   - 返回内容

```java
@Aspect
@Component
public class LogAspect {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Pointcut("execution(* cn.edu.blog.controller.*.*(..))")
    public void log(){
    }

    @Before("log()")
    public void doBefore(JoinPoint joinPoint){
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        String url = request.getRequestURL().toString();
        String ip = request.getRemoteAddr();
        String classMethod = joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        ResultLog resultLog = new ResultLog(url, ip, classMethod, args);
        logger.info("Request : "+resultLog);
    }

    @After("log()")
    public void doAfter(){
        // logger.info("---------doAfter---------");
    }

    @AfterReturning(returning = "result",pointcut = "log()")
    public void doAfterReturning(Object result){
        logger.info("Result : {}"+result);
    }

    private class ResultLog {
        private String url;
        private String ip;
        private String classMethod;
        private Object[] args;

        public ResultLog(String url, String ip, String classMethod, Object[] args) {
            this.url = url;
            this.ip = ip;
            this.classMethod = classMethod;
            this.args = args;
        }

        @Override
        public String toString() {
            return "ResultLog{" +
                    "url='" + url + '\'' +
                    ", ip='" + ip + '\'' +
                    ", classMethod='" + classMethod + '\'' +
                    ", args=" + Arrays.toString(args) +
                    '}';
        }
    }
}
```

## 4. 页面处理

- _fragment：提取公共部分
- 错误页面的优化

## 5. 设计表

- 实体类：

  - 博客：id,title,content,description,picture,flag(转载or原创),views,createTime,updateTime
- 分类：id,name
  
  - 评论：id,nickname,email,avatar,comment,createTime
- 用户：id,username,password,avatar,role
  
- 实体类之间的关系：

  - 博客与标签：一个博客对应一个分类，一个分类可以对应多个博客
  - 博客与评论：一个博客对应多条评论
  - 博客与用户：一个用户对应多个博客
  - 评论之间（）：一个评论对应多条回复，多条回复对应一条评论

- 建表语句
  
  - 表建立的规则：1:n的外键放在n的一端，n:n的外键需要建立中间表

## 6. 后台管理

- 登录成功（重定向，防止表单重复提交）

1. 构建登录界面和后台管理界面

2. UserService和UserMapper

   ```java
   public interface UserService {
   
       public User checkUser(String name,String password);
   
   }
   ```

   ```java
   @Service
   public class UserServiceImpl implements UserService {
   
       @Autowired
       UserMapper userMapper;
   
       @Override
       public User checkUser(String username, String password) {
           return userMapper.checkUser(username,MD5Utils.code(password));//MD5加密，看下方
       }
   
   }
   ```

   

3. LoginController实现登录

   ```java
   @Controller
   @RequestMapping("/admin")
   public class LoginController {
   
       @Autowired
       UserService userService;
   
       /**
        * 来到登录页面
        * @return
        */
       @GetMapping
       public String loginPage(){
           return "admin/login";
       }
   
       /**
        * 登录成功重定向到博客管理页面
        * @param username
        * @param password
        * @return
        */
       @PostMapping("/login")
       public String login(@RequestParam("username") String username,
                           @RequestParam("password") String password,
                           HttpSession session,
                           RedirectAttributes attributes){
           User user = userService.checkUser(username, password);
           if(user != null){
               user.setPassword(null);
               session.setAttribute("user",user);
               return "redirect:/admin/blogs";
           } else {
               attributes.addFlashAttribute("message","用户名和密码有误！");
               return "redirect:/admin";
           }
       }
   }
   ```

4. MD5加密

   ```java
   public class MD5Utils {
   
       public static String code(String password) {
           try {
               // 得到一个信息摘要器
               MessageDigest digest = MessageDigest.getInstance("md5");
               byte[] result = digest.digest(password.getBytes());
               StringBuffer buffer = new StringBuffer();
               // 把每一个byte 做一个与运算 0xff;
               for (byte b : result) {
                   // 与运算
                   int number = b & 0xff;// 加盐
                   String str = Integer.toHexString(number);
                   if (str.length() == 1) {
                       buffer.append("0");
                   }
                   buffer.append(str);
               }
               // 标准的md5加密后的结果
               return buffer.toString();
           } catch (NoSuchAlgorithmException e) {
               e.printStackTrace();
               return "";
           }
       }
   
       public static void main(String[] args) {
           System.out.println(MD5Utils.code("love22"));
       }
   }
   ```

   - 将得到的密码放入数据库中

5. 登录拦截器

   ```java
   public class LoginInterceptor implements HandlerInterceptor {
   
       @Override
       public boolean preHandle(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler) throws Exception {
           if(request.getSession().getAttribute("user") == null){
               response.sendRedirect("/admin");
               return false;
           }
   
           return true;
       }
   }
   ```

   ```java
   @Configuration
   public class MyMvcConfig implements WebMvcConfigurer {
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(new LoginInterceptor())
                   .addPathPatterns("/admin/**")
                   .excludePathPatterns("/admin")
                   .excludePathPatterns("/admin/login");
       }
       
   }
   ```

   

## 7. 分类管理

1. 分类页面

2. 定义url跳转到标签页（先从数据库中查数据，分页）

   ```java
       /**
        * 来到标签列表页面
        * @param pageNum
        * @param model
        * @return
        */
       @GetMapping("/types")
       public String toTypes(@RequestParam(defaultValue = "1",value = "pageNum") Integer pageNum, Model model){
           PageHelper.startPage(pageNum,5,"id desc");
           List<Type> types = typeService.getAllType();
           PageInfo<Type> pageInfo = new PageInfo<>(types);
           model.addAttribute("pageInfo",pageInfo);
           return "admin/types";
       }
   ```
   
3. 定义url跳转到标签新增页

   ```java
       /**
        * 来到标签添加页面
        * @param model
        * @return
        */
       @GetMapping("/types/input")
       public String toTypesInput(Model model){
           model.addAttribute("type",new Type());
           return "admin/types-input";
       }
   ```

4. 标签新增页面处理，提交form表单，重复处理

   1. 后端非空校验

       ```java
           @NotEmpty(message = "分类名称不能为空")
           private String name;
       ```

   2. 数据重复校验

      ```java
          /**
           * 标签保存提交
           * @param type
           * @param result
           * @param attributes
           * @return
           */
          @PostMapping("/types")
          public String typesPost(@Valid Type type, BindingResult result, RedirectAttributes attributes){
              //先从数据库中查询是否有重复的
              Type typeByName = typeService.getTypeByName(type.getName());
              if(typeByName != null){
                  result.rejectValue("name","nameError","已经存在该分类");
              }
              if(result.hasErrors()){
                  return "admin/types-input";
              }
      
              //保存数据
              int t = typeService.saveType(type);
              if(t == 0){
                  attributes.addFlashAttribute("message","新增失败");
              } else {
                  attributes.addFlashAttribute("message","新增成功");
              }
              return "redirect:/admin/types";
          }
      ```

   3. html页面

      1. 添加th:object="${type}"

      2. th:value="*{name}"

      3. 在controller中添加

         ```java
             /**
              * 来到标签修改页面
              * @param id
              * @param model
              * @return
              */
             @GetMapping("/types/input/{id}")
             public String toTypesEdit(@PathVariable("id") Integer id, Model model){
                 model.addAttribute("type", typeService.getTypeById(id));
                 return "admin/types-input";
       }
         ```
      
         
      
      ```html
      <form action="#" th:action="@{/admin/types}" th:object="${type}"  method="post" class="ui form">
                      <input type="hidden" name="id">
                      <div class="required field">
                          <div class="ui left labeled input">
                              <label class="ui teal basic label">名称</label>
                              <input type="text" name="name" placeholder="分类名称" th:value="*{name}">
                          </div>
                      </div>
      
      
                      <div class="ui error message"></div>
      
                      <!--/*/
                          <div class="ui negative message" th:if="${#fields.hasErrors('name')}">
                              <i class="close icon"></i>
                              <div class="header">验证失败</div>
                              <p th:errors="*{name}">提交信息不符合规则</p>
                          </div>
                      /*/-->
   </form>
      ```
      
      

5. 标签编辑和删除

   1. 来到标签编辑页面，和标签新增页面共用

      ```java
      /**
           * 标签修改提交
           * @param type
           * @param result
           * @param attributes
           * @return
           */
          @PostMapping("/types/{id}")
          public String typesEdit(@Valid Type type, BindingResult result, RedirectAttributes attributes){
              //先从数据库中查询是否有重复的
              Type typeByName = typeService.getTypeByName(type.getName());
              if(typeByName != null){
                  result.rejectValue("name","nameError","已经存在该标签");
              }
              if(result.hasErrors()){
                  return "admin/types-input";
              }
      
              //保存数据
              int t = typeService.updateTypeById(type);
              if(t == 0){
                  attributes.addFlashAttribute("message","更新失败");
              } else {
                  attributes.addFlashAttribute("message","更新成功");
              }
              return "redirect:/admin/types";
          }
      
          /**
           * 标签删除
           * @param id
           * @param attributes
           * @return
           */
          @GetMapping("/types/delete/{id}")
          public String typesDelete(@PathVariable("id") Integer id,RedirectAttributes attributes){
              typeService.deleteType(id);
              attributes.addFlashAttribute("message","删除成功！");
              return "redirect:/admin/types";
          }
      ```

      在tags-input页面中添加

      ```html
      <form action="#"th:object="${tag}" th:action="*{id}==null ? @{/admin/tags} : @{/admin/tags/id(id=*{id})}"   method="post" class="ui form">
      ```

      

## 8. 博客管理

1. 博客分页查询

   1. 先确定需要查询的数据，涉及到多表查询，选择将查询的数据封装到一个对象中，结果集返回这个对象。

      - 应该注意的是，我们当时的登录界面是转发到博客列表，此时应该改成重定向。

      - 使用pageHelper封装结果集，编写sql语句

        ```java
            private Integer id;
            private String title;
            private Tag tag;
            private boolean published;
            private Date updateTime;
        ```

        ```java
        @RequestMapping("/blogs")
            public String  toBlog(@RequestParam(value = "pageNum",defaultValue = "1") Integer pageNum, Model model){
        
                PageHelper.startPage(pageNum,5,"id desc");
                List<BlogQuery> blogs = blogService.getAllBlogQuery();
                PageInfo<BlogQuery> pageInfo = new PageInfo<>(blogs);
        
                List<Tag> tags = tagService.getAllTag();
                model.addAttribute("tags",tags);
                model.addAttribute("pageInfo",pageInfo);
                return "admin/blogs";
        
            }
        ```

        ```xml
            <resultMap id="queryBlog" type="cn.edu.blog.pojo.BlogQuery">
                <id property="id" column="id"></id>
                <result property="title" column="title"></result>
                <result property="published" column="published"></result>
                <result property="updateTime" column="update_time"></result>
        
                <association property="tag" javaType="cn.edu.blog.pojo.Tag">
                    <id column="id" property="id"></id>
                    <result column="name" property="name"></result>
                </association>
            </resultMap>
        
            <!--public List<Blog> getAllBlogQuery();-->
            <select id="getAllBlogQuery" resultMap="queryBlog">
                SELECT b.id,b.title,b.published,b.update_time,t.`name` FROM t_blog b
                LEFT JOIN t_tag t
                ON b.tag_id = t.id
            </select>
        ```

2. 实现查询功能

   1. 使用ajax

      ```javascript
      $("#search-btn").click(function () {
          loaddata();
      });
      
      
      function loaddata() {
          $("#table-container").load(/*[[@{/admin/blogs/search}]]*/"/admin/blogs/search",{
          title:$("[name='title']").val(),
          tagId:$("[name='tagId']").val(),
          page:$("[name='page']").val()
          });
      }
      ```

   2. controller，页面上加上blogList

      ```java
          @PostMapping("/blogs/search")
          public String search(BlogQuery blogQuery,@RequestParam(value = "pageNum",defaultValue = "1") Integer pageNum,
                               Model model){
      
              PageHelper.startPage(pageNum,5,"id desc");
              List<BlogQuery> blogs = blogService.getBlogSearch(blogQuery);
              System.out.println(blogs);
              PageInfo<BlogQuery> pageInfo = new PageInfo<>(blogs);
              System.out.println(blogs);
              model.addAttribute("pageInfo",pageInfo);
              return "admin/blogs :: blogList";
          }
      ```

   3. sql语句

      ```xml
          <!--public List<BlogQuery> getBlogSearch();-->
          <select id="getBlogSearch" resultMap="queryBlog">
      
              SELECT b.id,b.title,b.published,b.update_time,t.`name` FROM t_blog b
              LEFT JOIN t_tag t
              on b.tag_id = t.id
              <where>
                  <if test="tagId != null">
                      b.tag_id = #{tagId}
                  </if>
                  <if test="title != null">
                      <bind name="likeTitle" value=" '%' + title + '%' "/>
                      and b.title like #{likeTitle}
                  </if>
              </where>
          </select>
      ```

      

3. 博客的新增

    ```java
   @PostMapping("/blogs")
       public String BlogPost(Blog blog, RedirectAttributes attributes, HttpSession session){
           blog.setUser((User) session.getAttribute("user"));
           blog.setTag(tagService.getTagById(blog.getTag().getId()));
           int i = blogService.saveBlog(blog);
           if(i == 0){
               attributes.addFlashAttribute("message","新增博客成功！");
           } else {
               attributes.addFlashAttribute("message","新增博客失败！");
           }
   
           return "redirect:/admin/blogs";
       }
   ```

   ```java
   <!--public int saveBlog(Blog blog);-->
       <insert id="saveBlog">
           insert into t_blog
           (title,content,description,picture,flag,views,published,create_time,update_time,user_id,tag_id)
           values (#{title},#{content},#{description},#{picture},#{flag},#{views},#{published},#{createTime}
           ,#{updateTime},#{user.id},#{tag.id})
       </insert>
   ```

   

4. 博客的修改

   1. Controller

      ```java
      @GetMapping("/blogs/input/{id}")
          public String BlogEdit(@PathVariable("id") Integer id,Model model){
              model.addAttribute("blog",blogService.getBlogById(id));
              model.addAttribute("tags",tagService.getAllTag());
              return "admin/blogs-input";
          }
      ```

      ```xml
          <!--public Blog getBlogById(Integer id);-->
          <select id="getBlogById" resultType="cn.edu.blog.pojo.Blog">
              select * from t_blog where id = #{id};
          </select>
      ```

      ```html
      
      ```

5. 博客的删除

6. 博客的测试

   1. 发现分页功能出现了一些问题，搜索之后不能分页

      修改前端代码

      添加隐含域name值设置为pageNum，实现分页功能。因为后台pageHelper需要的属性是pageNum

      将分页的代码改成如下形式，即给他自定义属性pageNum

      ```html
      <a onclick="page(this)" th:attr="data-pageNum=${pageInfo.hasPreviousPage}?${pageInfo.prePage}:1" th:unless="${pageInfo.isFirstPage}" class="item">
                                              上一页
                                          </a>
                                          <a onclick="page(this)" th:attr="data-pageNum=${pageInfo.hasNextPage}?${pageInfo.nextPage}:${pageInfo.pages}" th:unless="${pageInfo.isLastPage}" class="item">
                                              下一页
                                          </a>
      ```

      js中的代码

      search-btn意味着点击分页按钮之后查询到第一页的数据，然后加载数据。

      page()函数意味着将当前的pageNum的属性值设置为当前的下一页的页码，可以参照f12，然后加载数据。

      坑：pagenum是小写，不是我们设置的pageNum。

      ```javascript
              $("#search-btn").click(function () {
                  $("[name='pageNum']").val(1);
                  loaddata();
              });
      
              function page(obj) {
                  $("[name='pageNum']").val($(obj).data("pagenum"));
                  loaddata();
              }
      
              function loaddata() {
                  $("#table-container").load(/*[[@{/admin/blogs/search}]]*/"/admin/blogs/search",{
                      title:$("[name='title']").val(),
                      tagId:$("[name='tagId']").val(),
                      pageNum:$("[name='pageNum']").val()
                  });
              }
      ```

      

## 10. 前端页面

1. 博客列表
2. top分类
3. top标签
4. 最新博客推荐
5. 博客详情
6. 搜索功能

## 11. 评论功能

1. 添加隐含域blog.id,parentComment.id

## 项目部署

nohup java -jar blog1.1.jar > blog-log.txt 2>&1 &



A、Logger.log为该日志存放的文件名称名+后缀名(不存在的时候将自动创建)
      B、2>&1 输出所有的日志文件(包括错误信息的文件)
      C、& 后台启动