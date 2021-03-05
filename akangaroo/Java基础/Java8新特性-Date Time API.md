# Date Time API

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



## LocalDateTime

总是表示本地日期和时间

### 获取当前的日期和时间

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

### toLocalDate()等

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

### of()和parse()

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

### DateTimeFormatter

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

### 时间的链式加减

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

### 调整日期和时间

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

### with()

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

### 时间的比较

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

### Duration和Period

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



## ZonedDatetime

表示一个带时区的日期和时间

### 直接获取时区和日期时间

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

### 使用atZone指定时区

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

### 时区间的转换

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



## DateTimeFormatter

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

## Instant

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

## 时间API之间相互转换图

![image-20201110154725620](http://img.akangaroo.cn/typora/java8Time.png)









