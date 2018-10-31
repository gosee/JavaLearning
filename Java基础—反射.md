# Java基础—反射
## 反射的概念
Java反射就是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；并且能改变它的属性。而这也是Java被视为动态（或准动态，为啥要说是准动态，因为一般而言的动态语言定义是程序运行时，允许改变程序结构或变量类型，这种语言称为动态语言。从这个观点看，Perl，Python，Ruby是动态语言，C++，Java，C#不是动态语言。）语言的一个关键性质。这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
**反射就是把java类中的各种成分映射成一个个的Java对象**
## 理解Class类
类的正常加载过程：Class对象的由来是将class文件读入内存，并为之创建一个Class对象。
![加载过程](https://github.com/gosee/photo/blob/master/20170513133210763.png)
* 对象照镜子后可以得到的信息：某个类的数据成员名、方法和构造器、某个类到底实现了哪些接口。对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个类的有关信息。
* Class 对象只能由系统建立对象
* 一个类在 JVM 中只会有一个Class实例
* 每个类的实例都会记得自己是由哪个 Class 实例所生成

1. Class是一个类：

    ```java
    public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
    private static final int ANNOTATION= 0x00002000;
    private static final int ENUM      = 0x00004000;
    private static final int SYNTHETIC = 0x00001000;

    private static native void registerNatives();
    static {
        registerNatives();
    }

    /*
     * Private constructor. Only the Java Virtual Machine creates Class objects.
     * This constructor is not used and prevents the default constructor being
     * generated.
     */
    private Class(ClassLoader loader) {
        // Initialize final field for classLoader.  The initialization value of non-null
        // prevents future JIT optimizations from assuming this final field is null.
        classLoader = loader;
    }
    ...
    ...
    }
```

1. Class这个类封装了什么信息？
    
    **Class是一个类，封装了当前对象所对应的类的信息**
　　 
　　 一个类中有属性，方法，构造器等，比如说有一个Person类，一个Order类，一个Book类，这些都是不同的类，现在需要一个类，用来描述类，这就是Class，它应该有类名，属性，方法，构造器等。Class是用来描述类的类
　　 Class类是一个对象照镜子的结果，对象可以看到自己有哪些属性，方法，构造器，实现了哪些接口等等
1. 对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个类的有关信息。
2. Class 对象只能由系统建立对象，一个类（而不是一个对象）在 JVM 中只会有一个Class实例

定义一个Person类
```java
    public class Person {
        String name;
        private int age;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public int getAge() {
            return age;
         }
        public void setAge(int age) {
            this.age = age;
        }

        //包含一个带参的构造器和一个不带参的构造器
        public Person(String name, int age) {
            super();
            this.name = name;
            this.age = age;
         }
        public Person() {
            super();
        }
    }
```

对象为什么需要照镜子呢？

　　　　1. 有可能这个对象是别人传过来的

　　　　2. 有可能没有对象，只有一个全类名 

　　通过反射，可以得到这个类里面的信息

```java
public class ReflectionTest {

    public static void main(String [] args) {
        Class clazz = null;

        //1.得到Class对象
        clazz = Person.class;
        //2.返回字段的数组
        Field[] fields = clazz.getDeclaredFields();
        System.out.println();  //插入断点
    }
}
```
调试信息
![](https://github.com/gosee/photo/blob/master/20181030172400.png)