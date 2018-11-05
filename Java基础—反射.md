# Java基础—反射
## 反射的概念
Reflection（反射）是Java被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的內部信息，并能直接操作任意对象的内部属性及方法。
Java反射机制主要提供了以下功能：    
* 在运行时构造任意一个类的对象
* 在运行时获取任意一个类所具有的成员变量和方法
* 在运行时调用任意一个对象的方法（属性）
* 生成动态代理

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
　　 
    * 一个类中有属性，方法，构造器等，比如说有一个Person类，一个Order类，一个Book类，这些都是不同的类，现在需要一个类，用来描述类，这就是Class，它应该有类名，属性，方法，构造器等。Class是用来描述类的类
    * Class类是一个对象照镜子的结果，对象可以看到自己有哪些属性，方法，构造器，实现了哪些接口等等
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

## 方法-Method

```java
    public class MethodTest {
    public static void main(String [] args) throws Exception{
        Class clazz = Class.forName("com.jiupai.study.biz.reflect.Person");

        //
        //1.获取方法
        //  1.1 获取取clazz对应类中的所有方法--方法数组（一）
        //     不能获取private方法,且获取从父类继承来的所有方法
        Method[] methods = clazz.getMethods();
        for(Method method:methods){
            System.out.print(" "+method.getName());
        }
        System.out.println();

        //
        //  1.2.获取所有方法，包括私有方法 --方法数组（二）
        //  所有声明的方法，都可以获取到，且只获取当前类的方法
        methods = clazz.getDeclaredMethods();
        for(Method method:methods){
            System.out.print(" "+method.getName());
        }
        System.out.println();

        //
        //  1.3.获取指定的方法
        //  需要参数名称和参数列表，无参则不需要写
        //  对于方法public void setName(String name) {  }
        Method method = clazz.getDeclaredMethod("getName");
        System.out.println(method);
        //  而对于方法public void setAge(int age) {  }
        method = clazz.getDeclaredMethod("setAge", int.class);
        System.out.println(method);
        //  这样写是获取不到的，如果方法的参数类型是int型
        //  如果方法用于反射，那么要么int类型写成Integer： public void setAge(Integer age) {  }
        //  要么获取方法的参数写成int.class

        //
        //2.执行方法
        //  invoke第一个参数表示执行哪个对象的方法，剩下的参数是执行方法时需要传入的参数
        Object obje = clazz.newInstance();
        method.invoke(obje, 2);

        //如果一个方法是私有方法，第三步是可以获取到的，但是这一步却不能执行
        //私有方法的执行，必须在调用invoke之前加上一句method.setAccessible（true）;
    }
}
```

自定义一个方法
* 把类对象和类方法名作为参数，执行方法
* 把全类名和方法名作为参数，执行方法

```java
/**
     *
     * @param obj: 方法执行的那个对象.
     * @param methodName: 类的一个方法的方法名. 该方法也可能是私有方法.
     * @param args: 调用该方法需要传入的参数
     * @return: 调用方法后的返回值
     *
     */
    public Object invoke(Object obj, String methodName, Object ... args) throws Exception{
        //1. 获取 Method 对象
        //   因为getMethod的参数为Class列表类型，所以要把参数args转化为对应的Class类型。

        Class [] parameterTypes = new Class[args.length];
        for(int i = 0; i < args.length; i++){
            parameterTypes[i] = args[i].getClass();
            System.out.println(parameterTypes[i]);
        }

        Method method = obj.getClass().getDeclaredMethod(methodName, parameterTypes);
        //如果使用getDeclaredMethod，就不能获取父类方法，如果使用getMethod，就不能获取私有方法
        //2. 执行 Method 方法
        //3. 返回方法的返回值
        return method.invoke(obj, args);
    }

    /**
     * @param className: 某个类的全类名
     * @param methodName: 类的一个方法的方法名. 该方法也可能是私有方法.
     * @param args: 调用该方法需要传入的参数
     * @return: 调用方法后的返回值
     */
    public Object invoke(String className, String methodName, Object ... args){
        Object obj = null;

        try {
            obj = Class.forName(className).newInstance();
            //调用上一个方法
            return invoke(obj, methodName, args);
        }catch(Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```
这种反射实现的主要功能是可配置和低耦合。只需要类名和方法名，而不需要一个类对象就可以执行一个方法。如果我们把全类名和方法名放在一个配置文件中，就可以根据调用配置文件来执行方法

## 字段-Field 

```java
public class FiledTest {
    public static void main(String[] args) throws Exception {
        String className = "com.jiupai.study.biz.reflect.Person";
        Class clazz = Class.forName(className);

        //1.获取字段
        //  1.1 获取所有字段 -- 字段数组
        //     可以获取公用和私有的所有字段，但不能获取父类字段
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.print(" " + field.getName());
        }
        System.out.println();

        //  1.2获取指定字段
        Field field = clazz.getDeclaredField("name");
        System.out.println(field.getName());

        Person person = new Person("ABC", 12);

        //2.使用字段
        //  2.1获取指定对象的指定字段的值
        Object val = field.get(person);
        System.out.println(val);

        //  2.2设置指定对象的指定对象Field值
        field.set(person, "DEF");
        System.out.println(person.getName());

        //  2.3如果字段是私有的，不管是读值还是写值，都必须先调用setAccessible（true）方法
        //     比如Person类中，字段name字段是公用的，age是私有的
        field = clazz.getDeclaredField("age");
        field.setAccessible(true);
        System.out.println(field.get(person));
    }
}
```

## 构造器-Constructor

```java
    public class ConstructorTest {
    public static void main () throws Exception {
        String className = "com.atguigu.java.fanshe.Person";
        Class<Person> clazz = (Class<Person>) Class.forName(className);

        //1. 获取 Constructor 对象
        //   1.1 获取全部
        Constructor<Person>[] constructors =
                (Constructor<Person>[]) Class.forName(className).getConstructors();

        for(Constructor<Person> constructor: constructors){
            System.out.println(constructor);
        }

        //  1.2获取某一个，需要参数列表
        Constructor<Person> constructor = clazz.getConstructor(String.class, int.class);
        System.out.println(constructor);

        //2. 调用构造器的 newInstance() 方法创建对象
        Object obj = constructor.newInstance("zhagn", 1);
    }
}
```

## 注解 -- Annotation
定义一个Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(value={ElementType.METHOD})
public @interface AgeValidator {
    public int min();
    public int max();
}
```

Person的方法setAge加注解
```java
@AgeValidator(min=18,max=35)
    public void setAge(int age) {
        this.age = age;
    }
```
 
通过反射的方式为属性赋值，才能获取到注解
如果在程序中要获取注解，然后获取注解的值进而判断我们赋值是否合法，那么类对象的创建和方法的创建必须是通过反射而来的

 
```java
public class AnnotationTest {
    public static void main (String [] args) throws Exception {
        Person person = new Person();
        person.setAge(100);
        System.out.println(person.getAge());

        String className = "com.jiupai.study.biz.reflect.Person";

        Class clazz = Class.forName(className);
        Object obj = clazz.newInstance();

        Method method = clazz.getDeclaredMethod("setAge", int.class);
        int val = 20;

        //获取指定名称的注解
        Annotation annotation = method.getAnnotation(AgeValidator.class);
        if(annotation != null){
            if(annotation instanceof AgeValidator){
                AgeValidator ageValidator = (AgeValidator) annotation;
                if(val < ageValidator.min() || val > ageValidator.max()){
                    throw new RuntimeException("年龄非法");
                }
            }
        }
        method.invoke(obj, 20);
        System.out.println(obj);
    }
}
```

## 小结
1. Class: 是一个类; 一个描述类的类.
    封装了描述方法的 Method,
    描述字段的 Filed,
    描述构造器的 Constructor 等属性.

2. 如何得到 Class 对象:
    *   Person.class
    *   person.getClass()
    *   Class.forName("com.atguigu.javase.Person")
  
3. 关于 Method:
    * 如何获取 Method:
        * getDeclaredMethods: 得到 Method 的数组.
        * getDeclaredMethod(String methondName, Class ... parameterTypes)
  
    * 如何调用 Method
        *   如果方法时 private 修饰的, 需要先调用 Method 的　setAccessible(true), 使其变为可访问
        *   method.invoke(obj, Object ... args);
  
4. 关于 Field:
    *   如何获取 Field: getField(String fieldName)
    *   如何获取 Field 的值: 
        * setAccessible(true)
        * field.get(Object obj)
    *   如何设置 Field 的值:
  　　　　field.set(Obejct obj, Object val)

5. 了解 Constructor 和 Annotation 