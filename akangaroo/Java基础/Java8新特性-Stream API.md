# Stream API

## Stream基础

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

## Stream的创建

新建perosn类用于测试

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



## 筛选

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



## 映射

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

## 查找和匹配

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



## 规约

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



## 收集

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



## 并行

把一个普通`Stream`转换为可以并行处理的`Stream`非常简单，只需要用`parallel()`进行转换。

## StreamAPI 练习

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



