先来一张 JVM 中的内存模型 。 

 

在Java 虚拟机原理这本书中介绍了类会被初始化的 5 种情况 。

1 遇到 new getstatic putstatic 和 invokestatic 这 4 条指令时，这4 条指定分别对应使用 new 关键字创建对象，读取和设置一个静态字段（被 final 修饰的静态字段除外，因为已经在编译期间把结果放到常量池中了）和调用一个类的静态方法 。

2 对类进行反射调用时 。

3 当其父类没有被初始化时，要初始化父类 。

4 当虚拟机启动时，用户需要指定一个包含 main 方法的类，虚拟机会优先初始化这个类。

5 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果 REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

对照着这些再来看一下我们经常混淆的类中结构的加载顺序 ，可能会有更加深刻的认识 。

关于类中结构的加载顺序 ，首次创建对象时 ，类中的静态方法 / 静态字段首次被访问时 ，Java 解释器必须先查找类路径 ，以定位.class 文件；然后载入 .class (这将创建一个 Class 对象) ，有关静态初始化的所有动作都会执行 。因此 ，静态初始化只在 Class 对象首次加载的时候进行一次 。当用 new 创建对象时 ，首先在堆上为对象分配足够的存储空间 。然后将堆中的属性分别赋上默认的初始值 。为属性显示赋值（如果有的话） 。最后执行构造器 。

综上我们可以得出这样的结论 ，类的加载顺序整体上为 “ 父类静态—》子类静态—》父类非静态—》父类构造器—》子类非静态—》子类构造 。” 



 

下面说一些看似类被初始化 ，其实并没有的情况 。

A 通过子类应用父类静态字段 ，不会导致子类初始化 。

```
public class SuperClass {
    static {
        System.out.println("SuperClass Init !!!");
    }
    public static int value = 123;
}

public class SubClass extends SuperClass{
    static {
        System.out.println("SubClass Init !!!");
    }
}

public class Test {
    public static void main(String[] args) {
        System.out.println(SubClass.value);
    }
}

```

B 通过数组定义来引用类 ，不会触发此类的初始化 。（ 左右拖动屏幕查看代码 ）

```
public class Test{
    public static void main(String[] args) {
        //System.out.println(SubClass.value);
        SuperClass[] superClasses = new SuperClass[10];
    }
}
```
运行结果并没有打印出 ”SuperClass Init !!!” ，这里并没有触发 SuperClass 类的初始化 。这里触发了另一个名为 “ [Lcom.sun.jojo.noinitclass.SuperClass ” 的类的初始化 ，他是虚拟机自动创建的直接继承于 java.lang.Object 的子类 ，创建动作由字节码指令 newarray 触发 。

C 常量在编译期间就会调入类的常量池中 ，所以直接引用变量的类并没有被初始化 。（ 左右拖动屏幕查看代码 ）

```
public class ConstClass {
    static {
        System.out.println("ConstClass Init !!!");
    }
    public static final int value = 123;
}

public class Test{
    public static void main(String[] args) {
        //System.out.println(SubClass.value);
        //SuperClass[] superClasses = new SuperClass[10];
        System.out.println(ConstClass.value);
    }
}
// 123   PS. ConstClass Init !!! 并没有打印
```
接口的初始化和类的初始化类似 ，区别在于 5 种情况的第三种 ：子类的初始化过程中其父类必须先初始化 ，但接口初始化时不要求其父接口也进行初始化 ，只有在用到父接口时 ，才会去初始化 。

 
