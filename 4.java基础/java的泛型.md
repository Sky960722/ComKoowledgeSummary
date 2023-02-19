# JAVA的泛型

## 泛型好处

1. 泛型程序设计意味着编写的代码可以对多种不同类型的对象重用

## 泛型类，泛型方法

1. 泛型类是一有一个或多个类型变量的类
2. 泛型方法是一个带有类型参数的方法
3. 区别从尖括号和类型变量看出
4. 泛型方法可以在普通类中定义，也可以在泛型类中定义

## 类型变量的限定

1. 有时，类或方法需要对类型变量加以约束

~~~java
class ArrayAlg{
    public static <T extends Comparable> T min(T[] a){
        if(a == null || a.length ==0) return null;
        T smallest = a[0];
        for(int i = 1;i < a.length;i++){
            if(smallest.compareTo(a[i]) > 0) smallest = a[i];
        }
        return smallest;
    }
}
~~~

2. 上述代码说明T只能是实现了Comparable接口的类

## 泛型代码和虚拟机

### 类型擦除

1. 无论何时定义一个泛型类型，都会自动提供一个相应的原始类型。这个原始类型的名字就是去掉类型参数后的泛型类型名。

2. 转换泛型表达式

   Pair<Employee> buddies = ....;

   Employee buddy = buddies.getFirst();

   buddies返回的其实是Object类型，底层会强制转换

3. 转换泛型方法

   ~~~java
   class DateInterval extends Pair<LocalDate>{
       public void setSecond(LocalDate second){
           if(second.compareTo(getFirst() >= 0)){
            	super.setSecond(second);   
           }
       }
   }
   ~~~

   1. 这个方法在类型擦除后，会和Pair的方法产生冲突，这时候，编译器会在DateInterval类中生成一个桥方法

      public void setSecond(Object second){setSecond ()(LocalDate) second )；}

4. 总结

   1. 虚拟机没有泛型，只有普通的类和方法
   2. 所有的类型参数都会替换为它们的限定类型
   3. 会合成桥方法来保持多态
   4. 为保持类型安全性，必要时会插入强制类型转换

### 限制与局限性

1. 不能用基本类型实例化类型参数
2. 运行时类型查询只适用于原始类型
3. 不能创建参数化类型的数组
4. Varargs警告
   1. Java虽然不支持泛型类型的数组。但是向参数个数可变的方法传递一个泛型类型，却只会警告
      1. 增加注解@SuppressWarnings("unchecked")
      2. @SafeVarargs
5. 不能实例化e类型变量
6. 不能构造泛型数组
7. 泛型类的静态上下文中类型变量无效
8. 不能抛出或捕获泛型类的实例
9. 注意擦除后的冲突

## 泛型类型的继承规则

### 通配符概念

1. 在通配符类型中，允许类型参数发生变化

2. 带有超类型限定的通配符允许你写入一个泛型对象，带有子类型限定的通配符允许你读取一个泛型对象

   子类型：Pair<? extends Employee>

   超类型：Pair<? super Manager>

3. 超类型还有一种应用，用于接口上。表示的使用T类型的一个对象，或者使用T的一个超类型对象。也就是说，这个对象传参进去，既可以是他自己实现的一个接口类，也可以是他的父类，因为他会继承父类接口的方法。

   public staic <T extend Comparable <? super T> T min(T[] a)...

4. 无限定通配符：只能返回给Object，或者传参是null。
