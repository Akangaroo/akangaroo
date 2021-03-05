# Lambda表达式

## 什么是Lambda

之前，在java中传递一个代码段不容易，不能直接传递代码段。Java是一种面向对象语言，所以必须构造一种对象，这个对象的类需要有一个方法包含所需的代码。

**Java8之前排序**

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

**匿名内部类的写法**

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

**Lambda表达式的写法**

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

### Lambda表达式语法

- lambda符号左侧：参数列表
- lambda符号右侧：lambda体，即lambda表达式中需要执行的功能

语法格式：

- 数据类型可以省略不写
- lambda表达式需要函数式接口的支持

```java
// 语法格式1：无参数，无返回值
          () -> System.out.println("Hello World");
// 语法格式2：有一个参数，无返回值
          (x) -> System.out.println(x);
// 语法格式3：有一个参数，小括号可以省略不写
          x -> System.out.println(x);
// 语法格式4：有两个参数，有返回值
          Comparator\<Integer\> com = (e1, e2) -> {
                      System.out.println("函数式接口");
                      return Integer.compare(e1, e2);
          };
// 语法格式5：有两个参数，有返回值，但只有一条语句，return和{}都可以省略不写
Comparator<Integer> com = (e1, e2) -> Integer.compare(e1, e2);
```



如果一个Lambda表达式只在某些分支返回一个值，二另外一些分支不返回值，这是不合法的。如下所示，只是在x >= 0 的时候返回1，x < 0的情况却没有考虑。

```java
(int x) -> { if(x>=0) return 1; }
```





## 函数式接口

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

接收`@FunctionalInterface`作为参数的时候，可以把实例化的匿名类改写为Lambda表达式，能大大简化代码



## Lambda表达式案例

### 案例一

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

### 案例二

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

### 案例三

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



## 四大内置核心函数式接口

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



### Consumer\<T\>

消费型接口，输入数据，但不输出。

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

### Supplier\<T\>

供给型接口，输出，但没有输入

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

### Function\<T, R\>

函数型接口，一般将T转为R

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

### Predicate\<T\>

断言型接口，一般用于判断

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





# 方法引用

## 语法形式

**若Lambda 体中的内容方法已经实现了，我们可以使用“方法引用”**

方法引用符号`::`

- 主要有三种语法形式
  1. `对象::实例方法名`
  2. `类::静态方法名`
  3. `类::实例方法名`
- 注意：lambda体中的返回值类型和参数类型要和函数式接口定义的抽象方法一致

构造器引用

- 格式
  - `ClassName::new`
- 注意：需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致

数组引用

- 格式
  - `Type::new`

## 案例

新建一个Person类用于测试

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

`对象::实例方法名`

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

`类::静态方法名`

```java
    @Test
    public void test02() {
        Comparator<Integer> com = (x, y) -> Integer.compare(x, y);

        Comparator<Integer> com1 = Integer::compare;
        int result = com1.compare(5, 7);
        System.out.println(result);
    }
```

`类::实例方法名`

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

`构造器引用`

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

`数组引用`

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


