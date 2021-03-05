# Optional类

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

## 创建

`of()`

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

`empty()`

```java
    /**
     * 只是构建一个空的Optional类
     */
    @Test
    public void test02() {
        Optional<Object> op = Optional.empty();
    }
```

`ofNullable()`

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



## 常用方法

### get()

如果Optional有值则将其返回，否则抛出异常 `NoSuchElementException`

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

### isPresent()

如果值存在返回true，否则返回false

```java
    @Test
    public void test05() {
        Optional<Object> o = Optional.ofNullable(null);
        
        if (o.isPresent()) {
            System.out.println(o.get());
        }
        
    }
```

### ifPresent()

`ifPresent(Consumer<? super T> consumer)`

如果Optional实例有值则为其调用consumer，否则不做处理

```java
    @Test
    public void test06() {
        Optional<String> o = Optional.ofNullable("Hello");
        o.ifPresent(System.out::println);
    }
```



### orElse()

如果有值则将其返回，否则返回指定的其它值。

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

### orElseGet()

`orElseGet(Supplier<? extends T> other)`

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

### map()

`map(Function<? super T, ? extends U> mapper)`

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

### flatMap()

```java
    @Test
    public void test10() {
        Optional<String> op1 = op.flatMap((e) -> Optional.of("Hello"));
        System.out.println(op1.get());
    }

运行结果：
Hello
```

`map()`和`flatMap()`的区别

`Function()`函数的返回值类型不同，`map()`返回值类型为`U`，而`flatMap()`返回值类型为`Optional`

```java
	public<U> Optional<U> map(Function<? super T, ? extends U> mapper)
    
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)
```



### filter()

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

