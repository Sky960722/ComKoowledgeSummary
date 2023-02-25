# JAVA的日期和时间

## java.util.Date

1. 表示特定的瞬间，精确到毫秒

2. 构造器：

   1. Date()：使用无参构造器创建的对象可以获取本地当前时间。 

   2.  Date(long date) 

3. 常用方法 

   1. getTime():返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象 表示的毫秒数。 
   2. toString():把此 Date 对象转换为以下形式的 String： dow mon dd hh:mm:ss zzz yyyy 其中： dow 是一周中的某一天 (Sun, Mon, Tue,  Wed, Thu, Fri, Sat)，zzz是时间标准。 

4. java.text.SimpleDateFormat类

   1. java.text.SimpleDateFormat 类是一个不与语言环境有关的方式来格式化和解析日期的具体类。 

   2. 它允许进行格式化：日期 -> 文本、解析：文本 -> 日期

   3. 格式化： 

      1. SimpleDateFormat() ：默认的模式和语言环境创建对象 

      2. public SimpleDateFormat(String pattern)：该构造方法可以用参数pattern 指定的格式创建一个对象，该对象调用：

      3.  public String format(Date date)：方法格式化时间对象date 
   
   4. 解析： 
      1. public Date parse(String source)：从给定字符串的开始解析文本，以生成 一个日期。

## java.time和java.time.format

1. 在Java API中有两种人类时间，本地日期/时间和时区时间。本地日期/时间包含日期和当天的时间，但是与时区信息没有任何关联。

2. ~~~java
   java.time.LoacalDate
   static LocalDate now() //获取当前的LocalDate
   static LocalDate of(int year,int month,int dayOfMonth)
   static LocalDate of(int year,Month month,intdayOfMonth) //用给定的年、月(1到12之间的整数或者Month枚举的值) 和 日(1到31之间)产生一个本地日期
   LocalDate plus(TmporalAmount amountToAdd)
   LocalDate minus(TemporalAmount amountToSubtract)
   //产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration和Period类实现了TemporalAmount接口
   LocalDate withDayOfMonth(int dayOfMonth)
   LocalDate withDayOfYear(int dayOfyear)
   LocalDate whitMonth(int month)
   LocalDate withYear(int year)
   //返回一个新的LocalDate，将月份日期，年日期，月或年修改为给定值
   ~~~
