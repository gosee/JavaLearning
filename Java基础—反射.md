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

获取Class对象的三种方式
* 　　通过类名获取      类名.class    
* 　　通过对象获取      对象名.getClass()
* 　　通过全类名获取    Class.forName(全类名)

```java
public class ReflectionTest {

    public static void main(String [] args) throws ClassNotFoundException {
        Class clazz = null;

        //1.得到Class对象
        clazz = Person.class;
        //2.返回字段的数组
        Field[] fields = clazz.getDeclaredFields();

        //2.通过对象名
        //这种方式是用在传进来一个对象，却不知道对象类型的时候使用
        Person person = new Person();
        clazz = person.getClass();
        //上面这个例子的意义不大，因为已经知道person类型是Person类，再这样写就没有必要了
        //如果传进来是一个Object类，这种做法就是应该的
        Object obj = new Person();
        clazz = obj.getClass();
        
        //3.通过全类名(会抛出异常)
        //一般框架开发中这种用的比较多，因为配置文件中一般配的都是全类名，通过这种方式可以得到Class实例
        String className=" com.atguigu.java.fanshe.Person";
        clazz = Class.forName(className);

        //字符串的例子
        clazz = String.class;

        clazz = "javaTest".getClass();

        clazz = Class.forName("java.lang.String");

        System.out.println();  //插入断点
    }
}
```

查阅 API 可以看到 Class 有很多方法：

* getName()：获得类的完整名字。
* getFields()：获得类的public类型的属性。
* getDeclaredFields()：获得类的所有属性。包括private 声明的和继承类
* getMethods()：获得类的public类型的方法。
* getDeclaredMethods()：获得类的所有方法。包括private 声明的和继承类
* getMethod(String name, Class[] parameterTypes)：获得类的特定方法，name参数指定方法的名字，parameterTypes 参数指定方法的参数类型。
* getConstructors()：获得类的public类型的构造方法。
* getConstructor(Class[] parameterTypes)：获得类的特定构造方法，parameterTypes 参数指定构造方法的参数类型。
* newInstance()：通过类的不带参数的构造方法创建这个类的一个对象。
　　

    ```java　　     
    public static void main(String[] args) {
        Class clazz = null;

        //1.得到Class对象
        clazz = Person.class;
        //2.通过对象名
        //这种方式是用在传进来一个对象，却不知道对象类型的时候使用
        Person person = new Person();
        clazz = person.getClass();
        //上面这个例子的意义不大，因为已经知道person类型是Person类，再这样写就没有必要了
        //如果传进来是一个Object类，这种做法就是应该的
        Object obj = new Person();
        clazz = obj.getClass();


        //3.通过全类名(会抛出异常)
        //一般框架开发中这种用的比较多，因为配置文件中一般配的都是全类名，通过这种方式可以得到Class实例
        String className = "com.jiupai.study.biz.reflect.Person";
        try {
            clazz = Class.forName(className);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        //获得类完整的名字
        className = clazz.getName();
        System.out.println("class name : " + className);

        //获得类的public类型的属性。
        Field[] fields = clazz.getFields();
        for (Field field : fields) {
            System.out.println("public field name : " + field.getName());
        }

        //获得类的所有属性。包括私有的
        Field[] allFields = clazz.getDeclaredFields();
        for (Field field : allFields) {
            System.out.println("all field name : " + field.getName());
        }

        //获得类的public类型的方法。这里包括 Object 类的一些方法
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            System.out.println("public method name : " + method.getName());
        }

        //获得类的所有方法。
        Method[] allMethods = clazz.getDeclaredMethods();
        for (Method method : allMethods) {
            System.out.println("all method name : " + method.getName());
        }

        //获得指定的属性
        Field f1 = null;
        try {
            f1 = clazz.getField("name");
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        System.out.println("field [name] : " + f1);
        //获得指定的私有属性
        Field f2 = null;
        try {
            f2 = clazz.getDeclaredField("age");
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        //启用和禁用访问安全检查的开关，值为 true，则表示反射的对象在使用时应该取消 java 语言的访问检查；反之不取消
        f2.setAccessible(true);
        System.out.println("field [age] : " + f2);

        //创建这个类的一个对象
        Object p2 = null;
        try {
            p2 = clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        //将 p2 对象的  f2 属性赋值为 Bob，f2 属性即为 私有属性 name
        try {
            f2.set(p2, 18);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        //使用反射机制可以打破封装性，导致了java对象的属性不安全。
        try {
            System.out.println("field.get(object): " + f2.get(p2));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        //获取构造方法
        Constructor[] constructors = clazz.getConstructors();
        for (Constructor constructor : constructors) {
            System.out.println("constructor:" + constructor.toString());
        }
    }
    ```

执行结果
![](https://github.com/gosee/photo/blob/master/20181101153600.png)
