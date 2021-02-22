# Spring

## 简介

Spring是容器（可以管理所有的组件（类））框架

> 什么是容器(Container): 从程序设计角度看就是封装对象的对象,因为存在放入、拿出等操作,所以容器还要管理对象的生命周期,如Tomcat就是Servlet和JSP的容器;

为什么说Spring是一个一站式的轻量级开源框架呢？

​	轻量级：不是指Spring框架的模块少，数量很轻，而是==指Spring框架的非侵入性，即对象可以不必依赖Spring的API类==

Spring的优点
- 非侵入式：可以不依赖Spring的API
- **依赖注入：DI------控制反转(IOC)最经典的实现**
- **面向切面编程：AOP**
- 容器：Spring是一个容器
- 组件化：可以使用XML和Java注解组合一些对象
- 一站式：SpringMVC和Spring JdbcTemplate

Spring的体系结构图

![img](Spring-IOC.assets/480452-20190318225849216-2097896352.png)

核心容器
- **spring-core**：提供了框架的基本组成部分，包括 IOC 和依赖注入功能。

- **spring-beans**：提供 BeanFactory，工厂模式的微妙实现，它移除了编码式单例的需要，并且可以把配置和依赖从实际编码逻辑中解耦。

- **spring-context**：模块建立在由core和 beans 模块的基础上建立起来的，它以一种类似于JNDI注册的方式访问对象。Context模块继承自Bean模块，并且添加了国际化（比如，使用资源束）、事件传播、资源加载和透明地创建上下文（比如，通过Servelet容器）等功能

- **spring-expression**：提供了强大的表达式语言，用于在运行时查询和操作对象图。

  

## IOC和DI

### IOC(**Inversion of Control**)

IOC—Inversion of Control，即”控制反转”，不是什么技术，而是一种设计思想。在Java开发中，IOC意味着将**创建对象的权力交给容器控制，而不是传统的在你的对象内部直接控制**。

IOC是有专门一个容器来创建这些对象，即由IOC容器来控制对象的创建而不再显式地使用new；谁控制谁？当然是**IOC容器控制了对象**；控制什么？那就是**主要控制了外部资源获取和生命周期**（不只是对象也包括文件等）。

### DI(**Dependency Injection**)

IOC的另一种表述方式：即组件以一些预先定义好的方式(例如：setter 方法)接受来自于容器的资源注入。相对于IOC而言，这种表述更直接。

### IOC容器在Spring中的实现

1. 在通过IOC容器读取Bean的实例之前，需要先将IOC容器本身实例化。
2. **Spring IOC容器**
    - ***BeanFactory***: Spring最底层的接口, 只提供了IoC的功能, 负责创建、组装、管理bean，所以一般不适用BeanFactory, 推荐使用ApplicationContext(应用上下文)
    - ***ApplicationContext*** : ApplicationContext接口继承了BeanFactory, 还提供了AOP继承、国际化处理、事件传播、统一资源价值等功能; 可以查看该接口的继承体系;
3. BeanFactory和ApplicationContext之间的区别：
    - BeanFactory有**延迟初始化**（懒加载）的特点,在创建Spring容器的时候,不会立刻去创建容器,管理Bean对象,而是要等到从容器中获取对象的时候,才去创建对象.
    - ApplicationContext在创建Spring容器的时候,会把容器中管理的Bean**立刻初始化**,而不会等到获取Bean的时候才初始化

## 给Bean的属性赋值

### Bean赋值的途径

#### 构造器注入

标签：constructor-arg

* 标签出现的位置：bean标签的内部
* 标签中的属性：
  1. type：用于指定要注入的数据的数据类型。

  2. index：指定索引位置的参数赋值

  3. name：用于给构造函数指定名称的参数赋值  

  4. value：

  5. ref：

```xml
<!--index-->
<bean id="person" class="cn.akangaroo.pojo.Person">
    <constructor-arg index="0" value= "kyw" />
    <constructor-arg index="1" value= "18" />
</bean >

<!--index && type-->
<bean id="person" class="cn.akangaroo.pojo.Person">
    <constructor-arg value= "10010" index ="0" type="java.lang.Integer" />
    <constructor-arg value= "Book01" index ="1" type="java.lang.String" />
    <constructor-arg value= "Author01" index ="2" type="java.lang.String" />
    <constructor-arg value= "20.2" index ="3" type="java.lang.Double" />
</bean >

<!--name-->
<bean id="person" class="cn.akangaroo.pojo.Person">
  <constructor-arg name="name" value="kyw"></constructor-arg>
  <constructor-arg name="age" value="18"></constructor-arg>
  <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>

<!--配置一个日期对象-->
<bean id="now" class="java.util.Date"></bean>
```

#### set方法注入

标签：property

* 标签出现的位置：bean标签的内部
* 标签的属性：
  * name：用于指定时所调用的方法名称  
  * value：用于提供基本类型和String类型的参数
  * ref：用于指定其他的bean类型的数据。

```xml
<bean id="person" class="cn.akangaroo.pojo.Person">
  <property name="age" value="21"></property>
  <property name="birthday" ref="now"></property>
  <property name="name" value="kyw"></property>
</bean>
    
<!--配置一个日期对象-->
<bean id="now" class="java.util.Date"></bean>
```

### 复杂类型/集合类型的注入

1. 用于给List结构集合标签注入的标签有：
   * list
   * array
   * set
2. 用于给Map结构集合注入的标签有：
   * map
   * props

**结构相同标签可以互换**

```xml
<bean id="accountService" class="cn.akangaroo.service.Impl.AccountServiceImpl">
    <property name="myArray">
        <array>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </array>
    </property>

    <property name="myList">
        <list>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </list>
    </property>

    <property name="mySet">
        <set>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </set>
    </property>

    <property name="myMap">
        <map>
            <entry key="testA" value="aaa"></entry>
            <entry key="testB">
                <value>bbb</value>
            </entry>
        </map>
    </property>

    <property name="myProps">
        <props>
            <prop key="testC">Ccc</prop>
            <prop key="testD">Ddd</prop>
        </props>
    </property>
</bean>
```

### 注入可以使用的值

#### null值

```xml
<bean class="cn.akangaroo.Book" id="bookNull" >
           <property name= "bookId" value ="2000"/>
           <property name= "bookName">
               <null/>
           </property>
           <property name= "author" value ="nullAuthor"/>
           <property name= "price" value ="50"/>
     </bean >

```

#### 内部bean

注意：容器无法获取到内部的bean

```java
<bean id="shop" class="cn.akangaroo.Shop" >
    <property name= "book">
        <bean class= "cn.akangaroo.Book" >
           <property name= "bookId" value ="1000"/>
           <property name= "bookName" value="innerBook" />
           <property name= "author" value="innerAuthor" />
           <property name= "price" value ="50"/>
        </bean>
    </property>
</bean >
```

## Spring对bean的管理

### Bean实例化的方式

#### 无参构造器(常用)

在spring的配置文件中使用bean标签以后，配以id和class属性之后，且没有其它的属性和标签时，采用的就是默认构造函数创建，如果没有默认构造函数，创建失败。

```java
//类
public class Hero {
    public Hero(){
        System.out.println("构建Hero");
    }
}

```

```xml
<bean id="hero" class="cn.akangaroo.Hero"></bean>
```



#### 静态工厂(了解)

```java
//类
public class Hero{

}
//工厂
public class HeroFactory {
    public static Hero createInstance(){
        Hero h = new Hero();
        return h;
    }
}

```

```xml
<bean id="hero" class="cn.akangaroo.factory.HeroFactory" factory-method="createInstance"></bean>
```



#### 实例工厂(了解)

```java
// 类
public class Hero {
}
// Cat3工厂
public class HeroFactory {

    public Hero createInstance(){
        Hero h = new Hero();
        return h;
    }
}

```

 ```xml
<!--
        第一个bean是创建实例工厂对象,第二个bean是通过实例工厂对象调用工厂方法创建Hero的对象
-->
<bean id="heroFactory" class="cn.akangaroo.factory.HeroFactory"/>
<bean id="hero" factory-bean="heroFactory" factory-method="createInstance"/>
 ```



### Bean对象的作用范围

|      类别      | 说明                                                         |
| :------------: | :----------------------------------------------------------- |
| **singleton**  | 单例，默认值                                                 |
| **prototype**  | 多例，每次从容器中调用Bean时,都返回一个新的实例              |
|    request     | 作用于web应用的请求范围                                      |
|    session     | 作用于web应用的会话范围                                      |
| global-session | 作用于集群环境的会话范围（全局会话范围），当不是集群环境时，他就是session |

### Bean对象的生命周期

单例对象：
* 出生：容器创建时对象出生
* 活着：容器一直在，对象就活着
* 死亡：容器销毁，对象消失

**和容器的生命周期息息相关**

```java
public class AccountServiceImpl implements AccountService {

    public AccountServiceImpl() {
        System.out.println("AccountService对象创建了！");
    }

    public void saveAccount() {
        System.out.println("Service中的代码执行了！");
    }

    public void init(){
        System.out.println("初始化");
    }

    public void destroy(){
        System.out.println("销毁");
    }
}

```

```xml
<bean id="accountService" class="cn.akangaroo.service.Impl.AccountServiceImpl" scope="singleton"
      init-method="init" destroy-method="destroy"></bean>
```

```java
/**
* 测试
*/
public class Client {
    public static void main(String[] args) {
        //获取核心容器对象
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");

        AccountService as = context.getBean("accountService", AccountServiceImpl.class);

        as.saveAccount();

        //手动关闭容器
        context.close();
    }
}
```



多例对象：

* 出生：当我们使用Spring框架创建对象
* 活着：对象只要一直使用过程中就一直活着
* 死亡；当对象长时间不用且没有别的对象引用时，由Java的垃圾回收机制回收。

### 配置信息的继承

如下的bean中存在重复的配置

```xml
<bean id="dept" class="cn.akangaroo.parent.bean.Department">
	<property name="deptId" value="100"/>
	<property name="deptName" value="IT"/>
</bean>

<bean id="emp01" class="cn.akangaroo.parent.bean.Employee">
	<property name="empId" value="1001"/>
	<property name="empName" value="Tom"/>
	<property name="age" value="20"/>

	<!-- 重复的属性值 -->	
	<property name="detp" ref="dept"/>
</bean>

<bean id="emp02" class="cn.akangaroo.parent.bean.Employee">
	<property name="empId" value="1002"/>
	<property name="empName" value="Jerry"/>
	<property name="age" value="25"/>

	<!-- 重复的属性值 -->
	<property name="detp" ref="dept"/>
</bean>
```



```xml
<!-- 以emp01作为父bean，继承后可以省略公共属性值的配置 -->
<bean id="emp02" parent="emp01">
	<property name="empId" value="1002"/>
	<property name="empName" value="Jerry"/>
	<property name="age" value="25"/>
</bean>

```



### 引用外部属性文件

这是直接配置数据库连接池

```xml
<!-- 直接配置 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="root"/>
	<property name="password" value="root"/>
	<property name="jdbcUrl" value="jdbc:mysql:///test"/>
	<property name="driverClass" value="com.mysql.jdbc.Driver"/>
</bean>

```

引用外部的数据

```properties
<!--外部属性文件-->
jdbc.username=root
jdbc.password=root
jdbc.url=jdbc:mysql:///test
jdbc.driverClass=com.mysql.jdbc.Driver
```

```xml
<context:property-placeholder location="classpath:dbconfig.properties"></context:property-placeholder>
<!-- 从properties属性文件中引入属性值 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
	<property name="jdbcUrl" value="${jdbc.url}"/>
	<property name="driverClass" value="${jdbc.driverClass}"/>
</bean>

```



## SpEL表达式

SpEL使用**#{…}**作为定界符，所有在大框号中的字符都将被认为是SpEL表达式

使用字面量

```xml
整数
<property name="count" value="#{5}"/>

小数
<property name="frequency" value="#{89.7}"/>

科学计数法
<property name="capacity" value="#{1e4}"/>

String
<property name="name" value="#{'Chuck'}"/>

Boolean
<property name="enabled" value="#{false}"/>
```

引用其它bean

```xml
<bean id="emp" class="cn.akangaroo.bean.Employee">
	<property name="empId" value="1003"/>
	<property name="empName" value="Kate"/>
	<property name="age" value="21"/>
	<property name="detp" value="#{dept}"/>
</bean>

```

引用其它bean的某个值作为属性的值

```xml
<bean id="emp05" class="cn.edu.bean.Employee">
	<property name="empId" value="1003"/>
	<property name="empName" value="Kate"/>
	<property name="age" value="21"/>
	<property name="deptName" value="#{dept.deptName}"/>
</bean>

```

调用静态方法

```java
<bean id="employee" class="cn.akangaroo.bean.Employee">
	<!-- 在SpEL表达式中调用类的静态方法 -->
	<property name="circle" value="#{T(java.lang.Math).PI*20}"/>
</bean>

```

调用非静态方法

```xml
<!-- 创建一个对象，在SpEL表达式中调用这个对象的方法 -->
<bean id="salaryGenerator" class="cn.akangaroo.bean.SalaryGenerator"/>

<bean id="employee" class="cn.akangaroo.bean.Employee">
	<!-- 通过对象方法的返回值为属性赋值 -->
	<property name="salayOfYear" value="#{salaryGenerator.getSalaryOfYear(5000)}"/>
</bean>

```



## 注解配置bean

### 注解标识组件

@Component：标识一个受Spring IOC容器管理的组件

@Respository：标识一个受Spring IOC容器管理的持久化层组件

@Service：标识一个受Spring IOC容器管理的业务逻辑层组件

@Controller：标识一个受Spring IOC容器管理的表述层控制器组件

组件命名规则：

- 默认情况：使用组件的简单类名首字母小写后得到的字符串作为bean的id
- 使用组件注解的value属性指定bean的id

### 扫描组件

```xml
<context:component-scan base-package="cn.akangaroo"/>
```

包含与排除

```xml
    <context:component-scan base-package="cn.akangaroo" >
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

```xml
    <context:component-scan base-package="cn.akangaroo.controller" use-default-filters="false">
        <!--只扫描控制器-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

`use-default-filters="false"`代表不使用默认的过滤器，默认的过滤器扫描component，service，controller，respository等注解标识的组件。

type中可以填的主要是

annotation：代表注解

assignable：过滤该类的子类

### 组件装配

**@Autowired**

- 作用：自动按照类型注入。只要容器中有**唯一的bean对象类型**和要注入的变量类型匹配，就可以注入成功。如果IOC容器中没有任何bean的类型和要注入的变量类型匹配，则报错。如果**有多个类型匹配时，先按照数据类型匹配，再按照变量名称匹配**。
- 位置：可以是成员变量上，也可以是方法上

**@Qualifier**

​	按照类型注入的基础之上，再按照名称注入。他在给类成员注入时不能单独使用（配合autowired），但是在给方法参	数注入时可以

**@Resource**

- 作用：直接按照bean的id注入。可以独立使用。
- 属性：name：用于指定bean的id

>以上三个注入都只能注入其它bean类型的数据，而基本类型和String类型无法使用上述注解实现。另外，集合类型的注入只能通过XML来实现。



## 相关注解

**@Configuration**

- 作用：指定当前类是一个配置类
- 细节：当配置类作为AnnotationConfigApplicationContext的参数时，此注解可以不写。

**@ComponentScan**

- 作用：用于通过注解指定spring在创建容器时要扫描的包
- 属性：
  - value：用于指定创建容器时要扫描的包，我们使用此注解就等于在XML中配置了`<context:component-scan base-package="cn.akangaroo"></context:component-scan>`
  - basePackages：和value的作用是一样的

**@Bean**

- 作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中。
- 属性：name：用于指定bean的id，当不写时，**默认值是当前方法的名称**。
- 细节：当我们使用注解配置时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。查找的方式和Autowired注解的作用是一样的。

**@Import**

- 作用：导入其它的配置类
- 属性：value：用于指定其它配置类的字节码，当我们使用Import的注解之后，有Import注解的类就是父配置类，导入的都是子配置类。

**@PropertySource**

- 作用：用于指定properties文件的位置
- 属性：value：指定文件的名称和路径。关键字：classpath，表示类路径下

**@scope**

- 作用：用于指定当前bean的范围是单例还是多例



