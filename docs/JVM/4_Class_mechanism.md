## 类加载机制

向前看看不到未来，因为未来不在前面，它在脚下。

### 一、什么是类加载机制？类加载到使用完全过程？

类加载机制就是：把描述类的数据加载到内存，然后对数据进行校验、转换解析和初始化。最终形成被虚拟机直接使用的java 类型。

类加载的过程：**加载、验证、准备、解析、初始化、使用、卸载**

![image-20211116214556489](https://gitee.com/dlutlgy/window_typora/raw/master/images/image-20211116214556489.png)

### 二、类加载的过程

#### 1. 加载

此步骤需要做的是

1. 通过一个类的全限定名来获取定义此类的二进制字节流。 
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。 
3. 在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口。

#### 2. 验证

此步骤是为了确定加载到内存中的数据是安全的，不能危害虚拟机的安全。

这部分主要是对 **文件格式验证、元数据验证、字节码验证、和符号引用验证** 

1. 文件格式验证

主要是验证字节流是否符合 Class 文件格式的规范，并且能被当前虚拟机进行处理。

+ 是否以魔数开头
+ 主次版本号是都在当前虚拟机中接受范围之内
+ 常量池中的常量是是否有不被支持的常量类型
+ ......

2. 元数据验证

此阶段主要是对字节码描述的信息进行语义分析，确保符合要求。

+ 这个类是否有父类
+ 这个类的父类是不是继承了不允许继承的类
+ 如果该类不是抽象类，那么该类是不是实现了其父类或者接口中要求的所有方法。
+ ......

3. 字节码验证

字节码验证时验证过程中最复杂的一个阶段，主要是通过数据流分析和控制流分析，确定程序是合法的，是符合逻辑的。

+ 保证任意时刻操作数栈上的类型与指令代码序列都能配合工作。
+ ......

4. 符号引用验证

此阶段的主要目的是确保解析行为能正常执行

+ 符号引用中通过字符串描述的全限定名是否能找到对应的类。
+ ......

#### 3. 准备

准备阶段进行内存分配的仅仅是类变量，实例变量实在对象初始化的时候才分配。这里静态变量初始化 0 值，如果是有final 关键字的常量，则直接赋值。

#### 4.解析

解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程，符号引用就理解为一个标示，而在直接引用直接指向内存中的地址。

#### 5.初始化

初始化阶段就是去执行类构造器中的 <clinit>()方法的过程。



### 三、类加载器的种类

1. 启动类加载器

C++语言实现，他独立于虚拟机外部，他只加载 <JAVA_HOME>\lib 目录或者是 -Xbootclasspath 参数指定的路径存放的。启动类加载器是没法办法被Java程序直接调用的。如果强制调用，会显示 null 。

2. 扩展类加载器

扩展类加载器负责加载 <JAVA_HOME>\lib\ext 目录中，或者被 java.ext.dirs 系统遍历所指定的路径中所有的类库。允许被用户扩展 Java SE 的功能。

3. 应用程序类加载器（系统类加载器）

他负责加载类路径下的所有类库，开发者可以直接使用。在没有自己定义类加载器的情况下，这就是默认的类加载器。

4. 用户自定义类加载器

通过继承 java.lang.ClassLoader 类的方式实现。

![image-20211120143543051](https://gitee.com/dlutlgy/window_typora/raw/master/images/image-20211120143543051.png)

### 四、什么是双亲委派模型

如果一个类加载器收到了类加载的请求，它首先不会自己去加载这个类，而是把这个请求委派给父类加载器去完成，每一层的类加载器都是如此，这样所有的加载请求都会被传送到顶层的启动类加载器中，只有当父加载无法完成加载请求（它的搜索范围中没找到所需的类）时，子加载器才会尝试去加载类。

### 五、什么情况下会对类立即执行初始化（在类的加载阶段）

共六种情况

+ 遇到new、getstatic、putstatic 或 invokestatic 这四条字节码指令时，如果类型没有进行过初始化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有： 
  + 使用new关键字实例化对象的时候。 
  + 读取或设置一个类型的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外） 的时候。 
  + 调用一个类型的静态方法的时候。
+ 使用 java.lang.reflect 包的方法对类型进行反射调用的时候，如果类型没有进行过初始化，则需 要先触发其初始化。
+ 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。 
+ 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先 初始化这个主类。
+ 当使用 JDK 7新加入的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解 析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial四种类型的方法句 柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。 
+ 当一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有 这个接口的实现类发生了初始化，那该接口要在其之前被初始化。