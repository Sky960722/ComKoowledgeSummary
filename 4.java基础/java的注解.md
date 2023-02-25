# JAVA的注解

## 简介

1. 在Java中，注解是当作一个修饰符来使用，它被置于被注解项之前，中间没有分号。每一个注解的名称前面都加上了@符号。
2. 注解其实就是代码里的特殊标记, 这些标记可以在编译, 类加 载, 运行时被读取, 并执行相应的处理。
3. 通过使用 Annotation, 程序员 可以在不改变原有逻辑的情况下, 在源文件中嵌入一些补充信息。
4. 代 码分析工具、开发工具和部署工具可以通过这些补充信息进行验证 或者进行部署。

## 注解语法

1. 注解是由注解接口来定义的

~~~java
modifiers @interface AnnotationName
{
	elementDeclaration
	elementDeclaration
}
~~~

2. 注解有6个类型
   1. 基本类型
   2. String
   3. Class(具有一个可选的类型参数，例如Class<? extends MyClass>)
   4. enum类型
   5. 注解类型
   6. 由前面所述类型组成的数组。

## 方法

~~~java
java.lang.reflect.AnnotatedElement
boolean isAnnotationPresent(Class<? extend Annotation> annotationType)
//如果该项具有给定类型的注解，则返回true
<T extends Annotation> T getAnnotation(Class<T> annotationType) //获得给定类型的注解，如果该项不具有这样的注解，则返回null
<T extends Annotation> T[] getAnnotationByType(Class<T> annotationType) //获得某个可重复注解类型的所有注解，或者返回长度为0的数组
Annotation[] getAnnotations() //获得作用于该项的所有注解，包括继承而来的注解。如果没有出现任何注解，那么将返回一个长度为0的数组
Annotation[] getDeclaredAnnotations() //获得为该项声明的所有注解，不包含继承而来的注解。
  
java.lang.annotation.Annotation
Class<? extends Annotation> annotationType() //返回Class对象，它用于描述该注解对象的注解接口。
boolean equals(Object other) //如果other是一个实现了与该注解对象相同的注解接口的对象，并且如果该对象和other的所有元素彼此相等，那么返回true
int hashCode() //返回一个与equals方法兼容、由注解接口名以及元素值衍生而来的散列码
String toString() //返回一个包含注解接口名以及元素值的字符串表示。
~~~

## 标准注解

| 注解接口                 | 应用场合                   | 目的                                                         |
| ------------------------ | -------------------------- | ------------------------------------------------------------ |
| Deprecated               | 全部                       | 将项标记为过时的                                             |
| SuppreddWarnings         | 除了包和注解之外的所有情况 | 组织某个给定类型的警告信息                                   |
| SafeWarags               | 方法和构造器               | 断言varargs参数可安全使用                                    |
| Override                 | 方法                       | 检查该方法是否覆盖了某一个超类方法                           |
| FunctionalInterface      | 接口                       | 将接口标记为只有一个抽象方法的函数式接口                     |
| PostConstruct PreDestroy | 方法                       | 被标记的方法应该在构造之后或移除之前立即被调用               |
| Resource                 | 类、接口、方法、域         | 在类或接口上：标记为在其他地方要用的资源。在方法或域上：为“注入”而标记 |
| Resources                | 类、接口                   | 一个资源数组                                                 |
| Generated                | 全部                       |                                                              |
| Target                   | 注解                       | 指明可以应用这个注解的那些项                                 |
| Retention                | 注解                       | 指明这个注解可以保留多久                                     |
| Doucumented              | 注解                       | 指明这个注解应该包含在注解项的文档中                         |
| Inherited                | 注解                       | 指明当这个注解应用于一个类的时候，能够自动被它的子类继承     |
| Repetable                | 注解                       | 指明这个注解可以在同一个项目上应用多次                       |

1. 4个标准的meta-annotation类型

   1. Retention

      1. @Retention: 只能用于修饰一个 Annotation 定义, 用于指定该 Annotation 的生命周期, @Rentention 包含一个 RetentionPolicy 类型的成员变量, 使用 @Rentention 时必须为该 value 成员变量指定值: 
         1. RetentionPolicy.SOURCE:在源文件中有效（即源文件保留），编译器直接丢弃这种策略的 注释
         2. RetentionPolicy.CLASS:在class文件中有效（即class保留） ， 当运行 Java 程序时, JVM  不会保留注解。 这是默认值
         3. RetentionPolicy.RUNTIME:在运行时有效（即运行时保留），当运行 Java 程序时, JVM 会 保留注释。程序可以通过反射获取该注释。

   2. @Target: 用于修饰 Annotation 定义, 用于指定被修饰的 Annotation 能用于 修饰哪些程序元素。 @Target 也包含一个名为 value 的成员变量。

      | 元素类型        | 注解适用场合 | 元素类型       | 注解适用场合         |
      | --------------- | ------------ | -------------- | -------------------- |
      | AMNOTATION_TYPE | 注解类型声明 | FIELD          | 成员域(包括enum常量) |
      | PACKAGE         | 包           | PARAMETER      | 方法或构造器参数     |
      | TYPE            | 类           | LOCAL_VARIABLE | 局部变量             |
      | METHOD          | 方法         | TYPE_PARAMETER | 泪下参数             |
      | CONSTRUCTOR     | 构造器       | TYPE_USE       | 类型用法             |

      
