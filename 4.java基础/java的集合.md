# JAVA的集合

## 集合框架的接口

1. Java集合框架为不同类型的集合定义了大量接口

   1. 有两个基本接口：Collection和Map。Collection继承了Iterable接口，for each循环可以出处理任何实现了Iterable接口的对象
   2. 集合框架主要有两个抽象类，AbstractCollection 和 AbstractMap
   3. 常用的集合

   | 集合类型      | 描述                             |
   | ------------- | -------------------------------- |
   | ArrayList     | 动态增长和缩减的索引序列         |
   | LinkedList    | 任何位置高效插入和删除的有序序列 |
   | ArrayDequeue  | 循环数组的双端队列               |
   | HashSet       | 没有重复元素的无序集合           |
   | TreeSet       | 有序集                           |
   | PriorityQueue | 允许高效删除最小元素的一个集合   |
   | HashMap       | 存储键/值关联的一个数据结构      |
   | TreeMap       | 键有序的一个映射                 |

   4. Map的映射视图

      1. Set<K> keySet()
      2. Collection<V> values()
      3. Set<Map.Entry<K,V> entrySet()

      这三个会分别返回3个视图，通过这些试图可以遍历Map类型的变量
