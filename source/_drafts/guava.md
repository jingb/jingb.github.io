---
title: guava
tags:
---

# 目标
> 
* 集合工具类使用
* Optional使用
* 参数校验使用
* 函数式基础
  * Function Predicate
  * map filter compose reduce(fold)

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
* 高级函数
  * 函数式里的map函数(映射)，在guava中是Collections2#transform
    * **关于transform写操作问题需要注意**
  * filter函数(过滤)保留了约定俗成的名字Collections2#filter
  * guava中没有fold(reduce)函数，java 8有

