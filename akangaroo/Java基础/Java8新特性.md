# 1. Date Time API

|        API        |        介绍        |
| :---------------: | :----------------: |
|   LocalDateTime   |     日期和时间     |
|     LocalDate     |        日期        |
|     LocalTime     |        时间        |
|     Duration      | 两个时刻之间的差值 |
|      Period       | 两个日期之间的差值 |
|   ZonedDateTime   |  带时区的日期时间  |
|      Instant      |       时间戳       |
| DateTimeFormatter |    时间格式转换    |



## 1.1 LocalDateTime

- 总是表示本地日期和时间

### 1.1.1 获取当前的日期和时间

```java
	@Test
	public void test01() {
        LocalDate date = LocalDate.now();
    
        Month month = date.getMonth();
        int monthValue = date.getMonthValue();
        System.out.println(month+" and "+monthValue); // 输出结果为：NOVEMBER and 11
    
        System.out.println("当前日期：" + date);
        LocalTime time = LocalTime.now();
        System.out.println("当前时间：" + time);
        LocalDateTime dateTime = LocalDateTime.now();
        System.out.println("当前日期和时间：" + dateTime);
    }

运行结果：
NOVEMBER and 11
当前日期：2020-11-20
当前时间：15:25:08.759
当前日期和时间：2020-11-20T15:25:08.759
```

### 1.1.2 toLocalDate()等

```java
	@Test
	public void test02() {
        LocalDateTime now = LocalDateTime.now();
        // 转换到当前日期
        LocalDate date = now.toLocalDate();
        // 转换到当前时间
        LocalTime time = now.toLocalTime();
        System.out.println(date);
        System.out.println(time);
    }

运行结果：
2020-11-20
15:25:50.792
```

### 1.1.3 of()和parse()

- `of()`：通过指定的时间来创建LocalDateTime
- `parse()`：解析时严格按照`ISO 8601`规范，时间都是两位

```java
	@Test
    public void test03() {
        System.out.println("========通过of()方法和指定的日期和时间创建LocalDateTime=========");
        LocalDate date = LocalDate.of(2020, 12, 28);
        LocalTime time = LocalTime.of(10, 3, 33, 333);
        LocalDateTime dateTime = LocalDateTime.of(date, time);
        LocalDateTime dateTime02 = LocalDateTime.of(2020, 12, 28, 9, 21, 22, 222);
        System.out.println(dateTime);
        System.out.println(dateTime02);

        System.out.println("===========将字符串转换为LocalDateTime格式===========");
        LocalDateTime parseDateTime = LocalDateTime.parse("2020-12-28T04:05:06");
        LocalDate parseDate = LocalDate.parse("2020-12-28");
        LocalTime parseTime = LocalTime.parse("04:05:06");
        System.out.println(parseDateTime);
        System.out.println(parseDate);
        System.out.println(parseTime);
    }

运行结果：
========通过of()方法和指定的日期和时间创建LocalDateTime=========
2020-12-28T10:03:33.000000333
2020-12-28T09:21:22.000000222
===========将字符串转换为LocalDateTime格式===========
2020-12-28T04:05:06
2020-12-28
04:05:06
```

### 1.1.4 DateTimeFormatter

- `DateTimeFormatter`：可以自定义输出格式，或者将非`ISO 8601`格式的字符串解析成`LocalDateTime`

```java
	@Test
    public void test04() {
        System.out.println("自定义输出的格式");
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss:SSS");
        System.out.println(dtf.format(LocalDateTime.now()));

        System.out.println("将一个非ISO 8601格式的字符串解析成LocalDateTime");
        LocalDateTime time = LocalDateTime.parse("2020/12/28 12:28:33:333", dtf);
        System.out.println(time);

    }

运行结果：
自定义输出的格式
2020/11/20 15:33:01:637
将一个非ISO 8601格式的字符串解析成LocalDateTime
2020-12-28T12:28:33.333
```

### 1.1.5 时间的链式加减

```java
	@Test
    public void test05() {
        LocalDateTime dateTime = LocalDateTime.of(2020, 12, 28, 12, 28, 28);
        LocalDateTime dt1 = dateTime.plusDays(2).minusHours(2);
        LocalDateTime dt2 = dateTime.plusDays(4);
        System.out.println(dt1);
        System.out.println(dt2);
    }

运行结果：
2020-12-30T10:28:28
2021-01-01T12:28:28
```

### 1.1.6 调整日期和时间

- 调整年：`withYear()`
- 调整月：`withMonth()`
- 调整日：`withDayOfMonth()`
- 调整时：`withHour()`
- 调整分：`withMinute()`
- 调整秒：`withSecond()`

```java
    @Test
	public void test06() {
        LocalDateTime dateTime = LocalDateTime.of(2020, 12, 28, 12, 28, 28);
        LocalDateTime dateTime1 = dateTime.withHour(14).withDayOfMonth(29);
        System.out.println(dateTime1);
    }

运行结果：
2020-12-29T14:28:28
```

### 1.1.7 with()

```java
    @Test
	public void test07() {
        // 本月第一天00:00时刻
        LocalDateTime firstDay = LocalDate.now().withDayOfMonth(1).atStartOfDay();
        System.out.println(firstDay);

        // 本月第一天和最后一天
        // 2020-11-01
        // 2020-11-30
        LocalDate firstDayOfMonth = LocalDate.now().with(TemporalAdjusters.firstDayOfMonth());
        LocalDate lastDay = LocalDate.now().with(TemporalAdjusters.lastDayOfMonth());
        System.out.println(firstDayOfMonth);
        System.out.println(lastDay);

        // 下月第一天
        LocalDate nextMonthFirstDay = LocalDate.now().with(TemporalAdjusters.firstDayOfNextMonth());
        System.out.println(nextMonthFirstDay);

        // 本月第一个周日
        LocalDate firstSunDay = LocalDate.now().with(TemporalAdjusters.firstInMonth(DayOfWeek.SUNDAY));
        System.out.println(firstSunDay);
    }

运行结果：
2020-11-01T00:00
2020-11-01
2020-11-30
2020-12-01
2020-11-01
```

### 1.1.8 时间的比较

```java
    @Test
	public void test08() {
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime target = LocalDateTime.of(2020, 12, 28, 13, 14, 25);
        System.out.println(now.isAfter(target));
        System.out.println("使用equals比较两个日期是否相等：" + now.equals(target));
        System.out.println(LocalDate.now().isBefore(LocalDate.of(2020,12,28)));
        System.out.println(LocalTime.now().isBefore(LocalTime.of(13,14)));

    }

运行结果：
false
使用equals比较两个日期是否相等：false
true
false
```

### 1.1.9 Duration Period

- `Duration`：表示两个时刻之间的时间间隔
- `Period`：表示两个日期之间的天数

```java
	@Test
    public void test09() {
        LocalDateTime start = LocalDateTime.of(2020, 12, 29, 13, 14, 56);
        LocalDateTime end = LocalDateTime.of(2021, 1, 1, 13, 14, 36);
        Duration d = Duration.between(start, end);
        System.out.println(d);

        Period p = LocalDate.of(2020, 12, 28).until(LocalDate.of(2020, 11, 9));
        System.out.println(p);
    }

运行结果：
PT71H59M40S：代表相差71小时，59分钟，40秒
P-1M-19D：代表相差1个月19天
```



## 1.2 ZonedDatetime

- 表示一个带时区的日期和时间

### 1.2.1 直接获取时区和日期时间

```java
    @Test
	public void test01() {
        ZonedDateTime zdt = ZonedDateTime.now();
        System.out.println(zdt);
        ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York"));
        System.out.println(zny);
    }

运行结果：
2020-11-20T15:47:11.090+08:00[Asia/Shanghai]
2020-11-20T02:47:11.091-05:00[America/New_York]
```

### 1.2.2 使用atZone指定时区

```java
    @Test
    public void demo02() {
        LocalDateTime time = LocalDateTime.of(2020, 12, 28, 15, 16, 17);
        ZonedDateTime z1 = time.atZone(ZoneId.systemDefault());
        ZonedDateTime z2 = time.atZone(ZoneId.of("America/New_York"));
        System.out.println(z1);
        System.out.println(z2);
    }

运行结果：
2020-12-28T15:16:17+08:00[Asia/Shanghai]
2020-12-28T15:16:17-05:00[America/New_York]
```

### 1.2.3 时区间的转换

```java
    public void demo03() {
        // 以中国时区获取当前时间
        ZonedDateTime zbj = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
        // 转换为纽约时间
        ZonedDateTime zny = zbj.withZoneSameInstant(ZoneId.of("America/New_York"));
        System.out.println(zbj);
        System.out.println(zny);
    }

运行结果：
2020-11-20T15:49:37.976+08:00[Asia/Shanghai]
2020-11-20T02:49:37.976-05:00[America/New_York]
```



## 1.3 DateTimeFormatter

- 时间格式化
- 在格式化字符串中，如果需要输出固定字符，可以用`'xxx'`表示。

```java
    @Test
    public void test01() {
        ZonedDateTime zdt = ZonedDateTime.now();
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss ZZZZ");
        System.out.println(dtf.format(zdt));
        System.out.println("----------------");

        DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy MM dd HH:mm EE", Locale.CHINA);
        DateTimeFormatter dtf3 = DateTimeFormatter.ofPattern("yyyy MMM dd HH:mm EE", Locale.CHINA);
        System.out.println(dtf2.format(zdt));
        System.out.println(dtf3.format(zdt));
        System.out.println("----------------");

        DateTimeFormatter dtf4 = DateTimeFormatter.ofPattern("EE MM/dd/yyyy HH:mm", Locale.US);
        DateTimeFormatter dtf5 = DateTimeFormatter.ofPattern("EE MMM/dd/yyyy HH:mm", Locale.US);
        DateTimeFormatter dtf6 = DateTimeFormatter.ofPattern("EEEE MMMM/dd/yyyy HH:mm", Locale.US);
        System.out.println(dtf4.format(zdt));
        System.out.println(dtf5.format(zdt));
        System.out.println(dtf6.format(zdt));

    }

运行结果
2020-11-20T15:53:37 GMT+08:00
----------------
2020 11 20 15:53 星期五
2020 十一月 20 15:53 星期五
----------------
Fri 11/20/2020 15:53
Fri Nov/20/2020 15:53
Friday November/20/2020 15:53
```

## 1.4 Instant

- 时间戳

```java
    @Test
	public void test01() {
        Instant now = Instant.now();
        // 秒
        System.out.println(now.getEpochSecond());
        // 毫秒
        System.out.println(now.getNano());

        System.out.println(System.currentTimeMillis());
    }

运行结果：
1605858931
808000000
1605858931808
```

- 通过时间戳+时区指定时间

```java
    @Test
	public void test02() {
        Instant ins = Instant.ofEpochSecond(1604991817);
        ZonedDateTime zdt = ins.atZone(ZoneId.systemDefault());
        System.out.println(zdt);
    }

运行结果：
2020-11-10T15:03:37+08:00[Asia/Shanghai]
```

## 1.5 时间API之间相互转换图

![image-20201110154725620](http://img.akangaroo.cn/typora/java8Time.png)





# 2. Lambda表达式

## 2.1 Lambda基础

> 之前，在java中传递一个代码段不容易，不能直接传递代码段。Java是一种面向对象语言，所以必须构造一种对象，这个对象的类需要有一个方法包含所需的代码。

- **Java8之前排序**

```java
public class LambdaTest {

    public static void main(String[] args) {
        String[] arr = {"hello", "world", "Hello", "World"};
        Arrays.sort(arr, new LambdaComparator());
        System.out.println(String.join(", ", arr));
    }
}

class LambdaComparator implements Comparator<String> {

    @Override
    public int compare(String o1, String o2) {
        return o1.compareTo(o2);
    }
}
```

- **匿名内部类的写法**

```java
    @Test
	public void test01() {
        String[] arr = {"hello", "world", "Hello", "World"};
        Arrays.sort(arr, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.compareTo(o2);
            }
        });
        System.out.println(String.join(", ", arr));
    }
```

- **Lambda表达式**
  - lambda符号左侧：参数列表
  - lambda符号右侧：lambda体，即lambda表达式中需要执行的功能

>  *          数据类型可以省略不写
>  *          **lambda表达式需要函数式接口的支持**
>  *          语法格式1：无参数，无返回值
>              *          () -> System.out.println("Hello World");
>  *          语法格式2：有一个参数，无返回值
>              *          (x) -> System.out.println(x);
>  *          语法格式3：有一个参数，小括号可以省略不写
>              *          x -> System.out.println(x);
>  *          语法格式4：有两个参数，有返回值
>             *          Comparator<Integer> com = (e1, e2) -> {
>                                System.out.println("函数式接口");
>                                return Integer.compare(e1, e2);
>                         };
>  *          语法格式5：有两个参数，有返回值，但只有一条语句，return和{}都可以省略不写
>             * Comparator<Integer> com = (e1, e2) -> Integer.compare(e1, e2);

```java
    @Test
    public void test02() {
        String[] arr = {"hello", "world", "Hello", "World"};
        Arrays.sort(arr, (e1, e2) -> {
            return e1.compareTo(e2);
        });
        System.out.println(String.join(", ", arr));
    }
```

- 如果只有一行语句可以替换为

```java
    @Test
    public void test03() {
        String[] arr = {"hello", "world", "Hello", "World"};
        Arrays.sort(arr, (e1, e2) -> e1.compareTo(e2));
        System.out.println(String.join(", ", arr));
    }
```

> 如果一个Lambda表达式只在某些分支返回一个值，二另外一些分支不返回值，这是不合法的。例如，(int x) -> { if(x>=0) return 1; }



## 2.2 函数式接口

`@FunctionalInterface`

- **接口有且仅有一个抽象方法**
- 允许定义静态方法
- 允许定义默认方法
- 允许`java.lang.Object`中的public方法

例如Comparator接口，只有一个抽象方法`int compare(T o1, T o2)`

```java
@FunctionalInterface
public interface Comparator<T> {
        int compare(T o1, T o2);
        boolean equals(Object obj);
        default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    ...
}
```

- 接收`@FunctionalInterface`作为参数的时候，可以把实例化的匿名类改写为Lambda表达式，能大大简化代码



## 2.3 Lambda练习

1. 声明一个MyFun接口

   ```java
   @FunctionalInterface
   public interface MyFun {
   
       Integer getValue(Integer value);
   }
   ```

2. 实现运算操作

   ```java
       @Test
       public void test03() {
           Integer num = operation(100, (x) -> x * x);
           System.out.println(num);
   
           Integer num2 = operation(200, (y) -> y + 200);
           System.out.println(num2);
   
       }
   
       public Integer operation(Integer num, MyFun mf) {
           return mf.getValue(num);
       }
   ```

---------------

1. 声明一个函数式接口

   ```java
   @FunctionalInterface
   public interface MyFunction {
   
       String getStr(String str);
   }
   ```

   

2. 字符串的处理

   ```java
       @Test
       public void test04() {
           String s = strHandler("\t\t\t 一剑霜寒十四州     ", str -> str.trim());
           System.out.println(s);
   
           String s2 = strHandler("ABBCCCCC", str -> str.toLowerCase());
           System.out.println(s2);
   
           String s3 = strHandler("满堂花醉三千客", str -> str.substring(2, 5));
           System.out.println(s3);
       }
   
       public String strHandler(String str, MyFunction mf) {
           return mf.getStr(str);
       }
   ```



--------------------

1. 声明函数式接口

   ```java
   public interface MyFunction2<T, R> {
   
       R getValue(T t1, T t2);
   }
   ```

2. 运算

   ```java
       @Test
       public void test05() {
           op(100L, 200L, (x, y) -> x + y);
       }
   
       public void op(Long num1, Long num2, MyFunction2<Long, Long> mf) {
           System.out.println(mf.getValue(num1, num2));
       }
   ```



## 2.4 四大内置核心函数式接口

```java
// 1. 消费型接口
Consumer<T>
    void accept(T t);

// 2. 供给型接口
Supplier<T>
    T get();

// 3. 函数型接口
Function<T, R>
    R apply(T t);

// 4. 断言型接口
Predicate<T>
    boolean test(T t);
```



- `Consumer<T>`消费型接口，输入数据，但不输出。

```java
    /**
     * 消费型接口
     *
     * @param money
     * @param con
     */
    public void happy(double money, Consumer<Double> con) {
        con.accept(money);
    }

    @Test
    public void test01() {
        happy(10000, (m) -> System.out.println("消费" + m + "元"));
    }
```

- `Supplier<T>`供给型接口，输出，但没有输入

```java
    /**
     * 供给型接口：产生指定个数的整数，并放入集合中
     * 
     * @param num
     * @param sup
     * @return
     */
    public List<Integer> getNumList(int num, Supplier<Integer> sup) {
        List<Integer> list = new ArrayList<>();

        for (int i = 0; i < num; i++) {
            Integer n = sup.get();
            list.add(n);
        }

        return list;
    }

    @Test
    public void test02() {
        List<Integer> list = getNumList(10, () -> (int) (Math.random() * 100));
        list.forEach(System.out::println);
    }
```

- `Function<T, R>`函数型接口，一般将T转为R

```java
    /**
     * 函数型接口：字符串的处理
     *
     * @param str
     * @param fun
     * @return
     */
    public String strHandler(String str, Function<String, String> fun) {
        return fun.apply(str);
    }
    
    @Test
    public void test03() {
        String s = strHandler("\t\t\t 我笑了啊     ", str -> str.trim());
        System.out.println(s);
    }
```

- `Predicate<T>`断言型接口，一般用于判断

```java
    /**
     * 断言型接口
     *
     * @param list
     * @param pre
     * @return
     */
    public List<String> filterStr(List<String> list, Predicate<String> pre) {
        List<String> strList = new ArrayList<>();

        for (String s : list) {
            if (pre.test(s)) {
                strList.add(s);
            }
        }

        return strList;
    }
    @Test
    public void test04() {
        List<String> list = Arrays.asList("Hello", "World", "你好");
        List<String> newList = filterStr(list, str -> str.length() > 3);
        newList.forEach(System.out::println);
    }
```





# 3. 方法引用

## 3.1 语法形式

- 方法引用符号`::`

- 若Lambda 体中的内容方法已经实现了，我们可以使用“方法引用”
  - 主要有三种语法形式
    1. `对象::实例方法名`
    2. `类::静态方法名`
    3. `类::实例方法名`
  - 注意：lambda体中的返回值类型和参数类型要和函数式接口定义的抽象方法一致
- 构造器引用
  - 格式
    - ClassName::new
  - 注意：需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致
- 数组引用
  - 格式
    - Type::new

## 3.2 案例

- 新建一个Person类用于测试

```java
class Person {
    private String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    ...省略getter，setter，toString
}
```

- `对象::实例方法名`
  - 注意：lambda体中的返回值类型和参数类型要和函数式接口定义的抽象方法一致
  - 例如：Consumer<T>中的void accept(T t)和PrintStream中的public void println(String x)返回值类型一致

```java
	@Test
    public void test01() {
        Consumer<String> con = x -> System.out.println(x);
        con.accept("111");

        PrintStream ps = System.out;
        Consumer<String> con1 = ps::println;
        con1.accept("abcde");
    }
```

- `类::静态方法名`

```java
    @Test
    public void test02() {
        Comparator<Integer> com = (x, y) -> Integer.compare(x, y);

        Comparator<Integer> com1 = Integer::compare;
        int result = com1.compare(5, 7);
        System.out.println(result);
    }
```

- `类::实例方法名`
  - 要求：第一个参数是实例方法的调用者，第二个参数是这个方法的参数

```java
    @Test
    public void test03() {
        BiPredicate<String, String> bp = (x, y) -> x.equals(y);
        // 要求：第一个参数是实例方法的调用者，第二个参数是这个方法的参数
        BiPredicate<String, String> bp2 = String::equals;
        boolean result = bp2.test("aaa", "aaa");
        System.out.println(result);
    }
```

- `构造器引用`
  - 有参数的函数式接口为`Function<T, R>`
  - 无参数的函数式接口为`Supplier<T>`

```java
    @Test
    public void test04() {
        Supplier<Person> sup = () -> new Person();

        Supplier<Person> sup2 = Person::new;
        Person person = sup2.get();
        person.setName("zhangsan");
        System.out.println(person);
    }
	
    @Test
    public void test05() {
        Function<String, Person> fun = (name) -> new Person(name);

        Function<String, Person> fun1 = Person::new;
        Person person = fun1.apply("lisi");
        System.out.println(person.getName());
    }
```

- `数组引用`

```java
    @Test
    public void test06() {
        Function<Integer, String[]> fun = (x) -> new String[x];
        String[] strings = fun.apply(5);
        System.out.println(strings.length);

        Function<Integer, String[]> fun2 = String[]::new;
        String[] strings1 = fun2.apply(6);
        System.out.println(strings1.length);
    }
```



# 4. Stream API

## 4.1 Stream基础

- 流（Stream）是数据渠道，用于操作数据源（集合，数组等）所生成的元素序列。

- 特点：
  1. Stream自己不会存储元素
  2. Stream不会改变源对象。相反，他们会返回一个持有结果的新Stream
  3. Stream操作是延迟执行的。这意味着他们会等到需要结果的时候才执行

- Stream操作的三个步骤
  1. 创建Stream
     - 一个数据源（集合，数组），获取一个流
  2. 中间操作
     - 一个中间操作链，对数据源的数据进行处理
  3. 终止操作
     - 一个终止操作，执行中间操作链，并产生结果

## 4.2 Stream的创建

- 新建perosn类用于测试

```java
public class Person {
    private String name;

    public Person() {
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }
}
```



```java
public class StreamAPITest {

    /**
     * 创建Stream
     */
    @Test
    public void test01() {
        // 1. 通过Collection 系列集合提供的stream() 和 parallelStream()
        List<String> list = new ArrayList<>();
        Stream<String> stream = list.stream();

        // 2. 通过Arrays中的静态方法stream()获取数组流
        Person[] people = new Person[10];
        Stream<Person> stream1 = Arrays.stream(people);

        // 3. 通过Stream中的of()方法获取流
        Stream<String> stream2 = Stream.of("aa", "bb", "cc");

        // 4. 创建无限流
        // 迭代
        Stream.iterate(0, (x) -> x + 2)
                .limit(10)
                .forEach(System.out::println);

        System.out.println("---------------------------");

        // 生成
        Stream.generate(() -> Math.random())
                .limit(20)
                .forEach(System.out::println);

    }
}

```



## 4.3 筛选

- `filter(Predicate<? super T> predicate)`接收Lambda，从流中排除某些元素
- `limit(long maxSize)`截断流，使元素不超过给定数量
- `skip(long n)`跳过元素，返回一个扔掉了前n个元素的流，若流中的元素不满足n个，则返回一个空流，与limit(n)互补
- `distinct`通过流所生成元素的hashCode() 和 equals() 去除重复元素

> `惰性求值`：多个**中间操作**可以连接起来形成一个**流水线**，除非流水线上触发终止操作，否则**中间操作不会执行任何的处理**！而在**终止操作时一次性全部处理**。

```java
public class StreamAPITest2 {

    List<Person> persons = Arrays.asList(
            new Person("zhangsan"),
            new Person("lisi"),
            new Person("wangwu"),
            new Person("zhaoliu"),
            new Person("tianqi"),
            new Person("tianqi")
    );

    @Test
    public void test01() {
        persons.stream()
                .filter((e) -> e.getName().length() > 4)
                // 需要重写equals()和hashCode()
                .distinct()
                .forEach(System.out::println);

    }
}
```



## 4.4 映射

- `map(Function<? super T, ? extends R> mapper)`接收Lambda，将元素转换成其他形式或者提取信息，接受一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新元素

  ```java
      @Test
      public void test02() {
          List<String> list = Arrays.asList("aaa", "bbb", "ccc", "ddd");
  
          list.stream()
                  .map((str) -> str.toUpperCase())
                  .forEach(System.out::println);
  
          System.out.println("-------------");
  
          persons.stream()
                  .map(Person::getName)
                  .forEach(System.out::println);
          
          // map()返回的是一个流中流
          Stream<Stream<Character>> streamStream = list.stream()
                  .map(x -> {
                      List<Character> list1 = new ArrayList<>();
  
                      for (Character ch : x.toCharArray()) {
                          list1.add(ch);
                      }
  
                      return list1.stream();
                  });
  
          streamStream.forEach((stream) -> {
              stream.forEach(System.out::println);
          });
          
      }
  
  	
  ```
  
  
  
- `flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

  ```java
      @Test
      public void test02() {
          List<String> list = Arrays.asList("aaa", "bbb", "ccc", "ddd");
          
  		list.stream()
                  .flatMap(x -> {
                      List<Character> list1 = new ArrayList<>();
  
                      for (Character ch : x.toCharArray()) {
                          list1.add(ch);
                      }
  
                      return list1.stream();
                  })
                  .forEach(System.out::println);
          
      }
  ```

- `map`和`flatmap`的区别

  - 调用`map()`之后返回的是一个String类型的数组
  - 调用`flatMap()`之后返回的是string类型的变量

  > map(func)函数会对每一条输入进行指定的func操作，然后为每一条输入返回一个对象；
  >
  > 而flatMap(func)也会对每一条输入进行执行的func操作，然后每一条输入返回一个对象，但是最后会将所有的对象再合成为一个对象；

```java
    public void test() {
        String[] words = {"Hello", "World"};

        Stream<String[]> stream = Arrays.stream(words).map(e -> e.split(""));

        stream.forEach(e -> System.out.println(Arrays.toString(e)));

        Stream<String> stream1 = Arrays.stream(words).flatMap(e -> Arrays.stream(e.split("")));
        stream1.forEach(System.out::println);
    }

运行结果：
[H, e, l, l, o]
[W, o, r, l, d]
HelloWorld
```

## 4.5 查找和匹配

- `allMatch`是否匹配所有元素
- `anyMatch`是否匹配至少一个元素
- `noneMatch`没有匹配的元素
- `findFirst`返回第一个元素
- `findAny`返回当前流中的任意元素
- `count`返回流中元素的总个数
- `max`返回流中的最大值
- `min`返回流中的最小值

```java
public class StreamAPITest3 {

    List<Person> personList = Arrays.asList(
            new Person("zhangsan"),
            new Person("lisi"),
            new Person("wangwu"),
            new Person("zhaoliu")
    );

    @Test
    public void test01() {
        // allMatch是否匹配所有元素
        boolean b = personList.stream()
                .allMatch((e) -> e.getName().length() == 0);
        System.out.println(b);

        // anyMatch是否匹配至少一个元素
        boolean b1 = personList.stream()
                .anyMatch((e) -> e.getName().length() == 4);
        System.out.println(b1);

        // noneMatch没有匹配的元素
        boolean b2 = personList.stream()
                .noneMatch((e) -> e.getName().length() == 4);
        System.out.println(b2);

        // findFirst返回第一个元素
        Optional<Person> op = personList.stream()
                .sorted((e1, e2) -> e1.getName().compareTo(e2.getName()))
                .findFirst();

        System.out.println(op.get());

        // findAny返回当前流中的任意元素
        Optional<Person> op1 = personList.stream()
                .filter((e) -> e.getName().length() == 4)
                .findAny();

        System.out.println(op1.get());

        // count返回流中元素的总个数
        long count = personList.stream()
                .count();
        System.out.println(count);

        // max返回流中的最大值
        Optional<Person> op2 = personList.stream()
                .max((e1, e2) -> e1.getName().compareTo(e2.getName()));
        System.out.println(op2.get());

        // min返回流中的最小值
        Optional<String> op3 = personList.stream()
                .map(Person::getName)
                .min(String::compareTo);
        System.out.println(op3.get());

        /*Optional<Person> op3 = personList.stream()
                .min((e1, e2) -> e1.getName().compareTo(e2.getName()));
        System.out.println(op3.get());*/
    }
}
```



## 4.6 规约

```java
public class StreamAPITest4 {

    @Test
    public void test01() {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);

        Integer sum = list.stream()
                .reduce(0, (x, y) -> x + y);

        System.out.println(sum);

        Optional<Integer> op = list.stream()
                .reduce((x, y) -> x + y);
        if (op.isPresent()) {
            System.out.println(op.get());
        }
    }
}
```

- `reduce()`方法传入的对象是`BinaryOperator`接口，它定义了一个`apply()`方法，负责把上次累加的结果和本次的元素 进行运算，并返回累加的结果

```java
@FunctionalInterface
public interface BinaryOperator<T> {
    // Bi操作：两个输入，一个输出
    T apply(T t, T u);
}
```



## 4.7 收集

- `collect()`将流转换为其他形式，接收一个Colletor接口的实现，用于给Stream中元素汇总的方法

```java
public class StreamAPITest4 {

    List<Person> personList = Arrays.asList(
            new Person("zhangsan"),
            new Person("lisi"),
            new Person("wangwu"),
            new Person("zhaoliu")
    );

    @Test
    public void test02() {
        
        // 转换为List
        List<String> list = personList.stream()
                .map(Person::getName)
                .collect(Collectors.toList());

        list.forEach(System.out::println);

        System.out.println("---------------------------");
        
        // 转换为HashSet
        HashSet<String> hashSet = personList.stream()
                .map(Person::getName)
                .collect(Collectors.toCollection(HashSet::new));

        hashSet.forEach(System.out::println);
    }

    @Test
    public void test03() {
        // 总数
        Long count = personList.stream()
                .collect(Collectors.counting());
        System.out.println(count);

        // 平均值
        Double avg = personList.stream()
                .collect(Collectors.averagingDouble((e) -> e.getName().length()));
        System.out.println(avg);

        // 总和
        Double sum = personList.stream()
                .collect(Collectors.summingDouble((e) -> e.getName().length()));
        System.out.println(sum);

        // 最大值
        Optional<Person> op = personList.stream()
                .collect(Collectors.maxBy((e1, e2) -> e1.getName().compareTo(e2.getName())));
        if (op.isPresent()) {
            System.out.println(op.get());
        }

        // 最大值
        Optional<Person> op2 = personList.stream()
                .collect(Collectors.minBy((e1, e2) -> e1.getName().compareTo(e2.getName())));
        if (op.isPresent()) {
            System.out.println(op2.get());
        }

        // 综合所有数据
        DoubleSummaryStatistics all = personList.stream()
                .collect(Collectors.summarizingDouble((e) -> e.getName().length()));
        System.out.println(all);
    }
}
```

- 分组

```java
    @Test
    public void test04() {
        Map<String, List<Person>> map = personList.stream()
                .collect(Collectors.groupingBy(Person::getName));
        System.out.println(map);
    }
```



- 分区

```java
    @Test
    public void test05() {
        Map<Boolean, List<Person>> map = personList.stream()
                .collect(Collectors.partitioningBy((e) -> e.getName().length() > 4));
        System.out.println(map);
    }
```



## 4.8 并行

把一个普通`Stream`转换为可以并行处理的`Stream`非常简单，只需要用`parallel()`进行转换。

## 4.9 StreamAPI 练习

```java
public class StreamAPITest5 {

    List<Person> personList = Arrays.asList(
            new Person("zhangsan"),
            new Person("lisi"),
            new Person("wangwu"),
            new Person("zhaoliu")
    );

    /**
     * 给定一个数字列表，返回一个由每个数的平方构成的列表
     */
    @Test
    public void test01() {
        Integer[] nums = new Integer[]{1, 2, 3, 4, 5};

        Arrays.stream(nums)
                .map((x) -> x * x)
                .forEach(System.out::println);
    }

    /**
     * 用map和reduce 方法数一数流中有多少person
     */
    @Test
    public void test02() {
        Optional<Integer> op = personList.stream()
                .map((e) -> 1)
                .reduce(Integer::sum);
        if (op.isPresent()) {
            System.out.println(op.get());
        }
    }

    
}
```





# 5. Optional

## 5.1 创建

- `of()`

```java
    /**
     * of()方法通过工厂方法创建Optional类。
     * 创建对象时传入的参数不能为null。如果传入参数为null，则抛出NullPointerException 。
     */
    @Test
    public void test01() {
        Optional<String> op = Optional.of("Hello");
    }
```

- `empty()`

```java
    /**
     * 只是构建一个空的Optional类
     */
    @Test
    public void test02() {
        Optional<Object> op = Optional.empty();
    }
```

- `ofNullable()`

```java
    /**
     *     可以接受参数为null的情况
     *     源代码如下，就是empty() 和 of() 的综合
     *     public static <T> Optional<T> ofNullable(T value) {
     *         return value == null ? empty() : of(value);
     *     }
     */
        @Test
    public void test03() {
        Optional op = Optional.ofNullable(null);  
    }
```



## 5.2 常用方法

- `get()`如果Optional有值则将其返回，否则抛出异常 `NoSuchElementException`

```java
    @Test
    public void test() {

        try {
            Optional op = Optional.ofNullable(null);
            System.out.println("null-->" + op.get());
        } catch (NoSuchElementException e) {
            System.out.println("NoSuchElementException异常");
        }
    }
```

- `isPresent()`如果值存在返回true，否则返回false

```java
    @Test
    public void test05() {
        Optional<Object> o = Optional.ofNullable(null);
        
        if (o.isPresent()) {
            System.out.println(o.get());
        }
        
    }
```

- `ifPresent(Consumer<? super T> consumer)`如果Optional实例有值则为其调用consumer，否则不做处理

```java
    @Test
    public void test06() {
        Optional<String> o = Optional.ofNullable("Hello");
        o.ifPresent(System.out::println);
    }
```

- `orElse()`如果有值则将其返回，否则返回指定的其它值。

```java
    @Test
    public void test07() {
        Optional<String> o = Optional.ofNullable("Hello");
        String str = o.orElse("HELLO");
        System.out.println(str);

        Optional<String> o1 = Optional.ofNullable(null);
        String string = o1.orElse("World");
        System.out.println(string);
    }

运行结果：
Hello
World
```

- `orElseGet(Supplier<? extends T> other)`

```java
    @Test
    public void test08() {
        Optional<String> o = Optional.ofNullable("Hello");
        String str = o.orElseGet(() -> "HELLO");
        System.out.println(str);

        Optional<String> o1 = Optional.ofNullable(null);
        String string = o1.orElseGet(() -> "World");
        System.out.println(string);
    }

运行结果：
Hello
World
```

- `map(Function<? super T, ? extends U> mapper)`

```java
    @Test
    public void test09() {
        Optional<String> op = Optional.ofNullable("Hello");

        Optional<String> optional = op.map((e) -> e.toUpperCase());
        System.out.println(optional.get());
    }

运行结果：
HELLO
```

- `flatMap()`

```java
    @Test
    public void test10() {
        Optional<String> op1 = op.flatMap((e) -> Optional.of("Hello"));
        System.out.println(op1.get());
    }

运行结果：
Hello
```

- `map()`和`flatMap()`的区别

  `Function()`函数的返回值类型不同，`map()`返回值类型为`U`，而`flatMap()`返回值类型为`Optional`

  ```java
  	public<U> Optional<U> map(Function<? super T, ? extends U> mapper)
      
      public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)
  ```

- `filter()`

```java
    @Test
    public void test05() {
        Optional<String> op = Optional.of("OptionalTest");
        Optional<String> op1 = op.filter((e) -> e.length() > 5);
        System.out.println(op1.orElse("长度小于等于5"));

        Optional<String> op2 = Optional.of("hehe");
        Optional<String> op3 = op2.filter((e) -> e.length() > 5);
        System.out.println(op3.orElse("长度小于等于5"));
    }
```



