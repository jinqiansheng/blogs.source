---
title: java8新特性
date: 2019-11-12 16:19:36
tags:
 - Java
---
Java语言、编译器、类库、开发工具与JVM（Java虚拟机）带来了大量新特性。这些特性将颠覆以往的写代码习惯，简化开发量。

### Lambda表达式
**示例：**
``` java
(parameters) -> expression || (parameters) ->{ statements; }
```
**左侧：**
Lambda 表达式的列表参数，如getValue(T,R); 就应该是(T,R) -> xxxxx，如果无参数就直接写 () -> xxxxx，只有一个参数小括号可以不写;

**右侧：**
Lambda 表达式的方法体（Lambda体），就是对getValue(T,R)的实现，方法体有多条语句 可以用大括号括起来，如：-> {xxxxx;xxxxx;}，只有一条语句 return 和 {}都可以不写;

**<font color=#FF0000 >注意：</font>**
Lambda表达式只支持函数式接口（只有一个方法的接口）可以使用@FunctionalInterface 修饰，可以检查接口是否是函数式接口;

**四大内置核心函数：**

类名|类型|方法
:--:|:--:|:--:  
Consumer< T >|消费型接口|void accept(T t)
Supplier< T >|供给型接口|T get()
Function< T, R >|函数型接口|R apply(T t)
Predicate< T >|断言型接口|boolean test(T t)

**其他函数式接口：**

类名|参数说明|方法
:--:|:---|:--:  
BiFunction<T, U, R>|参数类型有2个，为T、U，返回值为|R R apply(T t, U u);
UnaryOperator< T >|参数为T，对参数为T的对象进行一元操作，并返回T类型结果|T apply(T t);
BinaryOperator< T >|参数为T，对参数为T得对象进行二元操作，并返回T类型得结果|T apply(T t1， T t2);
BiConsumcr(T, U)|参数为T、U，无返回值|void accept(T t, U u);
ToIntFunction< T >|参数类型为T，返回值分别为int分别计算int得函数|int applyAsInt(T value);
ToLongFunction< T >|参数类型为T，返回值分别为long分别计算long得函数|long applyAsLong(T value);
ToDoubleFunction< T >|参数类型为T，返回值分别为double分别计算double得函数|double applyAsDouble(T value);
IntFunction< R >|参数分别为int返回值为R|R apply(int value);
LongFunction< R >|参数分别为long返回值为R|R apply(long value)
DoubleFunction< R >|参数分别为double返回值为R|R apply(double value)

**方法引用：**
若Lambda体中的内容有方法已经实现了，我们可以使用方法引用（可以理解为方法引用是Lambda表达式的另外一种表现形式；

**主要有三种语法方式：**

```
对象::跟实例方法名
类::静态方法名
类::实例方法名
```

**详解：**
Lambda体中调用的方法的参数列表与返回值类型必须与函数式接口的抽象方法的参数列表和返回值类型保持一致;
若Lambda参数列表中的第一参数是实例方法的调用者，而第二个参数的实例方法的参数时，可以使用`ClassName::method;`
1. **构造器引用：**&nbsp;&nbsp;`ClassName::New;`
**<font color=#FF0000 >注意：</font>**&nbsp;需要调用的构造器的参数列表与函数式接口中的抽象方法的参数列表保持一致;
2. **数组引用:** &nbsp;&nbsp;`Type::new;`
### Stream
Stream的操作是操作的复制的数据流，不改变原数据;

**三个操作步骤:**

**1. 创建Stream**
  - 可以通过Collection系列集合提供的stream()或parallelStream();
  - 通过Arrays中的静态方法stream()获取数组流;
  - 通过Stream类中的静态方法of();
  - 通过Stream中的iterate()或generate()使用迭代的方式创建无限流; 

**2. 中间操作**
多个中间操作可以连接形成一个流水线，除非流水线触发终止操作，否则中间操作不会执行	任何处理！而是再终止操作时一次性全部处理掉，称为“惰性求值”;   

**<font color=#708090>筛选与切片：</font>**
   - filter: 接收Lambda,从流中排除某些元素;
   - limit: 阶段流，使其元素不超过给定数量;
   - skip(n)：跳过元素，返回一个扔掉前面n个元素的流，若流中元素不足n个，则返回一个空流。与limit(n)互补;
   - distinct: 筛选，通过流所生成元素的hashCode()和equals()去除重复元素;  
   
**<font color=#708090>映射：</font>**

   - map: 接收Lambda，将元素转换成其他形式或提取信息，接收一个函数作为参数，该参数会被应用到每个元素上，并将其映射成一个新元素;
   -  flatMap: 接收一个函数作为参数，将流中的每一个值都换成另一个流，然后把所有的流连接成一个流;

**<font color=#708090>排序：</font>**   
- sorted() 自然排序 按照Comparable中的compareTo 方法进行排序
- sorted(Comparator com) 定制排序

**3.  终止操作（终端操作）**
查找与匹配：
- allMatch 检查是否匹配所有元素
- anyMatch 检查是否至少匹配一个元素
- noneMatch 检查是否没有匹配所有元素
- findFirst 返回第一个元素
- findAny 返回当前流中的任意元素
- count 返回流中的元素总个数
- max 返回流中的最大值
- min 返回流中的最小值

**<font color=#FF0000 >注意：</font>**
使用stream获取流进行查找默认采用串行，parallelStream是并行查找,可以声明性的通过paraller()与sequential()进行切换并行流和顺序流;
### Optional类
Optional(java.util.Optional) 是一个容器类，代表一个值存在或者不存在，原来null表示一个值不存在，现在Optional可以更好的表达这个概念，并且可以避免空指针异常;

**常用方法：**
- Optional.of(T t) 创建一个Optional实例;
- Optional.empty() 创建一个空的Optional;
- Optional.ofNullable(T t) 若t不为null，创建Optional实例,否则创建一个空实例;
- ifPresent() 判断是否包含值;
- orElse(T t) 如果调用对象包含值，返回该值，否则返回t;
- orElseGet(Supplier s) 如果调用对象包含值返回该值，否则返回s获取的值;
- map(Function f) 如果有值对齐处理，并返回处理后的Optional，否则返回Optional.empty();
- flatMap(Function mapper) 与map类似，要求返回的值必须是Optional;

### 新的时间日期API
LocalDate 日期、LocalTime 时间、LocalDateTime 日期和时间 是属于不可变的对象，使用的是国际化的ISO-8601日历系统可以通过.now()方法获取当前时间或用of(年，月，日，时，分，秒)获取参数给定的时间;
- Instant: 时间戳 now()获取默认是UTC时区;
- Duration: 计算两个时间之间的间隔、period：计算两个日期之间的间隔
- TemporalAdjuster: 时间校正器。如将日期调整到下周等操作。TemporalAdjusters 类提供了大量的TemporalAdjuster的实现;
- DateTimeFormatter: 该类提供了日期格式化的一些常量,也可以使用该类的ofPattern(String)进行自定义;
- ZonedDate、ZonedTime、ZonedDateTime 带时区的时间，ZoneId类中包含了所有的时区信息，getAvailableZoneIds() 可以获取所有时区信息，of(id) 指定时区信息获取ZoneId对象;

**示例：**
```~
LocalDateTime ldt = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));//获取上海时区的时间;
```



