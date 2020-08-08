## 类加载器

> 作用：加载.class字节码文件

### new 对象的过程

```java
public class Student {
    //私有属性
    private String name;

    //构造方法
    public Student(String name) {
        this.name = name;
    }
}
```

- 类是模板，模板是抽象的；对象是具体的，是对抽象的实例化

```java
public class classTest {
    public static void main(String[] args) {
        //s1、s2、s3为不同对象
        Student s1 = new Student("zsr");	//引用放在栈里，具体的实例放在堆里
        Student s2 = new Student("gcc");
        Student s3 = new Student("BareTH");
        System.out.println(s1.hashCode());
        System.out.println(s2.hashCode());
        System.out.println(s3.hashCode());
        //class1、class2、class3为同一个对象
        Class<? extends Student> c1 = s1.getClass();
        Class<? extends Student> c2 = s2.getClass();
        Class<? extends Student> c3 = s3.getClass();
        System.out.println(c1.hashCode());
        System.out.println(c2.hashCode());
        System.out.println(c3.hashCode());
    }
}
```

结果：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806100452.png)

通过结果发现，s1，s2，s3的hashCode是不同的，因为他们是不同的对象

c1，c2，c3的hashCode是相同的，因为他们是同一个类，类是抽象的



```java
public class classLoaderTest {
    public static void main(String[] args) {
        Student student = new Student("zyx");
        Class<? extends Student> c = student.getClass();
        ClassLoader classLoader = c.getClassLoader();

        // AppClassLoader
        System.out.println(classLoader);
        // ExtClassLoader
        System.out.println(classLoader.getParent());
        // null (c++编写，无法识别)
        System.out.println(classLoader.getParent().getParent());

    }
}
```

结果：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806100342.png)

new对象的流程

1. 首先Class Loader读取字节码.class文件，加载初始化生成Student模板类

2. 通过Student模板类new出三个对象

   ![image-20200806101022579](C:\Users\znts\AppData\Roaming\Typora\typora-user-images\image-20200806101022579.png)

### 类加载器的类别

编写一个程序：

```java
public class classLoaderTest {
    public static void main(String[] args) {
        Student student = new Student("zyx");
        Class<? extends Student> c = student.getClass();
        ClassLoader classLoader = c.getClassLoader();

        // AppClassLoader
        System.out.println(classLoader);
        // ExtClassLoader
        System.out.println(classLoader.getParent());
        // null (c++编写，无法识别)
        System.out.println(classLoader.getParent().getParent());

    }
}

```

结果：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806101210.png)



#### BoostrapCLassLoader(启动类(根)加载器)

1. c++编写，加载java核心库`java.*`，构造`拓展类加载器`和`应用程序加载器`

2. `根加载器`加载`拓展类加载器`，并且将`拓展类加载器`的父加载器设置为`根加载器`，然后再加载`应用程序加载器`，将`应用程序加载器`的父加载器设置为`拓展类加载器`

3. 根加载器存放位置

   ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806102316.png)

#### ExtClassLoader(拓展类加载器)

1. java编写，加载扩展库，开发者可以直接使用标准扩展库类加载器

2. java9之前为`ExtClassloader`，Java9以后改名为`PlatformClassLoader`

3. 存放位置

   ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806102732.png)

#### AppClassLoader(应用程序加载器)

1. `java`编写，加载程序所在的目录
2. 是Java`默认`的类加载器

#### CustomClassLoader(用户自定义类加载器)

1. `java`编写,用户自定义的类加载器,可加载指定路径的`class`文件



### 类加载的过程

1. 装载：根据查找路径找到相应的class文件，然后导入
2. 链接：
   1. 检查：检查待加载的class文件的正确性
   2. 准备：给类中的静态变量分配存储空间。(这里用到static关键字的知识)
   3. 解析：将符号引用转换成直接引用(可选)
3. 初始化：对静态变量和静态代码块执行初始化操作，这个阶段才是真正开始执行类中的字节码

### 双亲委派机制

1. 类加载器收到类加载的请求
2. 将这个请求向上委托给父类加载器去完成，一直向上委托，直到根加载器`BootstrapClassLoader`
3. 根加载器检查是否能够加载当前类，能加载就结束，使用当前的加载器；否则就抛出异常，通知子加载器进行加载；子加载器重复该步骤。

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200806105203.png)

发现报错了，这就是`双亲委派机制`起的作用，当类加载器委托到`根加载器`的时候，`String类`已经被`根加载器`加载过一遍了，所以不会再加载，从一定程度上防止了危险代码的植入!!

**作用总结**：

1. 防止重复加载同一个`.class`。通过不断委托父加载器直到根加载器，如果父加载器加载过了，就不用再加载一遍。保证数据安全。
2. 保证系统核心`.class`，如上述的`String类`不能被篡改。通过委托方式，不会去篡改核心`.class`，即使篡改也不会去加载，即使加载也不会是同一个`.class`对象了。不同的加载器加载同一个`.class`也不是同一个`class`对象。这样保证了`class`执行安全。

### 经典题目

```java
public  class One {
    public One(){
        System.out.println("one 构造");
    }
    {
        System.out.println("one 实例");
    }
    static {
        System.out.println("one 静态");
    }
}

```

```java
public class Two extends One {
    public Two(){
        System.out.println("two 构造");
    }
    {
        System.out.println("two 实例");
    }
    static {
        System.out.println("two 静态");
    }

}
```

```java
public class OneTest extends Two{
    public static void main(String[] args) {
        System.out.println("开始");
        new Two();
        new Two();
        System.out.println("结束");
    }
}
```

结果

![image-20200806144233881](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200806144233881.png)

- 分析：
  1. 代码从main方法开始执行，main方法是Demo的静态方法，就会触发Demo的类加载
  2. Two是Demo的父类，所以在加载Demo前会先加载Two，但是One又是Two的父类，就会优先加载
  3. 加载One的静态代码块输出One静态
  4. 加载Two的静态代码块输出Two静态
  5. Demo没有静态代码块就会执行main方法的内容输出开始
  6. 在构造Two的实例的时候会先构造One的实例，就会先执行One的代码块和构造方法，然后才执行Two的代码块和构造方法，所以输出One实例 One构造 Two实例 Two构造
  7. 进行第二次构造Two的时候和第一次构造一样输出One实例  One构造 Two实例 Two构造

如果我们稍微改一下代码让Demo不继承Two会发生怎样的情况

```java
public class OneTest {
    public static void main(String[] args) {
        System.out.println("开始");
        new Two();
        new Two();
        System.out.println("结束");
    }
}

```

![image-20200806160032111](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200806160032111.png)

- 我们就会发现一开始打印的是 开始 其实我们只要抓住一点类加载就是在使用到他的时候才会发生加载 这个使用包括上面说到的三点 创建实例 访问静态属性或者方法 或者继承他的类要被使用