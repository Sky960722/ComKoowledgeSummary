# JAVA的流

## 流的概念

1. 流遵循了“做什么而非怎么做”的原则

2. 流和集合的差异

   1. 流并不存储其元素
   2. 流的操作不会修改其数据源
   3. 流的操作是尽可能惰性执行的

3. 创建流的流程

   1. 创建一个流
   2. 指定将初始流转换为其他流的中间操作
   3. 应用终止操作，从而产生结果

4. ~~~
   java.util.stream.Stream<T>
   Stram<T> filter(Predicate<? super T> p) //产生一个流，其中包含当前流中满足p的所有元素
   long count() //产生当前流中元素的数量。这是一个终止操作
   
   java.util.Collection<E>
   default Stream<E> stream()
   default Stream<E> parallelStream() //产生当前集合中所有元素的顺序流或并行流
   ~~~

5. 流的创建

   1. Collection接口的stream方法

   2. 静态的Stream.of方法。该方式接受可变参数

   3. Stream.empty方法产生一个不包含任何元素的流。Stream.generate方法接受一个不包含任何引元的函数。Stream.iterate方法能产生序列，它会接受一个种子和一个函数，并且会反复地将函数应用到之前的结果上。

   4. ~~~java
      java.util.stream.Stream
      static <T> Stream<T> of(T...values) //产生一个元素为给定值的流
      static <T> Stream<T> empty() //产生一个不包含任何元素的流
      static <T> Stream<T> generate(Supplier<T> s) //产生一个无限流，它的值是通过反复调用函数s而构建的
      static <T> Stream<T> iterate(T seed,UnaryOperator<T> f) 
      static <T> Stream<T> iterate(T seed,Predicate<? super T> hasNext,UnaryOperator<T> f)//产生一个无限流
      static <T> Stream<T> ofNullable(T t) //如果t为null，返回一个空流，否则返回包含t的流
      
      java.util.Arrays
      static <T> Stream<T> stream(T[] array,int startInclusive,int endExclusive) //产生一个流，它的元素由数组中指定范围内的元素构成
      
      java.util.regex,Pattern
      Stream<String> splitAsStream(CharSequence input) //产生一个流，它的元素是输入中由该模式界定的部分
      
      java.nio.file.Files
      static Stream<String> lines(Path path)
      static Stream<String> lines(Path path,Charset cs) //产生一个流，它的元素是指定文件中的行
      
      java.util.stream.StreamSupport
      static <T> stream<T> stream(Spliterabor<T> spliterator,boolean parallel) //产生一个流，它包含了由给定的可分割迭代器产生的值。
      
      java.lang.Iterable
      Spliterator<T> spliterator() //为这个Iterable产生一个可分割的迭代器。默认实现不分割也不报告尺寸
      
      java.util.Spliterators
      static <T> Spliterator<T> spliteratorUnkownSize(Iterator<? extends T> iterator,int characteristics) //用给定的特性将一个迭代器转换为具有未知尺寸的可分割的迭代器
      ~~~

## 流的转换

1. filter、map和flatMap方法

    1. ~~~java
       java.util.stream.Stream
       Stream<T> filter(Predicate<? super T> predicate) //产生一个流，它包含当前流中所有满足谓词条件的元素
        
       <R> Stream<R> map(Function<? super T,? extend R> mapper) //产生一个六，它包含将mapper应用于当前流中所有元素产生的结果
        
       <R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper) //产生一个流，它是通过将mapper应用于当前流中所产生的结果连接到一起而获得的
       ~~~

2. 抽取子流和组合流

   1. ~~~java
      java.util.stream.Stream
      Stream<T> limit(long maxSize) //产生一个流，其中包含了当前流中最初的maxSize个元素
      
      Steam<T> skip(long n) //产生一个流，它的元素是当前流中除了前n个元素之外的所有元素
      
      Steam<T> takeWhile(Predicate<? super T> predicate) //产生一个流，它的元素是当前流中所有满足谓词条件的元素
      
      Stteam<T> dropWhile(Predicate<? super predicate>) //产生一个流，它的元素是当前流中排除不满足谓词条件的元素之外的所有元素
      
      static <T> Stream<T> concat(Stream<? extends T> a,Stream<? extends T> b) //产生一个流，它的元素是a的元素后面跟着b的元素
      ~~~

3. 其他的流的转换
   
      ~~~java
      java.util.stream.Stream
      Stream<T> distinct() //产生一个流，包含当前流中所有不同的元素
      
      Stream<T> sorted() 
      Stream<T> sorted(Comparator<? super T> comparator) //产生一个流，它的元素是当前流中的所有元素按照顺序排列的
      
      Stream<T> peek(Consumer<? super T> action) //产生一个流，它与当前流中的元素相同，在获取其中每个元素时，会将其传递给action
      ~~~
      

## 简单约束

1. 约束是一种终结操作，它们会将流约简为可以在程序中使用的非流值

2. ~~~java
   java.util.stream.Stream
   Option<T> max(Comparator<? super T> comparator)
   Option<T> min(Comparator<? super T> comparator) //分别产生这个流的最大元素和最小元素，如果这个流为空，会产生一个空的Optional对象。这些操作都是终结操作
   
   Optional<T> findFirst()
   Optional<T> findAny() //分别产生这个流的第一个和任意一个元素，如果这个流为空，会产生一个空的Optional对象
   
   boolean anyMatch(Predicate<? super T> predicate)
   boolean allMatch<Predicate<? super T> predicate)
   boolean noneMatch(Predicate<? super T> predicate) //匹配或不匹配返回true
   ~~~

## Optional类型

### 1. 定义

1. Optional<T>对象是一种包装器对象，要么包装了类型T的对象，要门没有包装任何对象

### 获取Option值

1. ~~~java
   java.util.Optional
   T orElse(T other) //产生这个Optional的值，或者在该Option为空时，产生other
   T orElseGet(Supplier<? extends T> other) //产生这个Optional的值，或者在该Optional为空时，产生调用other的结果
   <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) //产生这个Optional的值，或者在该Optional为空时，抛出调用ExceptionSupplier的结果
   T get()
   T orElseThrow() //产生这个Optional的值，或者在该Optional为空时，抛出一个NosuchElementException异常
   boolean isPresent() //如果该Optional不为空，则返回true
   ~~~

### 消费Optional值

1. ~~~java
   java.util.Optional
   void ifPresent(Consumer<? super T> action) //如果该Optional不为空，就将它的值传递给action
   void ifPresentOrElse(Consumer<? super T> action,Runnable emptyAction) //如果该Optional不为空，就将它的值传递给action，否则调用emptyAction
   ~~~

管道化Optional值

1. ~~~java
   java.util.Optional
   <U> optional<U> map(Function<? super T,? extends U> mapper) //产生一个Optional，如果当前的Optional的值存在，那么所产生的Optional的值是通过将给定的函数应用于当前的Optional的值而得到的；否则，产生一个空的Optional
   Optional<T> filter(Predicate<? super T> predicate) //产生一个Optional，如果当前的Optional的值满足给定的谓词条件，那么所产生的Optional的值就是当前Optional的值；否则，产生一个空的Optional
   Optional<T> or(Supplier<? extends Option<? extends T>> supplier) //如果当前Optional不为空，则产生当前的Optional；否则由supplier产生一个Optional
   ~~~

### 创建Optional值

1. ~~~java
   java.util.Optional
   static <T> Optional<T> of(T value)
   static <T> optional<T> ofNullable(T value) //产生一个具有给定值的Optional
   static <T> Optional<T> empty() //产生一个空Optional
   ~~~

### flatMap

1. ~~~java
   java.util.Optional
   <U> Optional<U> flatMap(Function<? super T,? extends Optional<? extends U>> mapper) //如果Optional存在，产生将mapper应用于当前Optional值所产生的结果，或者在当前Optinal为空时，返回一个空Optinal
   ~~~

### 收集结果

1. ~~~java
   java.util.Stream.BaseStream 
   Iterator<T> iterator() //c产生一个用于获取当前流中各个元素的迭代器
   
   java.util.stream.Stream
   void forEach(consumer<? super T> action) //在流的每个元素上调用action
   Object[] toArray() 
   <A> A[] toArray(InterFunction<A[]> generator) //产生一个对象数组，或者再将引用A[]::new传递给构造器时，返回一个A类型的数组
   <R,A> R collect(Coleector<? super T,A,R> collector) //使用给定的收集器来收集当前流中的元素。COllectors类有用于多种收集器的工厂方法
   
   long getCount() //产生汇总后的元素的个数
   (int|long|double) getSum() 
   double getAverage() //产生汇总后的元素的总和或平均值，或者在没有任何元素时返回0
   (int|long|double) getMax()
   (int|long|double) getMin() //产生汇总后的元素的最大值和最小值，或者在没有任何元素时，产生(Integer|long|Double).(MAX|MIN)_VALUE
   ~~~

### java.util.stream.Collertors有各种收集器

1. ~~~java
   java.util.stream.Collectors
   static <T> Collector<T,?,List<T>> toList()
   static <T> Collector<T,?,Set<T>> toSet() //产生一个元素收集到列表或集合中的收集器
   ~~~

### 基本流类型

1. 基本类型包装到包装器对象是低效的，因此，有专门的基本类型流。

### 并行流

1. 流使并行处理块操作变得很容易。但是，必须要有一个并行流，可以通过Collection.parallelStream()或者parallel方法获得
2. 操作要点
   1. 并行化会导致大量的开销，只有面对非常大的数据集才划算
   2. 只有在底层的数据源可以被有效地分割为多个部分时，将流并行化才有意义
   3. 并行流使用地线程池可能会因诸如文件I/O或网络访问这样的操作被组塞而饿死
