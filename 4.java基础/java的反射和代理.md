# JAVA的反射

## 反射的定义和作用

1. 反射库提供了一个丰富且精巧的工具集，用来编写能够动态操作Java代码的程序
2. 能够分析类能力的程序称为反射
3. 作用：
   1. 运行时分析类的能力
   2. 运行时检查对象，例如，编写一个适用于所有类的toString方法
   3. 实现泛型数组操作代码。
   4. 利用Method对象

## Class类

1. 程序运行时，Java为所有对象维护一个运行时类型标识，这个信息会跟踪每个对象所属的类。
2. 虚拟机为每个类型管理一个唯一的Class对象。因此，可以利用==运算符实现两个类对象的比较

~~~java
if (e.getClass()  == Employee.class)
~~~

3. 三种方法获得Class对象

   ~~~java
   //第一种
   Employee e;
   Class cl = e.getClass();
   
   //第二种
   var generator = new Random();
   Class cl = generator.getClass();
   String name = cl.getName();
   
   String className = "java.util.Random";
   Class cl = Class.forName(className);
   
   //第三种 class关键字
   Class cl1 = Random.class
   ~~~

4. 调用getConstructor方法生成一个Constructor类型对象

~~~java
String className = "java.util.Random";
Class cl = Class.forName(className);
Object obj = cl.getConstructor().newInstance();
~~~

5. 常用函数

~~~java
java.lang.Class
static Class forName(String className) //返回一个Class对象，表示名为className的类

Constructor getConstructor(Class...paramterTypes) //生成一个对象，描述有指定参数类型的构造器
    
java.lang.reflect.Constructor
Object newInstance(Object...params) //将params传递到构造器，来构造这个构造器声明类的一个新实例
~~~

6. 加载资源

~~~java
java.lang.Class
    URL getResource(String name) 
    InputStream getResourceAsStream(String name)
    //找到与类同一位置的资源，返回一个可以用来加载资源的URL或者输入流。如果没有找到资源，则返回null，所以不会抛出异常或者I/O错误。
~~~

## 利用反射分析类的能力

1. java.lang.reflect包中有三个类Field,Method和Constructorf分别用于描述类的字段、方法和构造器。
2. getName方法，返回字段、方法或构造器的名称
3. 代码示例

~~~java
package reflection;

import java.lang.reflect.*;
import java.security.PublicKey;
import java.util.Scanner;

public class ReflectionTest {

    public static void main(String[] args) throws ClassNotFoundException {
        String name;
        if (args.length > 0) {
            name = args[0];
        } else {
            Scanner in = new Scanner(System.in);
            System.out.println("Enter class name (e.g. java.util.Date)");
            name = in.next();
        }

        Class<?> cl = Class.forName(name);
        Class<?> superclass = cl.getSuperclass();
        String modifiers = Modifier.toString(cl.getModifiers());
        if (modifiers.length() > 0) System.out.println(modifiers + " ");
        System.out.println("class " + name);
        if (superclass != null && superclass != Object.class) System.out.println(" extends " + superclass.getName());

        System.out.print("\n{\n");
        printConstructors(cl);
        System.out.println();
        printMethods(cl);
        System.out.println();
        printFields(cl);
        System.out.println("}");


    }

    public static void printConstructors(Class cl) {
        Constructor[] constructors = cl.getDeclaredConstructors();

        for (Constructor c : constructors) {
            String name = c.getName();
            System.out.print("   ");
            String modifiers = Modifier.toString(c.getModifiers());
            if (modifiers.length() > 0) {
                System.out.print(modifiers + " ");
            }
            System.out.print(name + "(");
            Class[] paramTypes = c.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if (j > 0) System.out.print(", ");
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }

    public static void printMethods(Class cl) {
        Method[] methods = cl.getDeclaredMethods();

        for (Method m : methods) {
            Class<?> retType = m.getReturnType();
            String name = m.getName();

            System.out.print("  ");
            String modifiers = Modifier.toString(m.getModifiers());
            if (modifiers.length() > 0) System.out.print(modifiers + " ");
            System.out.print(retType.getName() + " " + name + "(");

            Class<?>[] paramTypes = m.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if (j > 0) System.out.print(", ");
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }


    }

    public static void printFields(Class cl) {
        Field[] fields = cl.getDeclaredFields();

        for (Field f : fields) {
            Class<?> type = f.getType();
            String name = f.getName();
            System.out.print("  ");
            String modifiers = Modifier.toString(f.getModifiers());
            if (modifiers.length() > 0) System.out.print(modifiers + " ");
            System.out.println(type.getName() + " " + name + ";");
        }
    }
}

~~~

4. 常用方法

~~~java
java.lang.Class
Field[] getFields()
Field[] getDeclaredFields()
//getFields方法将返回一个包含Field对象的数组，这些对象对应这个类或其超类的公共字段。getDeclaredFields返回去这个类的全部字段
Method[] getMethods()
Method[] getDeclaredMethods()
//返回包含Method对象的数组，getMethods返回所有的公共方法，包括从超类继承的。getDeclaredMethods返回这个类或接口的全部方法，但不包括由超类继承的方法
Constructor[] getConstructors()
Constructor[] getDeclardConstructors()
//返回包含Constructor对象的数组，前者公共构造器，后者全部构造器
String getPackageName()
//返回这个类的包名，如果这个类是一个数组类型，返回元素类型所属的包，是基本类型，则返回"java.lang"
java.lang.reflect.Field
java.lang.reflect.Method
java.lang.reflect.Constructor
Class getDeclaringClass()
//返回Class对象，表示定义了这个构造器、方法或字段的类
Class[] getExceptionTypes()
//返回一个Class对象数组，其中各个对象表示这个方法所抛出异常的类型
int getModifiers()
//返回一个整数,描述这个构造器，方法或字段的修饰符。使用Modifier类中的方法来分析这个返回值
String getName()
//返回一个表示构造器名、方法名或字段名的字符串
Class[] getParamterTypes()
//返回一个Class对象数组，其中各个对下给你表示参数的类型
Class getReturnType()
//返回一个用于表示返回类型的Class对象

java.lang.reflect.Modifier
static String toString(int modifiers)
static boolean isAbstract(int modifiers)
static boolean isFinal(int modifiers)
static boolean isInterface(int modifiers)
static boolean isNative(int modifiers)
static boolean isPrivate(int modifiers)
static boolean isProtected(int modifiers)
static boolean isPublic(int modifiers)
static boolean isStatic(int modifiers)
static boolean isStrict(int modifiers)
static boolean isStrict(int modifiers)
static boolean isSynchronized(int modifiers)
static boolean isVolatile(int modifiers)
//这些方法检测modifiers值与参数对应的二进制位
~~~

## 访问反射的对象

1. 代码

~~~java
package reflection;

import java.lang.reflect.AccessibleObject;
import java.lang.reflect.Array;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.ArrayList;

public class Objectanalyzer {

    private ArrayList<Object> visited = new ArrayList<>();

    public String toString(Object obj) throws IllegalAccessException {
        if(obj ==null)return "null";
        if(visited.contains(obj)) return "...";
        visited.add(obj);
        Class<?> cl = obj.getClass();
        if(cl == String.class) return (String) obj;
        if(cl.isArray()){
            String r = cl.getComponentType() + "[]{";
            for(int i = 0; i < Array.getLength(obj);i++){
                if(i >0) r += ",";
                Object val = Array.get(obj,i);
                if(cl.getComponentType().isPrimitive()) r +=val;
                else r+=toString(val);
            }
            return r +"}";
        }

        String r = cl.getName();

        do{
            r += "[";
            Field[] fields = cl.getDeclaredFields();
            AccessibleObject.setAccessible(fields,true);
            for(Field f : fields){
                if(!Modifier.isStatic(f.getModifiers())){
                    if(!r.endsWith("[")) r+=",";
                    r += f.getName() +"=";
                    Class t = f.getType();
                    Object val = f.get(obj);
                    if(t.isPrimitive()) r += val;
                    else  r += toString(val);
                }
            }
            r += "]";
            cl = cl.getSuperclass();
        }while (cl != null);

        return r;
    }
}

~~~

~~~java
package reflection;

import java.util.ArrayList;

public class ObjectAnalyzerTest {

    public static void main(String[] args) throws IllegalAccessException {
        ArrayList<Integer> square = new ArrayList<>();
        for(int i = 1;i <= 5;i++){
            square.add(i *i);
        }
        System.out.println(new Objectanalyzer().toString(square));
    }
}
~~~

2. 常用方法

~~~java
java.lang.reflect.AccessibleObject
void setAccessible(boolean flag)
//设置或取消这个可访问对象的可访问标志
void setAccessible(boolean flag)
boolean trySetAccessible()
//为这个可访问的对象设置可访问标志，如果拒绝访问则返回false
boolean isAccessible()]
//得到这个可访问对象的可访问标志值
static void setAccessible(AccessibleObject[] array,boolean flag)
//这是一个便利方法，用于设置一个对象数组的可访问标志

java.lang.reflect.Field
Object get(Object obj)
//返回obj对象中用这个Field对象描述的字段的值
void set(Object obj,Object newValue)
//将obj对象中这个Field对象描述的字段设置为一个新值
~~~

## 生成泛型数组代码

1. java.lang.reflect包中的Array类允许动态创建数组。

2. 代码示例

~~~java
public class CopyOfTest {

    public static void main(String[] args) {
        int[] a = {1,2,3};
        a = (int[]) goodCopyOf(a,10);
        System.out.println(Arrays.toString(a));

        Object[] b ={1,2,3};
        System.out.println(Arrays.toString(b));

    }

    public static Object goodCopyOf(Object a,int newLength){
        Class<?> cl = a.getClass();
        if(!cl.isArray())return null;
        Class<?> componentType = cl.getComponentType();
        int length = Array.getLength(a);
        Object newArray = Array.newInstance(componentType, newLength);
        System.arraycopy(a,0,newArray,0,Math.min(length,newLength));
        return newArray;
    }
}
~~~

2. 常用代码

~~~java
java.lang.reflect.Array
static Object get(Object array,int index)
static xxx getXxx(Object array,int index)
//将返回存储在给定数组中给定索引位置上的值
static void set(Object array,int index,Object newValue)
static SetXxx(Obectt array,int index,xxx newValue)
//将一个心智存储到给定数组中的给定位置
static int getLength(Object array)
//返回给定数组的长度
static Object newInstance(Class componectType,int length)
static Object newInstance(Class componectType,int[] lengths)
//返回一个有给定类型、给定大小的新数组
~~~

## 调用任意方法和构造器

1. 代码示例

~~~java
public class MethodTableTest {


    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException,
            IllegalAccessException {
        Method square = MethodTableTest.class.getMethod("square", double.class);
        Method sqrt = Math.class.getMethod("sqrt", double.class);

        printTable(1,10,10,square);
        printTable(1,10,10,sqrt);
    }

    public static double square(double x) {
        return x * x;
    }

    public static void printTable(double from, double to, int n, Method f) throws InvocationTargetException,
            IllegalAccessException {
        System.out.println(f);

        double dx = (to - from) / (n - 1);

        for (double x = from; x <= to; x += dx) {
            double y = (Double) f.invoke(null, x);
            System.out.printf("%10.4f | %10.4f%n", x, y);
        }
    }
}
~~~

2. 常用方法

~~~java
java.lang.reflect.Method
public Object invoke(Object implicitParameter,Object[] explicitParameters)
//调用这个对象描述的方法，传入给定参数，并返回方法的返回值。对于静态方法，传入null作为隐式参数
~~~

# JAVA的代理

1. 用途：利用代理可以在运行时创建实现了一组给定接口的新类。

2. 代理类的创建

   1. 代理类必须实现InvocationHandler接口

   2. ~~~java
      Object invoke(Object proxy,Method method,Object[] args)
      ~~~

   3. 用Proxy类的newProxyInstance方法创建一个代理对象，这个方法有三个参数

      1. 一个类加载器(class loader)
      2. 一个Class对象数组，每个元素对应需要实现的各个接口
      3. 一个调用处理器

3. 代理类的特性

   1. 所有的代理类都扩展了Proxy类。一个代理类只有一个实例字段——即调用处理器，它在Proxy超类中定义，完成代理对象任务所需要的任何额外数据都必须存储在调用处理器中

   2. 对于一个特定的类加载器和预设的一组接口来说，只能有一个代理类。

   3. ~~~java
      java.lang.reflect.InvocationHandler
      Object invoke(Object proxy,Method method,Object[] args) //定义这个方法包含一个动作
      
      java.lang.reflect.Proxy
      static Class<?> getProxyClass(ClassLoader loader,Class<?>... interfaces)//返回实现指定接口的代理类
      static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler handler) //构造实现指定接口的代理类的一个新实例。所有方法都调用给定处理器对象的invoke方法
      static boolean isProxyClass(Class<?> cl) //如果cl是一个代理类则返回true
      ~~~

