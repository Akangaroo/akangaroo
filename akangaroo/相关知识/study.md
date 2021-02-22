# 集合删除元素

```java
public class DeleteElement {


    public static void main(String[] args) {
        // 下面语句会报错
        // List<String> list = Arrays.asList("1","2","2","3");

        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        list.add("2");
        list.add("3");

        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String item = it.next();
            if ("2" == item) {
                it.remove();
            }
        }
        System.out.println(list.toString());
    }
}
```

> 调用Arrays.asList()生产的List的add、remove方法时报异常，这是由Arrays.asList() 返回的是Arrays的内部类ArrayList， 而不是java.util.ArrayList。Arrays的内部类ArrayList和java.util.ArrayList都是继承AbstractList，remove、add等方法AbstractList中是默认throw UnsupportedOperationException而且不作任何操作。java.util.ArrayList重新了这些方法而Arrays的内部类ArrayList没有重新，所以会抛出异常。

- 将上述语句改成如下形式即可成功解决问题

```java
		List<String> strings = Arrays.asList("1", "2", "2", "3");
        List<String> list = new ArrayList<>(strings);
```







# Apache Commons-Collections4

[参考文章](https://blog.csdn.net/f641385712/article/details/84109098)

## Bag

```java
    /**
     * bag：该集合会记录对象在集合中出现的次数
     */
    @Test
    public void test01() {
        Bag<String> hashBag = new HashBag<>();

        hashBag.add("s1");
        hashBag.add("s1");
        hashBag.add("s2");

        Iterator<String> it = hashBag.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }

        System.out.println("包中元素个数为：" + hashBag.size());
        System.out.println("s1元素个数为：" + hashBag.getCount("s1"));
        System.out.println("不重复的元素个数：" + hashBag.uniqueSet().size());
    }

运行结果：
s1
s1
s2
包中元素个数为：3
s1元素个数为：2
不重复的元素个数：2
```





## BidiMap：双重Map

==双向映射==

```java
/**
     * BidiMap：双重Map，可以根据值查找键，也可以根据键查找值
     */
    @Test
    public void test02() {
        BidiMap<String, String> bidiMap = new DualHashBidiMap<>();

        bidiMap.put("k1", "v1");
        bidiMap.put("k2", "v2");

        // 可以使用迭代器遍历
        MapIterator<String, String> it = bidiMap.mapIterator();
        while (it.hasNext()) {
            // 相当于调用getKey()，但是此方法必须调用
            String next = it.next();
            System.out.println(next + "\t" + it.getKey() + "\t" + it.getValue());
        }

        System.out.println("根据键拿到值：" + bidiMap.get("k1"));
        System.out.println("根据值拿到键：" + bidiMap.getKey("v2"));
        System.out.println(bidiMap.getOrDefault("k", "defaultValue"));

        // 得到一个逆序的视图
        BidiMap<String, String> inverseBidiMap = bidiMap.inverseBidiMap();

        // 因为是视图，所以无法删除
        inverseBidiMap.remove("k1");
        inverseBidiMap.removeValue("v2");

        System.out.println(bidiMap);
        System.out.println(inverseBidiMap);

    }


运行结果：
k1	k1	v1
k2	k2	v2
根据键拿到值：v1
根据值拿到键：k2
defaultValue
{k1=v1, k2=v2}
{v1=k1, v2=k2}
```



## MultiKeyMap：多键Map



```java
    /**
     * MultiKeyMap：多键Map
     */
    @Test
    public void test03() {

        MultiKey<String> multiKey = new MultiKey<>("001", "a");

        MultiKeyMap<String, String> multiKeyMap = new MultiKeyMap<>();

        // 多个键对应一个值
        multiKeyMap.put(multiKey, "v1");
        multiKeyMap.put("002", "a", "v2");
        multiKeyMap.put("003", "a", "A", "v3");
        System.out.println(multiKeyMap);

        System.out.println(multiKeyMap.get("001", "a"));
        System.out.println(multiKeyMap.get("003", "a", "A"));

    }

运行结果：
{MultiKey[002, a]=v2, MultiKey[001, a]=v1, MultiKey[003, a, A]=v3}
v1
v3
```



## MultiValuedMap：多值Map

```java
    /**
     * MultiValuedMap：多值Map
     */
    @Test
    public void test04() {
        MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();

        map.put("k1", "v1");
        map.put("k1", "v2");
        System.out.println(map);

        Collection<String> values = map.values();
        System.out.println(values);

/*        map.remove("k1");
        System.out.println(map);*/

        // 可以直接删除map中的某一个值
        map.removeMapping("k1","v2");
        System.out.println(map);

        // 可以一次性放入多个value
        map.putAll("k2", Arrays.asList("1", "2", "3"));
        System.out.println(map);

        MultiSet<String> keys = map.keys();
        System.out.println(keys); // [k1:1, k2:3], 后面的数字代表value的个数

        Set<String> strings = map.keySet();
        System.out.println(strings);

        System.out.println(map.size()); // 4，代表了map中所有value的个数

    }

运行结果
{k1=[v1, v2]}
[v1, v2]
{k1=[v1]}
{k1=[v1], k2=[1, 2, 3]}
[k1:1, k2:3]
[k1, k2]
4
```





## MultiSet：多值Set

```java
/**
     * MultiSet：
     */
    @Test
    public void test05() {
        MultiSet<String> multiSet = new HashMultiSet<>();

        multiSet.add("haha");
        multiSet.add("haha");
        multiSet.add("wuwu");
        multiSet.add("nanshou");
        multiSet.add("nanshou");

        System.out.println(multiSet);
        System.out.println(multiSet.size());

        // 批量添加
        multiSet.add("xiaowang", 5);
        System.out.println(multiSet);

        // 移除，只移除一个
        System.out.println("移除前个数：" + multiSet.getCount("haha"));
        multiSet.remove("haha");
        System.out.println("移除后个数：" + multiSet.getCount("haha"));

        // 一次性移除n个
        multiSet.remove("nanshou", multiSet.getCount("nanshou"));
        System.out.println(multiSet);

        // removeAll 把指定的key，全部移除
        multiSet.removeAll(Arrays.asList("wuwu", "hehe", "heihei", "xiaowang", "haha"));
        System.out.println(multiSet);

    }


运行结果
[nanshou:2, haha:2, wuwu:1]
5
[nanshou:2, haha:2, xiaowang:5, wuwu:1]
移除前个数：2
移除后个数：1
[haha:1, xiaowang:5, wuwu:1]
[]
```





## LRUMap：最近最少使用Map

```java
    /**
     * LRUMap：最近最少使用
     */
    @Test
    public void test06() {
        LRUMap<String, String> lruMap = new LRUMap<>(3);

        System.out.println(lruMap);
        System.out.println(lruMap.isFull());
        System.out.println(lruMap.size());
        // 默认100
        System.out.println(lruMap.maxSize());

        lruMap.put("k1", "v1");
        lruMap.put("k2", "v2");
        lruMap.put("k3", "v3");

        System.out.println(lruMap);
        System.out.println(lruMap.isFull());
        System.out.println(lruMap.size());
        System.out.println(lruMap.maxSize());

        /*lruMap.put("k4", "v4");
        lruMap.put("k5", "v5");
        System.out.println("没有过度使用k1的情况下：" + lruMap);*/

        lruMap.get("k1");
        lruMap.get("k1");
        lruMap.get("k1");

        lruMap.put("k4", "v4");
        lruMap.put("k5", "v5");
        System.out.println("过度使用k1的情况下：" + lruMap);

    }
```





## SingletonMap：单例Map

```java
    /**
     * SingletonMap
     */
    @Test
    public void test07() {
        SingletonMap<String, String> singletonMap = new SingletonMap<>();

        System.out.println(singletonMap); // {null=null}
        System.out.println(singletonMap.size()); // 1
        System.out.println(singletonMap.maxSize()); // 1

        // 不能再设置值，因为已经存满了
        // singletonMap.put("k1", "v1");  Cannot put new key/value pair - Map is fixed size singleton

        singletonMap.setValue("hehe");
        System.out.println(singletonMap); // {null=hehe}

        // 在构造的时候，就给key和value赋值
        singletonMap = new SingletonMap<>("k1", "v1");
        System.out.println(singletonMap);

    }
```







## MapUtils

```java
 if (map != null) {
     return Collections.emptyMap();
 }
// 可以替换成
return MapUtils.emptyIfNull(map);
```



```java
    @Test
    public void test05() {
        Map<String, String> map = new HashMap<>();
        map.put("k1", "v1");
        map.put("k2", "v2");
        map.put("k3", "v3");

        Map<String, String> invertMap = MapUtils.invertMap(map);
        System.out.println(map);
        System.out.println(invertMap);
    }

运行结果
{k1=v1, k2=v2, k3=v3}
{v2=k2, v3=k3, v1=k1}
```



```java
    @Test
    public void test05() {
        Map<String, String> map = new HashMap<>();
        map.put("k1", "v1");
        map.put("k2", "v2");
        map.put("k3", "v3");

        IterableMap<String, String> iterableMap = MapUtils.iterableMap(map);
        MapIterator<String, String> it = iterableMap.mapIterator();
        while (it.hasNext()) {
            it.next();
            System.out.println(it.getKey() + "\t" + it.getValue());
        }
    }
```



# commons-lang3

## StringUtils

```java
    @Test
    public void test01() {
        // isBlank() 和 isEmpty()
        // isBlank()判断是否空白字符串，null, "", " "都属于空白字符串
        // isEmpty()判断是否空字符串，null,""属于空字符串
        String str = " ";
        System.out.println(StringUtils.isEmpty(str));
        System.out.println(StringUtils.isBlank(str));
        System.out.println("---------------------------");

        // capitalize()首字母变为大写
        // uncapitalize()首字母变为小写
        System.out.println(StringUtils.capitalize("dz"));
        System.out.println(StringUtils.uncapitalize("DZ"));
        System.out.println("---------------------------");

        // remove()移除字符或字符串
        System.out.println(StringUtils.remove("hehe",'h'));
        System.out.println(StringUtils.remove("hehe","he"));
        System.out.println(StringUtils.removeStart("heihe","h"));
        System.out.println(StringUtils.removeEndIgnoreCase("HEiHE","HE"));
        System.out.println("---------------------------");

        // trim()去除字符串两端的控制符，空字符串，null返回null
        // trimToEmpty()，null返回""
        // trimToNull()，空和null都返回null
        // deleteWhitespace 去除字符串中所有的空白符和转义字符
        System.out.println(StringUtils.trim("   aaa   \t"));
        System.out.println(StringUtils.trimToEmpty(null));
        System.out.println(StringUtils.trimToNull(""));
        System.out.println(StringUtils.deleteWhitespace("   zhang san \t\t\n"));
        System.out.println("---------------------------");

        // strip(String str, String stripChars) 去掉str两端的在stripChars中的字符
        // StringUtils.strip("  abcyx", "xyz") = "  abc"，去除字符串两端的字符
        System.out.println(StringUtils.strip(" aaabbbccc \t\t    "));
        System.out.println(StringUtils.strip("[aaa, bbb, ccc]", "[]"));
        System.out.println(StringUtils.stripStart("[aaa, bbb, ccc]","[]"));
        System.out.println(StringUtils.stripEnd("[aaa, bbb, ccc]","[]"));
        System.out.println("---------------------------");

        // contains()
        System.out.println(StringUtils.contains("",""));
        System.out.println(StringUtils.contains("abc",""));
        System.out.println("---------------------------");

        // indexOf()
        // indexOfAny()
        System.out.println(StringUtils.indexOf("aaabb1cc1c",'1'));
        System.out.println(StringUtils.indexOfAny("aaa bbb", new String[]{"ab", " "}));
        System.out.println("---------------------------");

        // left right mid
        System.out.println(StringUtils.left("aaaaabbbbbb", 6));
        System.out.println(StringUtils.right("aaaaabbbbbb", 6));
        System.out.println(StringUtils.mid("aaaaabbbbbb", 5, 3));
        System.out.println("---------------------------");

        // split()
        System.out.println(Arrays.toString(StringUtils.split("aaaabbbb")));
        System.out.println(Arrays.toString(StringUtils.split("a-b-c-d", "-")));
        System.out.println("---------------------------");

        // join()
        System.out.println(StringUtils.join(new String[]{"1", "aaabbb", "ccc"}));
        System.out.println(StringUtils.join(new String[]{"1", "aaabbb", "ccc"}, "-"));

        System.out.println("---------------------------");

        // replace()
        System.out.println(StringUtils.replace("aaabbbccc","bbb", "zhangsan"));

        System.out.println("---------------------------");

        // overlay(String str,String new,int start,int end) 用字符串new 覆盖字符串str从start 到 end 之间的串
        System.out.println(StringUtils.overlay("aaabb", "zhangsan", 1, 3));

        System.out.println("---------------------------");

        // chop(String str) 去掉字符串的最后一个字符,比如\r\n
        System.out.println(StringUtils.chop("hehehe\r\n"));
        System.out.println(StringUtils.chop("hehehe"));

        System.out.println("---------------------------");

        // repeat(String str,int repart) 重复字符串repeat次
        System.out.println(StringUtils.repeat("张三", 3));

        System.out.println("---------------------------");

        // swapCase(String str) 字符串中的大写转小写，小写转换为大写
        System.out.println(StringUtils.swapCase("aaAAccDDEE"));

        System.out.println("---------------------------");

        // maxWidth最少是4
        // 应用场景： 需要将一段日志写入MySql，日志字段类型为varchar(1024)，那么当日志长度大于1024时，写入会失败，此时我们可以使用这个函数处理过长的日志字符串
        System.out.println(StringUtils.abbreviate("1234567", 4));
        // abbreviateMiddle 截断中间部分
        System.out.println(StringUtils.abbreviateMiddle("1234567890", "...", 5));

        System.out.println("---------------------------");

        // 在字符串两边填充空格，并返回带空格的新字符串，新字符串长度等于size参数的长度
        System.out.println(StringUtils.center("aaa",5));
        System.out.println(StringUtils.center("aaa",5, "="));
        
        System.out.println("---------------------------");

        // 左边填充字符串，右边填充字符串
        System.out.println(StringUtils.leftPad("aaa", 10, "==="));
        System.out.println(StringUtils.leftPad("aaa", 10));
        System.out.println(StringUtils.rightPad("aaa", 10, "==="));
        System.out.println(StringUtils.rightPad("aaa", 10));

    }
```







# Json操作

- [参考地址1](https://segmentfault.com/a/1190000011212806)
- [参考地址2](https://juejin.cn/post/6844904019614236680)

## fastjson

### 介绍

- `FastJson`的使用主要是三个对象
  - `JSON`fastJson的解析器，用于JSON格式字符串与JSON对象及javaBean之间的转换
  - `JSONObject`fastJson提供的json对象
  - `JSONArray`fastJson提供json数组对象

- `JSONArray`和`JSONObject`继承`JSON`：

- 常用的几个方法
  - `parseObject(String text, Class<T> clazz)`
  - `parseArray(String text, Class<T> clazz)`
  - `toJSONString(Object object)`

### 操作

- **依赖导入**

  ```xml
          <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>fastjson</artifactId>
              <version>1.2.75</version>
          </dependency>
  ```

- **定义三种形式的JSON字符串**

  ```java
      /**
       * json字符串-简单对象型
       */
      private static final String  JSON_OBJ_STR = "{\"studentName\":\"lily\",\"studentAge\":12}";
  
      /**
       * json字符串-数组类型
       */
      private static final String  JSON_ARRAY_STR = "[{\"studentName\":\"lily\",\"studentAge\":12},{\"studentName\":\"lucy\",\"studentAge\":15}]";
  
      /**
       * 复杂格式json字符串
       */
      private static final String  COMPLEX_JSON_STR = "{\"teacherName\":\"crystal\",\"teacherAge\":27,\"course\":{\"courseName\":\"english\",\"code\":1270},\"students\":[{\"studentName\":\"lily\",\"studentAge\":12},{\"studentName\":\"lucy\",\"studentAge\":15}]}";
  
  ```

- **创建对应的实体类**

  ```java
  public class Teacher {
  
      private String teacherName;
      private Integer teacherAge;
      private Course course;
      private List<Student> students;
  	
      ...
  }
  
  public class Course {
      private String courseName;
      private Integer code;
      
      ...
  }
  
  public class Student {
  
      private String studentName;
      private Integer studentAge;
      
      ...
  }
  ```

  

#### 简单类型字符串和JSONObject

```java
    /**
     * Json简单类型字符串 转为 JSONObject
     */
    @Test
    public void test01() {
        JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
        System.out.println(jsonObject.getString("studentName") + "\t" + jsonObject.getInteger("studentAge"));
    }

    /**
     * JSONObject 转为 Json简单类型字符串
     */
    @Test
    public void test02() {
        JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
        // 方式一
        String str = JSONObject.toJSONString(jsonObject);
        System.out.println(str);
        // 方式二
        String string = jsonObject.toJSONString();
        System.out.println(string);
    }
```



#### 数组类型字符串和JSONArray

```java
    /**
     * Json数组类型字符串 转为 JSONArray
     */
    @Test
    public void test03() {
        JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);

        // 方式一
        for (int i = 0; i < jsonArray.size(); i++) {
            JSONObject jsonObject = jsonArray.getJSONObject(i);
            System.out.println(jsonObject.getString("studentName") + "\t" + jsonObject.getInteger("studentAge"));
        }
        
        // 方式二
        for (Object obj : jsonArray) {
            JSONObject jsonObject = (JSONObject)obj;
            System.out.println(jsonObject.getString("studentName") + "\t" + jsonObject.getInteger("studentAge"));
        }
    }

	/**
     * JSONArray 转为 Json数组类型字符串
     */
    @Test
    public void test04() {
        JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);
        // 方式一
        String str = JSONArray.toJSONString(jsonArray);
        System.out.println(str);
        // 方式二
        String string = jsonArray.toJSONString();
        System.out.println(string);
    }
```



#### 复杂类型字符串和JSONObject

```java
    /**
     * 复杂类型字符串 转为 JSONObject
     */
    @Test
    public void test05() {
        JSONObject jsonObject = JSONObject.parseObject(COMPLEX_JSON_STR);
        System.out.println(jsonObject.getString("teacherName") + "\t" + jsonObject.getInteger("teacherAge"));

        JSONObject course = jsonObject.getJSONObject("course");
        System.out.println(course.getString("courseName") + "\t" + course.getString("code"));

        JSONArray students = jsonObject.getJSONArray("students");
        for (Object student : students) {
            JSONObject obj = (JSONObject)student;
            System.out.println(obj.getString("studentName") + "\t" + obj.getInteger("studentAge"));
        }

    }

    /**
     * JSONObject 转为 复杂类型字符串
     */
    @Test
    public void test06() {
        JSONObject jsonObject = JSONObject.parseObject(COMPLEX_JSON_STR);
        // 方式1
        String string = jsonObject.toJSONString();
        System.out.println(string);
        // 方式2
        String string1 = JSONObject.toJSONString(jsonObject);
        System.out.println(string1);
    }
```



#### 简单类型字符串和JavaBean

```java
    /**
     * Json简单类型字符串 转为 JavaBean
     */
    @Test
    public void test07() {
        // 方式一
        JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
        String studentName = jsonObject.getString("studentName");
        Integer studentAge = jsonObject.getInteger("studentAge");
        Student stu = new Student(studentName, studentAge);
        System.out.println(stu);
        // 方式二
        Student student = JSONObject.parseObject(JSON_OBJ_STR, Student.class);
        System.out.println(student);
    }

    /**
     * JavaBean 转为 Json简单类型字符串
     */
    @Test
    public void test08() {
        Student student = new Student("zhangsan", 12);
        String string = JSONObject.toJSONString(student);
        System.out.println(string);
    }
```



#### 数组类型字符串和JavaBean

```java
    /**
     * Json数组类型字符串 转为 JavaBean
     */
    @Test
    public void test09() {
        // 方式一
        JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);

        List<Student> list = new ArrayList<>();
        Student student = null;
        for (Object obj : jsonArray) {
            JSONObject jsonObject = (JSONObject)obj;
            String studentName = jsonObject.getString("studentName");
            Integer studentAge = jsonObject.getInteger("studentAge");

            student = new Student(studentName, studentAge);
            list.add(student);
        }
        System.out.println(list);

        // 方式二
        List<Student> students = JSONArray.parseArray(JSON_ARRAY_STR, Student.class);
        System.out.println(students);

    }

    /**
     * JavaBean 转为 Json-数组类型字符串
     */
    @Test
    public void test10() {
        List<Student> students = Arrays.asList(new Student("zhangsan", 11),
                new Student("lisi", 12));

        String string = JSONArray.toJSONString(students);
        System.out.println(string);

    }
```



#### 复杂类型字符串和JavaBean

```java
    /**
     * 复杂类型Json字符串 转为 JavaBean
     */
    @Test
    public void test11() {
        Teacher teacher1 = JSONObject.parseObject(COMPLEX_JSON_STR, Teacher.class);
        System.out.println(teacher1);
    }

    /**
     * JavaBean 转为 复杂类型Json字符串
     */
    @Test
    public void test12() {
        Teacher teacher = JSONObject.parseObject(COMPLEX_JSON_STR, Teacher.class);
        String string = JSONObject.toJSONString(teacher);
        System.out.println(string);
    }
```



#### JavaBean和JSONObject

```java
    /**
     * JavaBean 转为 JSONObject
     */
    @Test
    public void test13() {
        Student student = new Student("zhangsan", 14);
        // 方式1
        String jsonString = JSONObject.toJSONString(student);
        JSONObject jsonObject = JSONObject.parseObject(jsonString);
        System.out.println(jsonObject);

        // 方式2
        JSONObject jsonObject1 = (JSONObject) JSONObject.toJSON(student);
        System.out.println(jsonObject1);
    }

    /**
     * JSONObject 转为 JavaBean
     */
    @Test
    public void test14() {
        JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
        Student student1 = JSONObject.parseObject(jsonObject.toJSONString(), Student.class);
        System.out.println(student1);
    }



    /**
     * JavaBean 转为 JSONArray
     */
    @Test
    public void test15() {
        List<Student> students = Arrays.asList(new Student("zhangsan", 11),
                new Student("lisi", 12));

        // 方式1
        String jsonString = JSONArray.toJSONString(students);
        JSONArray jsonArray = JSONArray.parseArray(jsonString);
        System.out.println(jsonArray);

        // 方式2
        JSONArray jsonArray1 = (JSONArray) JSONArray.toJSON(students);
        System.out.println(jsonArray1);

    }

    /**
     * JSONArray 转为 JavaBean
     */
    @Test
    public void test16() {
        JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);
        List<Student> students = JSONArray.parseArray(jsonArray.toJSONString(), Student.class);
        System.out.println(students);
    }

    /**
     * 复杂类型JavaBean 转为 JSONObject
     */
    @Test
    public void test17() {
        List<Student> students = Arrays.asList(new Student("zhangsan", 11),
                new Student("lisi", 12));

        Course  course = new Course("english", 1270);

        Teacher teacher = new Teacher("zhangsan", 28, course, students);

        // 方式1
        String jsonString = JSONObject.toJSONString(teacher);
        JSONObject jsonObject = JSONObject.parseObject(jsonString);
        System.out.println(jsonObject);
        // 方式2
        JSONObject jsonObject1 = (JSONObject) JSONObject.toJSON(teacher);
        System.out.println(jsonObject1);
    }

    /**
     * 复杂JSONObject 转为 JavaBean
     */
    @Test
    public void test18() {
        JSONObject jsonObject = JSONObject.parseObject(COMPLEX_JSON_STR);
        Teacher teacher1 = JSONObject.parseObject(jsonObject.toJSONString(), Teacher.class);
        System.out.println(teacher1);
    }
```





## 可变长参数

