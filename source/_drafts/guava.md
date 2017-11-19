---
title: guava
tags:
---

# 还没搞懂的东西和参考
> 
* 延迟计算(lazy evaluation), 非限制计算(nonstrict evaluation)，名调用(call by name)
  * [延迟计算和Supplier](https://my.oschina.net/bairrfhoinn/blog/142985)
  * [去哪儿progape的典型示例](http://youdang.github.io/2017/01/10/note-on-guava/)
* RxJava
  * [SlideShare 对比RxJava和java 8](https://www.slideshare.net/jpaumard/java-8-stream-api-and-rxjava-comparison)
  * [stackoverflow关于二者对比](https://stackoverflow.com/questions/30216979/difference-between-java-8-streams-and-rxjava-observables)
  * **not an alternative implementation of Java8 Streams, nor the Collection framework, 而是Reactor pattern的一种实现**
* java 8
  * Spliterator和stream的底层构造函数相关，Connecting a stream on a source。Stream API由于Spliterator模式的存在变得易扩展
  * Stream无法重用
* Supplier的接口
* Collector接口和并行处理的关系
* **Stream#collect方法和reduce方法的比对，滥用reduce可能会让流无法并行**

<!-- more -->

# 目标
> 
* 集合工具类使用
* Optional使用
* 参数校验使用
* 函数式基础
  * Function Predicate
  * map filter compose reduce(fold)

# Optional
> 
* [被误用](http://youdang.github.io/2017/01/10/note-on-guava/)

# 集合

## Multiset
> 
* Multiset允许元素重复，适用于统计某个元素的出现次数
* 没有这种结构的时候统计一段文本出现单词的次数，一般是用Map遍历整段文本来实现
* multiset.size()和elementSet().size()区别
* [参考](http://www.cnblogs.com/peida/p/Guava_Multiset.html)

## Map相关
### Multimap
> 
* 替代程序里的Map&lt;K, Collection&gt;这样复杂的结构
* [参考](http://www.cnblogs.com/peida/p/Guava_Multimap.html)

| Implementation          |   Keys 的行为类似     |  　　　Values的行为类似       |
| --------                | :----:                 | :----:    |
|　　ArrayListMultimap    |      HashMap          |         　　ArrayList  |
|　　HashMultimap(实体要实现hashCode和equals方法   |       HashMap    | 　 HashSet  |
|　　LinkedListMultimap   |      LinkedHashMap\*  |            LinkedList\*  |
|　　LinkedHashMultimap   |    LinkedHashMap      |          LinkedHashSet  |
|　　TreeMultimap         |        TreeMap        |                  TreeSet  |
|　　ImmutableListMultimap|   ImmutableMap        |         ImmutableList  |
|　　ImmutableSetMultimap |  ImmutableMap         |        ImmutableSet  |

### BiMap 
> 
* inverse方法生成一个反向视图，可以通过value找key的map，局限是value同key一样都必须是唯一的 
* 比方国家和首都的集合，这种场景恰好合适

### RangeMap
> 
* key为一段区间的map
* 比方Map的key是分数的区间，value是等级

### 工具方法
> 
* MapDifference接口方便的实现map直接的集合操作


## ClassToInstanceMap
> 
* 使用于map的key值不是某种固定的类型
* 技术角度而言相当于Map&lt;Class&lt;? extends B&gt;, B&gt;

``` java
public class ActionHandler {

    private static final ClassToInstanceMap<Action> actionMap =
         new ImmutableClassToInstanceMap.Builder<Action>().
            put(RefreshAction.class, new RefreshAction()).
            put(ZoomAction.class, new ZoomAction()).
            build();

    public static Action getActionFor(Class<? extends Action> actionClasss) {
        return actionMap.getInstance(actionClasss);
    }
}

ActionHandler.getActionFor(ZoomAction.class).setEnabled(false);

```

# 函数式基础
> 
* 基础函数
  * ` public interface Function<F, T> { T apply(F input); } `
  * Predicate接口和Function的区别就是返回值是true或false，其他一样
  * b = f(a) c = g(b) **compose(g, f) = h(a, c)**

```java
Range<Integer> age35to50 = Range.closed(35, 55);
Predicate<Person> predicate = Predicates.compose(age35to50, new Function<Person, Integer>() {
  @Override
  public Integer apply(Person input) {
  	return input.getAge();
  }
});
FluentIterable.from(personList).filter(predicate);
```
> 
* 高级函数
  * 至少接受一个函数作为参数，返回结果是个函数，则此函数成为higher-order function
  * 函数式里的map函数(映射)，在guava中是Collections2#transform
    * **所谓惰性求值，指的是在对容器内元素进行遍历或者解引用时，才会调用求值函数进行求值**
    * **关于transform延迟求值问题需要注意**
  * FluentIterable.transformAndConcat即是flatMap函数
  * filter函数(过滤)保留了约定俗成的名字Collections2#filter
  * guava中没有fold(reduce)函数，java 8有
    * 这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合，最后生成一个值比方Integer、String或者map之类的。从这个意义上说，字符串拼接、数值的sum、min、max、average 都是特殊的 reduce
  

# java8
> 
* stateless和stateful
  * map和filter这种一般是无状态的(不要在方法里操作成员变量)，从流里读一个数据然后输出结果 
  * reduce、sum、max需要内部状态(internal state)来累积结果，而结果都是bounded size有界的
  * 像sort、distinct这样的操作符和map、filter有一个最大的区别，在进行下一步操作(排序、去重)之前程序需要知道之前的所有结果，这要求存储是无界的。这样的操作是**有状态操作**

{% asset_img 1.png 操作符明细对照 %}

## 重要接口
```
public interface Collector<T, A, R> { 
    Supplier<A> supplier(); 
    BiConsumer<A, T> accumulator(); 
    Function<A, R> finisher(); 
    BinaryOperator<A> combiner(); 
    Set<Characteristics> characteristics(); 
} 
```
> 
* 以Collections.toList为例
  * supplier方法new一个初始的List
  * accumulator方法是把流里的数据处理后往结果集里填
  * combiner、characteristics和并行处理相关

## groupingBy
> 
* 多级分组的概念，生成类似于先按菜品类型分类，再按卡路里级别分类的结构 `Map<Dish.Type, Map<CaloricLevel, List<Dish>>>` TODO：类似一个二级索引的结构，和guava的table类型相似？
* partitioningBy是groupingby的一种特殊情况，相当于key是boolean类型只有组

## Collect和reduce的比较
> 
要把`Stream<Integer> stream = …………;  转成一个List<Integer>`集合，可以使用
* stream.collect(Collections.toList())
``` java
List<Integer> resultList = null;
resultList = stream.reduce(new ArrayList<Integer>(), 
    new BiFunction<List<Integer>, Integer, List<Integer>>() {
    	@Override
    	public List<Integer> apply(List<Integer> t, Integer u) {
    		t.add(u);
    		return t;
    	}
}, new BinaryOperator<List<Integer>>() {
	@Override
	public List<Integer> apply(List<Integer> t, List<Integer> u) {
		t.addAll(u);
		return t;
	}
});
```
**reduce方法旨在把两个值结合起来生成一个新值，它是一个不可变的归约**

## 正确使用并行流
> java 8 in action里1到n求和的例子

1. 
``` java
public static long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
	.limit(n)
	.reduce(0L, Long::sum);
}
```
2. 
``` java 
public static long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i <= n; i++) {
	    result += i;
    }
    return result;
} 
``` 
3.
``` java
public static long parallelSum(long n) { 
    return Stream.iterate(1L, i -> i + 1) 
                 .limit(n) 
                 .parallel() 
                 .reduce(0L, Long::sum); 
} 
```
**版本3是最慢的** 
* 一是因为有Long拆箱装箱的开销
* **二是iterate直接生成的流不易于分成多个独立块**

```java
public static long parallelRangedSum(long n) { 
    return LongStream.rangeClosed(1, n) 
                     .parallel() 
                     .reduce(0L, Long::sum); 
} 
```

### 并行流
> 
* 什么时候适合用并行流
  * 先明确：**并行化过程本身需要对流做递归划分，把每个子流的归纳操作分配到不同的线程，然后把这些操作的结果合并成一个值。但在多个内核之间移动数据的代价也很大，所以很重要的一点是要保证在内核中并行执行工作的时间比在内核之间传输数据的时间长**
  * **数据集合要易于拆解成独立的小部分**
    * 和拆解相关的Spliterator(splitable iterator)接口


> 
* 注意如下情况
  * 装箱
  * 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元素顺序的操作，它们在并行流上执行的代价非常大
  * 要考虑流背后的数据结构是否易于分解。例如，ArrayList的拆分效率比LinkedList高得多，因为前者用不着遍历就可以平均拆分，而后者则必须遍历




{% asset_img 2.png 易拆解的数据结构 %}
