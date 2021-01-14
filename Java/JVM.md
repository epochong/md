# JVM

### Java内存结构

![image-20200224115031281](/Users/wangchong/Library/Application Support/typora-user-images/image-20200224115031281.png)

#### Java堆（Heap）

- 是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。

  - 此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。


#### 方法区（Method Area）

- 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。

  - 用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器（JIT编译器）编译后的代码等数据。
  - 运行时常量池：程序中的字面量（literal）如直接书写的100、”hello”和常量，符号引用都是放在常量池。
  - 具有动态性，用的比较多的就是String类的intern()方法
  - JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代的结论，取而代之的是meta Space。

方法区由两个区域组成：
1 、永久代：这个区域会存储包括类定义、结构、字段、方法（数据及代码）以及常量在内的类相关数据。它可以通过（以下两个是非堆区配置参数）-XX:PermSize及-XX:MaxPermSize来进行调节。若永久代(Perm Gen)空间用完，会导致java.lang.OutOfMemoryError: PermGenspace的异常。而且从JDK8开始，永久代被元空间所取代。
2 、存放数据
 ①方法区存储的是每个class的信息:
（1）类加载器引用(ClassLoader)
（2）运行时常量池：包含所有常量、字段引用、方法引用、属性
（3）字段数据：每个字段的名字、类型(如类的全路径名、类型或接口) 、修饰符（如public、abstract、final）、属性
（4）方法数据：每个方法的名字、返回类型、参数类型(按顺序)、修饰符、属性
（5）方法代码：每个方法的字节码、操作数栈大小、局部变量大小、局部变量表、异常表和每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

#### 程序计数器（Program Counter Register）

- 是一块较小的内存空间，存放当前线程所执行的字节码**行号指示器**。

- 字节码解释器工作时，通过改变这个计数器的值来选取下一个需要执行的字节码指令、分支、循环、跳转、异常处理、线程恢复等基础功能都需要这个计数器来完成。

#### JVM栈（JVM Stacks）

- 与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

  - 每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口、基本数据类型的变量，一个对象的引用，函数调用等信息。

#### 本地方法栈（Native Method Stacks）

本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。

Sun HotSpot虚拟机把本地方法和JVM虚拟机合二为一。

----

补充1：较新版本的Java（从Java 6的某个更新开始）中，由于JIT编译器的发展和”逃逸分析”技术的逐渐成熟，栈上分配、标量替换等优化技术使得对象一定分配在堆上这件事情已经变得不那么绝对了。

补充2：运行时常量池相当于Class文件常量池具有动态性，Java语言并不要求常量一定只有编译期间才能产生，运行期间也可以将新的常量放入池中，String类的intern()方法就是这样的。

### meta Space

#### 1.8之前

JVM分区可以分为线程共有——新生代、老年代、永久代，线程私有—虚拟机栈、本地方法栈、程序计数器。
触发GC的区域可以是新生代、老年代，可能发生OOM的区域：堆、虚拟机栈（例如循环递归）、永久代（Class加载过多）、本地方法栈

#### 1.8

是什么

JDK8 HotSpot JVM 将移除永久区（Permanent Generation space），使用本地内存来存储类元数据信息并称之为：元空间（Metaspace）。

Metaspace 容量：默认情况下，类元数据只受可用的本地内存限制（容量取决于是32位或是64位操作系统的可用虚拟内存大小）。新参数（MaxMetaspaceSize）用于限制本地内存分配给类元数据的大小。如果没有指定这个参数，元空间会在运行时根据需要动态调整。

回收

对于僵死的类及类加载器的垃圾回收将在元数据使用达到“MaxMetaspaceSize”参数的设定值时进行。适时地监控和调整元空间对于减小垃圾回收频率和减少延时是很有必要的。持续的元空间垃圾回收说明，可能存在类、类加载器导致的内存泄漏或是大小设置不合适。

为什么

这意味着不会再有java.lang.OutOfMemoryError: PermGen问题，也不再需要你进行调优及监控内存空间的使用。PermGen空间状况：这部分内存空间将全部移除。JVM的参数：PermSize 和 MaxPermSize 会被忽略并给出警告（如果在启用时设置了这两个参数）。

请一定要牢记，这个新特性也不能消除类和类加载器导致的内存泄漏。

JVM 种类有很多，比如 Oralce-Sun Hotspot, Oralce JRockit, IBM J9, Taobao JVM等等。需要注意的是，PermGen space是Oracle-Sun Hotspot才有，JRockit以及J9是没有这个区域。

元空间的使用情况可以从HotSpot1.8的详细GC日志输出中得到。

在设置了MaxMetaspaceSize的情况下，该空间的内存仍然会耗尽，进而引发“java.lang.OutOfMemoryError: Metadata space”错误。因为类加载器的泄漏仍然存在，因为Java不希望无限制地消耗本机内存，因此设置一个类似于MaxMetaspaceSize的限制。

> java.lang.OutOfMemoryError: Metaspace

### 类的生命周期

类的生命周期包括这几个部分，加载、连接、初始化、使用和卸载，其中前三步是类的加载的过程,如下图； 

![image-20200227121826631](/Users/wangchong/Library/Application Support/typora-user-images/image-20200227121826631.png)

- 加载，查找并加载类的二进制数据，在Java堆中也创建一个java.lang.Class类的对象

- 连接，连接又包含三块内容：验证、准备、解析 。 

  1）验证，文件格式、元数据、字节码、符号引用验证； 

  2）准备，为类的静态变量分配内存，并将其初始化为默认值；

  3）解析，把类中的符号引用转换为直接引用

- 初始化，为类的静态变量赋予正确的初始值

- 使用，new出对象程序中使用

- 卸载，执行垃圾回收

### Java跨平台

Java“跨平台”特性的实现原理如下：*.java文件，经过Java编译器编译，形成字节码文件——*.class，这种.class文件可以运行在JVM上，而JVM可以被安装到不同的平台，这就带来了Java的跨平台特性。

JVM也是一个软件，不同的平台有不同的版本。我们编写的Java源码，编译后会生成一种 .class 文件，称为字节码文件。Java虚拟机就是负责将字节码文件翻译成特定平台下的机器码然后运行。也就是说，只要在不同平台上安装对应的JVM，就可以运行字节码文件，运行我们编写的Java程序。

注意：编译的结果不是生成机器码，而是生成字节码，字节码不能直接运行，必须通过JVM翻译成机器码才能运行。不同平台下编译生成的字节码是一样的，但是由JVM翻译成的机器码却不一样。

应当说明的是：.class文件这种字节码文件，基于Unicode的字节码是不依赖于特定的计算机硬件架构，不是严格的二进制文件，而.cpp文件经过C++编译器编译后，形成的.obj文件，是一种纯二进制的机器指令。

C语言是“一次编写，到处编译”；Java是“一次编译，到处运行”。

### 类的加载

**类的加载指的是将类的.class文件中的二进制数据读入到内存中**，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并提供了访问方法区内的数据结构的接口。

虚拟机规范并没有指明二进制字节流要从一个class文件获取，或者说根本没有指明从哪里获取、怎样获取。这种开放使得Java在很多领域得到充分运用，例如：

- 从ZIP包中读取，这很常见，成为JAR，EAR，WAR格式的基础
- 从网络中获取，最典型的应用就是Applet
- 运行时计算生成，最典型的是动态代理技术，在java.lang.reflect.Proxy中，就是用了ProxyGenerator.generateProxyClass来为特定接口生成形式为“*$Proxy”的代理类的二进制字节流
- 有其他文件生成，最典型的JSP应用，由JSP文件生成对应的Class类

### 类加载机制

![image-20201128095607747](C:/Users/wangchong/AppData/Roaming/Typora/typora-user-images/image-20201128095607747.png)

答：JVM中类的装载是由类加载器（ClassLoader）和它的子类来实现的，Java中的类加载器是一个重要的Java运行时系统组件，它负责在运行时查找和装入类文件中的类。 由于Java的跨平台性，经过编译的Java源程序并不是一个可执行程序，而是一个或多个类文件。

当Java程序需要使用某个类时，JVM会确保这个类已经被

1. 加载
2. 连接（验证、准备和解析）
3. 初始化

#### 类的加载

加载阶段，虚拟机需要完成哪些事情：

> - 通过一个类的全限定名来获取定义此类的二进制字节流
> - 将获取到的二进制字节流转化成一种数据结构并放进方法区
> - 在内存中生成一个代表此类的java.lang.Class对象，作为访问方法区中各种数据的接口

是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件，然后产生与所加载类对应的Class对象。加载完成后，Class对象还不完整，所以此时的类还不可用。

#### 连接阶段

当类被加载后就进入连接阶段，这一阶段包括验证、准备（为静态变量分配内存并设置默认的初始值）和解析（将符号引用替换为直接引用）三个步骤。

验证是连接的第一步，这一阶段的目的主要是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，从而不会危害虚拟机自身安全。也就是说，当加载阶段将字节流加载进方法区之后，JVM需要做的第一件事就是对字节流进行安全校验，以保证格式正确，使自己之后能正确的解析到数据并保证这些数据不会对自身造成危害。

验证阶段主要分成四个子阶段：

> - 文件格式验证
> - 元数据验证
> - 字节码验证
> - 符号引用验证

我不在这里详细的说明每一阶段的校验主要干了什么事情，有兴趣的同学可以自行百度。

挑点重点来说吧，对字节流进行校验是由一个叫做Class文件检验器的东西所完成，其实还是代码实现。

准备：

准备阶段你只要掌握两个知识点：

1.准备阶段的目的：正式为**类变量**分配内存并设置**类变量初始值**的阶段，这些变量所使用的内存将在方法区中分配。

注意我的重点：是类变量（static）不是实例变量，还有，我们又知道了在JVM的方法区中不仅存储着Class字节流（按照运行时方法区的数据结构进行存储，上述的二进制字节流是不严谨的说法，只是为了大家好理解），还有我们的类变量。

2.这里的类变量初始值通常是指数据类型的零值。比如int的零值为0，long为0L，boolean为false… …真正的初始化赋值是在初始化阶段进行的。

额外一点，如果你设置的类变量还具有final字段，如下：

```
public static final int value = 123;
```

那么在准备阶段变量的初始值就会被直接初始化为123，具体原因是由于拥有final字段的变量在它的字段属性表中会出现ConstantValue属性。

**而什么叫做元数据呢？**

所谓的元数据是指用来描述**数据的数据**，更通俗一点就是描述代码间关系，或者代码与其它资源（例如数据库表）之间内在联系的数据，你也可以更简单的认为成框架中的各种@注解，因为这些@注解很简介的描述了大量有关各个类、方法、字段额外的信息或之间的联系。

元数据验证也就是验证这些额外的信息或它们之间的联系是否正确。

我们还得注意字节码验证，在字节码验证中涉及到了一个概念：**字节码流。**

**字节码流 = 操作码 + 操作数。**

操作码就是伪指令，操作数就是普通的Java数据，如int，float等等。

所以对字节码验证的过程就是对字节码流验证的过程，也就是验证操作码是否合法，操作数是否合法。

而符号引用验证涉及到**常量池解析**的知识，在下文中我们顺带着将符号引用验证带过就行，现在先不说。

解析：

这一阶段我个人觉得不太好理解并且非常重要，但我还是会一点点剖析难点，保证你能听懂，所以开始吧～～

先来看一下解析阶段的目的：虚拟机将常量池内的符号引用替换为直接引用。

然后说一下解析阶段最大的特点：发生时间不可预料，有可能和初始化阶段互相交换位置。至于原因，我们等下再说。

先来说看完解析阶段的目的吧，你有可能有三个疑问。哪个常量池？什么符号引用？什么直接引用？Ok，搞清这三个问题，解析这部分你也就学会了。

首先来说**常量池**：在Class的文件结构中我们就花了大量的篇幅去介绍了常量池，我们再来总结一下：**常量池(constant pool)指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。它包括了关于类、方法、接口等中的常量，也包括字符串常量。**

然后这段话中的常量池指的就是存在于.class文件中的常量池，结果在运行期被JVM装载，并且可以扩充的存在于方法区中的**运行时常量池**。

然后来看**符号引用**：在Class文件中我们也讲述了什么是符号引用。总的来说就是常量池中存储的那些描述类、方法、接口的字面量，你可以简单的理解为就是那些所需要信息的全限定名，目的就是为了虚拟机在使用的时候可以定位到所需要的目标。

最后来看**直接引用**：直接指向目标的指针、相对偏移量或能间接定位到目标的句柄。

现在我们对上面那句话进行重新解读：虚拟机将运行时常量池中那些仅代表其他信息的符号引用解析为直接指向所需信息所在地址的指针。

大概就是这样，我觉得你应该已经完全明白了。

解决一个遗留的问题：还记得刚才没有说到的符号引用吗？

这一阶段就是发生在JVM将符号引用转换为直接引用的时候，它的作用就是对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验，以确保解析动作能够正常执行！

在解析阶段主要有以下不同的动作，我只给大家罗列出来，不细讲，有兴趣的同学可以自行百度：

> - 类或接口的解析（注意数组类和非数组类）
> - 字段（简单名称+字段描述符）解析（注意递归搜索）
> - 类方法解析（注意递归搜索）
> - 接口方法解析（注意递归搜索）

在解析阶段还有一个很有意思的东西：**动态连接**！

它也是上面解析阶段发生时间不确定的直接原因：大部分JVM的实现都是延迟加载或者叫做动态连接。它的意思就是JVM装载某个类A时，如果类A中有引用其他类B，虚拟机并不会将这个类B也同时装载进JVM内存，而是等到执行的时候才去装载。

而这个被引用的B类在引用它的类A中的表现形式主要被登记在了符号表中，而解析的过程就是当需要用到被引用类B的时候，将引用类B在引用类A的符号引用名改为内存里的直接引用。这就是解析发生时间不可预料的原因，而且这个阶段是发生在方法区中的。

#### 初始化

最后JVM对类进行初始化，包括：

1)如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；

2)如果类中存在初始化语句，就依次执行这些初始化语句。 类的加载是由类加载器完成的，类加载器包括：根加载器（BootStrap）、扩展加载器（Extension）、系统加载器（System）和用户自定义类加载器（java.lang.ClassLoader的子类）。

从Java 2（JDK 1.2）开始，类加载过程采取了父亲委托机制（PDM）。PDM更好的保证了Java平台的安全性，在该机制中，JVM自带的Bootstrap是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。JVM不会向Java程序提供对Bootstrap的引用。

### 类加载器

从Java虚拟机的角度来说，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），这个类加载器使用C++语言实现（HotSpot虚拟机中），是虚拟机自身的一部分；另一种就是所有其他的类加载器，这些类加载器都有Java语言实现，独立于虚拟机外部，并且全部继承自java.lang.ClassLoader。

从开发者的角度，类加载器可以细分为：
![类加载器的双亲委派模型](https://img-blog.csdn.net/20160506184936657)

- 启动类加载器：Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载），由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作。
- 扩展类加载器：Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载JDK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库（如javax.*开头的类），开发者可以直接使用扩展类加载器。
- 应用程序类加载器：Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath：lib和classes同属classpath，两者的访问优先级为: lib>classes）所指定的类，开发者可以直接使用该类加载器
- 用户自定义类加载器：（应用场景）

  1. 加密：Java代码可以轻易的被反编译，如果你需要把自己的代码进行加密以防止反编译，可以先将编译后的代码用某种加密算法加密，类加密后就不能再用Java的ClassLoader去加载类了，这时就需要自定义ClassLoader在加载类的时候先解密类，然后再加载。
  2. 从非标准的来源加载代码：如果你的字节码是放在数据库、甚至是在云端，就可以自定义类加载器，从指定的来源加载类。
  3. 以上两种情况在实际中的综合运用：比如你的应用需要通过网络来传输 Java 类的字节码，为了安全性，这些字节码经过了加密处理。这个时候你就需要自定义类加载器来从某个网络地址上读取加密后的字节代码，接着进行解密和验证，最后定义出在Java虚拟机中运行的类。

#### JDK1.8

```ruby
[~]$ ls -l jdk
drwxr-xr-x 2 10 143     4096 4月   1 2016 bin
drwxr-xr-x 4 10 143     4096 4月   1 2016 db
drwxr-xr-x 3 10 143     4096 4月   1 2016 include
drwxr-xr-x 5 10 143     4096 4月   1 2016 jre
drwxr-xr-x 5 10 143     4096 4月   1 2016 lib
drwxr-xr-x 4 10 143     4096 4月   1 2016 man
-rw-r--r-- 1 10 143      525 4月   1 2016 release
-rw-r--r-- 1 10 143 21103627 4月   1 2016 src.zip
```

Jdk/lib目录结构

```ruby
-rw-r--r--   1 root  wheel    17M 10  5 18:40 ct.sym
-rw-r--r--   1 root  wheel   159K 10  5 18:40 dt.jar
-rw-r--r--   1 root  wheel    18K 10  5 18:40 ir.idl
-rw-r--r--   1 root  wheel   4.5K  9 11 15:07 packager.jar
-rw-r--r--   1 root  wheel    17M 10  5 18:40 tools.jar
```

#### jre

```csharp
[~]$ ls -l jre
drwxr-xr-x  2 10 143   4096 4月   1 2016 bin
-r--r--r--  1 10 143   3244 4月   1 2016 COPYRIGHT
drwxr-xr-x 15 10 143   4096 4月   1 2016 lib
drwxr-xr-x  3 10 143   4096 4月   1 2016 plugin
```

jre/lib目录结构

```csharp
[~]$ ls -l jre/lib
drwxr-xr-x  13 root  wheel   416B 10  5    ext
drwxr-xr-x  4 10 143     4096 4月   1 2016 amd64
-rwxr-xr-x  1 10 143 66024286 4月   1 2016 rt.jar
.....
```

### 类的唯一性

类加载器虽然只用于实现类的加载动作，但是对于任意一个类，都需要由加载它的类加载器和这个类本身共同确立其在Java虚拟机中的唯一性。通俗的说，JVM中两个类是否“相等”，首先就必须是同一个类加载器加载的，否则，即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要类加载器不同，那么这两个类必定是不相等的。

这里的“相等”，包括代表类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果，也包括使用instanceof关键字做对象所属关系判定等情况。

以下代码说明了不同的类加载器对instanceof关键字运算的结果的影响。

```java
package com.jvm.classloading;

import java.io.IOException;
import java.io.InputStream;

/**
 * 类加载器在类相等判断中的影响
 * 
 * instanceof关键字
 * 
 */

public class ClassLoaderTest {
    public static void main(String[] args) throws Exception {
        // 自定义类加载器
        ClassLoader myLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = getClass().getResourceAsStream(fileName);
                    if (is == null) {
                        return super.loadClass(fileName);
                    }
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);   
                } catch (IOException e) {
                    throw new ClassNotFoundException();
                }
            }
        };

        // 使用ClassLoaderTest的类加载器加载本类
        Object obj1 = ClassLoaderTest.class.getClassLoader().loadClass("com.jvm.classloading.ClassLoaderTest").newInstance();
        System.out.println(obj1.getClass());
        System.out.println(obj1 instanceof com.jvm.classloading.ClassLoaderTest);

        // 使用自定义类加载器加载本类
        Object obj2 = myLoader.loadClass("com.jvm.classloading.ClassLoaderTest").newInstance();
        System.out.println(obj2.getClass());
        System.out.println(obj2 instanceof com.jvm.classloading.ClassLoaderTest);
    }
}
```

输出结果：

> class com.jvm.classloading.ClassLoaderTest
> true
> class com.jvm.classloading.ClassLoaderTest
> false

myLoader是自定义的类加载器，可以用来加载与自己在同一路径下的Class文件。main函数的第一部分使用系统加载主类ClassLoaderTest的类加载器加载ClassLoaderTest，输出显示，obj1的所属类型检查正确，这是虚拟机中有2个ClassLoaderTest类，一个是主类，另一个是main()方法中加载的类，由于这两个类使用同一个类加载器加载并且来源于同一个Class文件，因此这两个类是完全相同的。

第二部分使用自定义的类加载器加载ClassLoaderTest，class com.jvm.classloading.ClassLoderTest显示，obj2确实是类com.jvm.classloading.ClassLoaderTest实例化出来的对象，但是第二句输出false。此时虚拟机中有3个ClassLoaderTest类，由于第3个类的类加载器与前面2个类加载器不同，虽然来源于同一个Class文件，但它是一个独立的类，所属类型检查是返回结果自然是false

### 双亲委派模型

#### 双亲委派模型过程

当前类加载器从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。

某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载，调用当前加载器的findClass()方法进行加载。

#### 好处

（1）主要是为了安全性，避免用户自己编写的类动态替换 Java的一些核心类，比如 String。

（2）同时也避免了类的重复加载，因为 JVM中区分不同类，不仅仅是根据类名，相同的 class文件被不同的 ClassLoader加载就是不同的两个类。

使用双亲委派模型的好处在于Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存在在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的Bootstrap ClassLoader进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。

相反，如果没有双亲委派模型而是由各个类加载器自行加载的话，如果用户编写了一个java.lang.Object的同名类并放在ClassPath中，那系统中将会出现多个不同的Object类，程序将混乱。因此，如果开发者尝试编写一个与rt.jar类库中重名的Java类，可以正常编译，但是永远无法被加载运行。

#### 双亲委派模型的系统实现

在java.lang.ClassLoader的loadClass()方法中，先检查是否已经被加载过，若没有加载则调用父类加载器的loadClass()方法，若父加载器为空则默认使用启动类加载器作为父加载器。如果父加载失败，则抛出ClassNotFoundException异常后，再调用自己的findClass()方法进行加载。

```java
//源码
protected synchronized Class<?> loadClass(String name,boolean resolve)throws ClassNotFoundException{
    //check the class has been loaded or not
    Class c = findLoadedClass(name);
    if(c == null){
        try{
            if(parent != null){
                c = parent.loadClass(name,false);
            }else{
                c = findBootstrapClassOrNull(name);
            }
        }catch(ClassNotFoundException e){
            //if throws the exception ,the father can not complete the load
        }
        if(c == null){
            c = findClass(name);
        }
    }
    if(resolve){
        resolveClass(c);
    }
    return c;
}
```

### 全盘委托机制

   当一个类运行时,可能有其他的类,这时由应用类加载器委托给扩展类加载器是否加载这些类,扩展类加载器再次向上委托引导类加载器是否加载这些类,引导类加载器判断后将有的类进行加载向内存中返回class对象后,再由扩展类加载器中有的类进行加载返回class对象,剩下全部有应用类加载器进行加载.

### 对象分配规则

- 对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。
- 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
- 长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，直到达到阀值对象进入老年区。
- 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
- 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。

### 对象创建过程

1.JVM遇到一条新建对象的指令时首先去检查这个指令的参数是否能在常量池中定义到一个类的符号引用。然后加载这个类（类加载过程在前面讲了）

2.为对象分配内存。一种办法“指针碰撞”、一种办法“空闲列表”，最终常用的办法“本地线程缓冲分配(TLAB)”

3.将除对象头外的对象内存空间初始化为0

4.对对象头进行必要设置

### 对象结构

Java对象由三个部分组成：对象头、实例数据、对齐填充。

#### 对象头

第一部分存储对象自身的运行时数据（1字宽）（一般占32/64 bit）：

HashCode（25bit），对象分代年龄（4bit），是否使用偏向锁（1bit），锁标志位（2bit）。

第二部分是指针类型（1字宽）：

指向对象的类元数据类型（即对象代表哪个类）。如果是数组对象，则对象头中还有一部分用来记录数组长度（1字宽）。

#### 实例数据

用来存储对象真正的有效信息（包括父类继承下来的和自己定义的）

#### 对齐填充

JVM要求对象起始地址必须是8字节的整数倍（8字节对齐）

### Java对象的定位方式

句柄池、直接指针。

### 判断对象是否存活

判断对象是否存活一般有两种方式：

- 引用计数：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。
- 可达性分析（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的，不可达对象。

### 引用的分类

| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |

从`JDK 1.2`版本开始，对象的引用被划分为`4`种级别，从而使程序能更加灵活地控制**对象的生命周期**。这`4`种级别**由高到低**依次为：**强引用**、**软引用**、**弱引用**和**虚引用**。

![image-20200227125044068](/Users/wangchong/Library/Application Support/typora-user-images/image-20200227125044068.png)

#### 强引用：

> GC时不会被回收

**强引用**是使用最普遍的引用。如果一个对象具有强引用，那**垃圾回收器**绝不会回收它。如下：

```java
Object strongReference = new Object();
```

当**内存空间不足**时，`Java`虚拟机宁愿抛出`OutOfMemoryError`错误，使程序**异常终止**，也不会靠随意**回收**具有**强引用**的**对象**来解决内存不足的问题。 如果强引用对象**不使用时**，需要弱化从而使`GC`能够回收，如下：

```java
strongReference = null;
```

显式地设置`strongReference`对象为`null`，或让其**超出**对象的**生命周期**范围，则`gc`认为该对象**不存在引用**，这时就可以回收这个对象。具体什么时候收集这要取决于`GC`算法。

```java
public void test() {
    Object strongReference = new Object();
    // 省略其他操作
}
```

在一个**方法的内部**有一个**强引用**，这个引用保存在`Java`**栈**中，而真正的引用内容(`Object`)保存在`Java`**堆**中。 当这个**方法运行完成**后，就会退出**方法栈**，则引用对象的**引用数**为`0`，这个对象会被回收。

但是如果这个`strongReference`是**全局变量**时，就需要在不用这个对象时赋值为`null`，因为**强引用**不会被垃圾回收。

**ArrayList的Clear方法：**

```java
public void clear() {
  	modCount++;
  	
  	// clear to let GC do its work
  	for (int i = 0; i < size; i++)
				elementData[i] = null;
  
  	size = 0;
}
```

在`ArrayList`类中定义了一个`elementData`数组，在调用`clear`方法清空数组时，每个数组元素被赋值为`null`。 不同于`elementData=null`，强引用仍然存在，避免在后续调用`add()`等方法添加元素时进行内存的**重新分配**。 使用如`clear()`方法**内存数组**中存放的**引用类型**进行**内存释放**特别适用，这样就可以及时释放内存。

#### 软引用：

> 描述有用但不是必须的对象，在发生内存溢出异常之前被回收

```java
// 强引用
String strongReference = new String("abc");
// 软引用
String str = new String("abc");
SoftReference<String> softReference = new SoftReference<String>(str);
```

**软引用**可以和一个**引用队列**(`ReferenceQueue`)联合使用。如果**软引用**所引用对象被**垃圾回收**，`JAVA`虚拟机就会把这个**软引用**加入到与之关联的**引用队列**中。

```java
ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
String str = new String("abc");
SoftReference<String> softReference = new SoftReference<>(str, referenceQueue);

str = null;
// Notify GC
System.gc();
// 虽然触发GC了，但是内存充足没有回收softReference
System.out.println(softReference.get()); // abc

Reference<? extends String> reference = referenceQueue.poll();
System.out.println(reference); //null
```

> 注意：软引用对象是在jvm内存不够的时候才会被回收，我们调用System.gc()方法只是起通知作用，JVM什么时候扫描回收对象是JVM自己的状态决定的。就算扫描到软引用对象也不一定会回收它，只有内存不够的时候才会回收。

当内存不足时，`JVM`首先将**软引用**中的**对象**引用置为`null`，然后通知**垃圾回收器**进行回收：

```java
if(JVM内存不足) {
    // 将软引用中的对象引用置为null
    str = null;
    // 通知垃圾回收器进行回收
    System.gc();
}
```

**应用场景：**

浏览器的后退按钮。按后退时，这个后退时显示的网页内容是重新进行请求还是从缓存中取出呢？这就要看具体的实现策略了。

1. 如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建；
2. 如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出。

这时候就可以使用软引用，很好的解决了实际的问题：

```java
    // 获取浏览器对象进行浏览
    Browser browser = new Browser();
    // 从后台程序加载浏览页面
    BrowserPage page = browser.getPage();
    // 将浏览完毕的页面置为软引用
    SoftReference softReference = new SoftReference(page);

    // 回退或者再次浏览此页面时
    if(softReference.get() != null) {
        // 内存充足，还没有被回收器回收，直接获取缓存
        page = softReference.get();
    } else {
        // 内存不足，软引用的对象已经回收
        page = browser.getPage();
        // 重新构建软引用
        softReference = new SoftReference(page);
    }
```

#### 弱引用：

> 描述有用但不是必须的对象，在下一次GC时被回收

**弱引用**与**软引用**的区别在于：只具有**弱引用**的对象拥有**更短暂**的**生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有**弱引用**的对象，不管当前**内存空间足够与否**，都会**回收**它的内存。不过，由于垃圾回收器是一个**优先级很低的线程**，因此**不一定**会**很快**发现那些只具有**弱引用**的对象。

```java
String str = new String("abc");
WeakReference<String> weakReference = new WeakReference<>(str);
str = null;
```

`JVM`首先将**软引用**中的**对象**引用置为`null`，然后通知**垃圾回收器**进行回收：

```java
str = null;
System.gc();
```

> 注意：如果一个对象是偶尔(很少)的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用Weak Reference来记住此对象。

下面的代码会让一个**弱引用**再次变为一个**强引用**：

```java
String str = new String("abc");
WeakReference<String> weakReference = new WeakReference<>(str);
// 弱引用转强引用
String strongReference = weakReference.get();
```

同样，**弱引用**可以和一个**引用队列**(`ReferenceQueue`)联合使用，如果**弱引用**所引用的**对象**被**垃圾回收**，`Java`虚拟机就会把这个**弱引用**加入到与之关联的**引用队列**中。

**简单测试：**

GCTarget.java

```java
public class GCTarget {
    // 对象的ID
    public String id;

    // 占用内存空间
    byte[] buffer = new byte[1024];

    public GCTarget(String id) {
        this.id = id;
    }

    protected void finalize() throws Throwable {
        // 执行垃圾回收时打印显示对象ID
        System.out.println("Finalizing GCTarget, id is : " + id);
    }
}
```

GCTargetWeakReference.java

```java
public class GCTargetWeakReference extends WeakReference<GCTarget> {
    // 弱引用的ID
    public String id;

    public GCTargetWeakReference(GCTarget gcTarget,
              ReferenceQueue<? super GCTarget> queue) {
        super(gcTarget, queue);
        this.id = gcTarget.id;
    }

    protected void finalize() {
        System.out.println("Finalizing GCTargetWeakReference " + id);
    }
}
```

WeakReferenceTest.java

```java
public class WeakReferenceTest {
    // 弱引用队列
    private final static ReferenceQueue<GCTarget> REFERENCE_QUEUE = new ReferenceQueue<>();

    public static void main(String[] args) {
        LinkedList<GCTargetWeakReference> gcTargetList = new LinkedList<>();

        // 创建弱引用的对象，依次加入链表中
        for (int i = 0; i < 5; i++) {
            GCTarget gcTarget = new GCTarget(String.valueOf(i));
            GCTargetWeakReference weakReference = new GCTargetWeakReference(gcTarget,
                REFERENCE_QUEUE);
            gcTargetList.add(weakReference);

            System.out.println("Just created GCTargetWeakReference obj: " +
                gcTargetList.getLast());
        }

        // 通知GC进行垃圾回收
        System.gc();

        try {
            // 休息几分钟，等待上面的垃圾回收线程运行完成
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 检查关联的引用队列是否为空
        Reference<? extends GCTarget> reference;
        while((reference = REFERENCE_QUEUE.poll()) != null) {
            if(reference instanceof GCTargetWeakReference) {
                System.out.println("In queue, id is: " +
                    ((GCTargetWeakReference) (reference)).id);
            }
        }
    }
}
```

运行`WeakReferenceTest.java`，运行结果如下：

![image-20200227133142211](/Users/wangchong/Library/Application Support/typora-user-images/image-20200227133142211.png)

可见`WeakReference`对象的生命周期基本由**垃圾回收器**决定，一旦垃圾回收线程发现了**弱引用对象**，在下一次`GC`过程中就会对其进行回收。

#### 虚引用

> （幽灵引用/幻影引用）:无法通过虚引用获得对象，用PhantomReference实现虚引用，虚引用用来在GC时返回一个通知。

**虚引用**顾名思义，就是**形同虚设**。与其他几种引用都不同，**虚引用**并**不会**决定对象的**生命周期**。如果一个对象**仅持有虚引用**，那么它就和**没有任何引用**一样，在任何时候都可能被垃圾回收器回收。

**应用场景：**

**虚引用**主要用来**跟踪对象**被垃圾回收器**回收**的活动。 **虚引用**与**软引用**和**弱引用**的一个区别在于：

> 虚引用必须和引用队列(ReferenceQueue)联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

```java
String str = new String("abc");
ReferenceQueue queue = new ReferenceQueue();
// 创建虚引用，要求必须与一个引用队列关联
PhantomReference pr = new PhantomReference(str, queue);
复制代码
```

程序可以通过判断引用**队列**中是否已经加入了**虚引用**，来了解被引用的对象是否将要进行**垃圾回收**。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的**内存被回收之前**采取必要的行动。

### GC是什么？为什么要有GC？

答：GC是垃圾收集的意思，内存处理是编程人员容易出现问题的地方，忘记或者错误的内存回收会导致程序或系统的不稳定甚至崩溃，Java提供的GC功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的，Java语言没有提供释放已分配内存的显示操作方法。Java程序员不用担心内存管理，因为垃圾收集器会自动进行管理。要请求垃圾收集，可以调用下面的方法之一：System.gc() 或Runtime.getRuntime().gc() ，但JVM可以屏蔽掉显示的垃圾回收调用。 垃圾回收可以有效的防止内存泄露，有效的使用内存。垃圾回收器通常是作为一个单独的低优先级的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收，程序员不能实时的调用垃圾回收器对某个对象或所有对象进行垃圾回收。

在Java诞生初期，垃圾回收是Java最大的亮点之一，因为服务器端的编程需要有效的防止内存泄露问题，然而时过境迁，如今Java的垃圾回收机制已经成为被诟病的东西。移动智能终端用户通常觉得iOS的系统比Android系统有更好的用户体验，其中一个深层次的原因就在于android系统中垃圾回收的不可预知性。

### 判断一个对象应该被回收

1. 该对象没有与GC Roots相连

   ​	可作为 GC roots 的对象

   ​	1）java 虚拟机栈（栈帧中的本地变量表）中引用的对象
   ​	2）方法区中类的静态属性引用的对象
   ​	3）方法区中常量引用的对象
   ​	4）本地方法栈中 JNI 引用的对象

2. 该对象没有重写finalize()方法或finalize()已经被执行过则直接回收（第一次标记）、否则将对象加入到F-Queue队列中（优先级很低的队列）在这里finalize()方法被执行，之后进行第二次标记，如果对象仍然应该被GC则GC，否则移除队列。 （在finalize方法中，对象很可能和其他 GC Roots中的某一个对象建立了关联，finalize方法只会被调用一次，且不推荐使用finalize方法）

### 回收方法区

方法区回收价值很低，主要回收废弃的常量和无用的类。

#### 如何判断无用的类

1. 该类所有实例都被回收（Java堆中没有该类的对象）

2. 加载该类的ClassLoader已经被回收

3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方利用反射访问该类

### 垃圾收集算法

GC最基础的算法有三种： 标记 -清除算法、复制算法、标记-压缩算法、分代收集算法

我们常用的垃圾回收器一般都采用分代收集算法。

#### 复制算法

新生代

“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

#### 标记 -清除算法

老年代、永久代

“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。

#### 标记-整理算法

老年代、永久代

标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

#### 分代收集算法

Java虚拟机将堆分为新生代和老年代，永久代，这样就可以根据各个年代的特点采用最适当的收集算法。新生代和老年代是垃圾回收的主要区域。

永久代是HotSpot虚拟机特有的概念，它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大。

内存分代示意图如下：

![image-20200227231029825](/Users/wangchong/Library/Application Support/typora-user-images/image-20200227231029825.png)

##### 新生代（JAVA堆）

> 某一个方法的局域变量、循环内的临时变量等等

朝生夕灭的对象，通俗点讲就是活不了多久就得死的对象。每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用**复制算法**，只需要付出少量存活对象的复制成本就可以完成收集。

新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低，在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高。

HotSpot将新生代划分为三块，一块较大的Eden空间和两块较小的Survivor空间，默认比例为8：1：1。划分的目的是因为HotSpot采用复制算法来回收新生代，设置这个比例是为了充分利用内存空间，减少浪费。新生成的对象在Eden区分配（大对象除外，大对象直接进入老年代），当Eden区没有足够的空间进行分配时，虚拟机将发起一次MinorGC。

GC开始时，对象只会存在于Eden区和From Survivor区，To Survivor区是空的（作为保留区域）。GC进行时，Eden区中所有存活的对象都会被复制到To Survivor区，而在FromSurvivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阀值（默认为15，新生代中的对象每熬过一轮垃圾回收，年龄值就加1，GC分代年龄存储在对的header中）的对象会被移到老年代中，没有达到阀值的对象会被复制到To Survivor区。接着清空Eden区和From Survivor区，新生代中存活的对象都在To Survivor区。接着， FromSurvivor区和To Survivor区会交换它们的角色，也就是新的To Survivor区就是上次GC清空的From Survivor区，新的From Survivor区就是上次GC的To Survivor区，总之，不管怎样都会保证To Survivor区在一轮GC后是空的。GC时当To Survivor区没有足够的空间存放上一次新生代收集下来的存活对象时，需要依赖老年代进行分配担保，将这些对象存放在老年代中。

- 伊甸园（Eden）：这是对象最初诞生的区域，并且对大多数对象来说，这里是它们唯一存在过的区域。
- 幸存者乐园（Survivor：From，To）：从伊甸园幸存下来的对象会被挪到这里。
- 终身颐养园（Tenured）：这是足够老的幸存对象的归宿。年轻代收集（Minor-GC）过程是不会触及这个地方的。当年轻代收集不能把对象放进终身颐养园时，就会触发一次完全收集（Major-GC），这里可能还会牵扯到压缩，以便为大对象腾出足够的空间。 与垃圾回收相关的JVM参数

##### 老年代（JAVA堆）

 在新生代中经历了多次（具体看虚拟机配置的阀值）GC后仍然存活下来的对象会进入老年代中。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。对象存活率高、没有额外空间对它进行分配担保，就必须使用“**标记-清除**”或“**标记-整理**”算法来进行回收。

例子：缓存对象、数据库连接对象、单例对象（单例模式）等等。

##### 永久代（方法区）

此类对象一般一旦出生就几乎不死了，它们几乎会一直永生不灭，记得，只是几乎不灭而已。

永久代是HotSpot虚拟机特有的概念，它采用永久代的方式来实现方法区，其他的虚拟机实现没有这一概念，而且HotSpot也有取消永久代的趋势，在JDK 1.7中HotSpot已经开始了“去永久化”，把原本放在永久代的字符串常量池移出。永久代主要存放常量、类信息、静态变量等数据，与垃圾回收关系不大，

垃圾回收不会发生在永久代，如果永久代满了或者是超过了临界值，会触发完全垃圾回收(Full GC)。如果你仔细查看垃圾收集器的输出信息，就会发现永久代也是被回收的。这就是为什么正确的永久代大小对避免Full GC是非常重要的原因。请参考下Java8：从永久代到元数据区 (注：Java8中已经移除了永久代，新加了一个叫做元数据区的native内存区)

在我们常用的hotspot虚拟机（JDK默认的JVM）中，方法区也被亲切的称为永久代，其实在很久很久以前，是不存在永久代的。当时永久代与年老代都存放在一起，里面包含了JAVA类的实例信息以及类信息。但是后来发现，对于类信息的卸载几乎很少发生，因此便将二者分离开来。幸运的是，这样做确实提高了不少性能。于是永久代便被拆分出来了。
这一部分区域的GC与年老代采用相似的方法，由于都没有“备用仓库”，二者都是只能使用**标记/清除和标记/整理算法**。

### Minor GC与Full GC

新生代内存不够用时候发生MGC也叫YGC，JVM内存不够的时候发生FGC

新生代GC（Minor GC）：Minor GC指发生在新生代的GC，因为新生代的Java对象大多都是朝生夕死，所以Minor GC非常频繁，一般回收速度也比较快。当Eden空间不足以为对象分配内存时，会触发Minor GC。

 老年代GC（Full GC/Major GC）：Full GC指发生在老年代的GC，出现了FullGC一般会伴随着至少一次的Minor GC（老年代的对象大部分是Minor GC过程中从新生代进入老年代），比如：分配担保失败。FullGC的速度一般会比Minor GC慢10倍以上。当老年代内存不足或者显式调用System.gc()方法时，会触发Full GC。

Meta space提高Full GC的性能，在Full GC期间，Metadata到Metadata pointers之间不需要扫描了

### Full GC案例

某个业务系统在一段时间突然变慢，我们怀疑是因为出现**内存泄露**问题导致的，于是踏上排查之路。

#### 首先确定问题

频繁Full GC现象

#### 虚拟机进程状况工具：jps

找出正在运行的虚拟机进程，最主要是找出这个进程在**本地虚拟机的唯一ID**（LVMID，Local Virtual Machine Identifier），因为在后面的排查过程中都是需要这个LVMID来确定要监控的是哪一个虚拟机进程。

同时，对于本地虚拟机进程来说，LVMID与操作系统的进程ID（PID，Process Identifier）是一致的，使用Windows的任务管理器或Unix的ps命令也可以查询到虚拟机进程的LVMID。

jps命令格式为：
jps [ options ] [ hostid ]
使用命令如下：
使用jps：jps -l
使用ps：ps aux | grep tomat

找到你需要监控的ID（假设为20954）

#### 虚拟机统计信息监视工具：jstat

监视虚拟机各种运行状态信息。

jstat命令格式为：
jstat [ option vmid [interval[s|ms] [count]] ]
使用命令如下：
jstat -gcutil 20954 1000
意思是每1000毫秒查询一次，一直查。gcutil的意思是已使用空间站总空间的百分比。

查询结果表明：这台服务器的新生代Eden区（E，表示Eden）使用了28.30%（最后）的空间，两个Survivor区（S0、S1，表示Survivor0、Survivor1）分别是0和8.93%，老年代（O，表示Old）使用了87.33%。程序运行以来共发生Minor GC（YGC，表示Young GC）101次，总耗时1.961秒，发生Full GC（FGC，表示Full GC）7次，Full GC总耗时3.022秒，总的耗时（GCT，表示GC Time）为4.983秒。

#### 找出导致频繁Full GC的原因

分析方法通常有两种：
1）把堆dump下来再用MAT等工具进行分析，但dump堆要花较长的时间，并且文件巨大，再从服务器上拖回本地导入工具，这个过程有些折腾，不到万不得已最好别这么干。
2）更轻量级的在线分析，使用“Java内存影像工具：jmap”生成堆转储快照（一般称为headdump或dump文件）。

jmap命令格式：
jmap [ option ] vmid
使用命令如下：
jmap -histo:live 20954

查看存活的对象情况，如下图所示：

存活对象
按照一位IT友的说法，数据不正常，十有八九就是泄露的。在我这个图上对象还是挺正常的。

可以看出HashTable中的元素有5000多万，占用内存大约1.5G的样子。这肯定不正常。

定位到代码
定位带代码，有很多种方法，比如前面提到的通过MAT查看Histogram即可找出是哪块代码。——我以前是使用这个方法。 也可以使用BTrace，我没有使用过。

举例：

一台生产环境机器每次运行几天之后就会莫名其妙的宕机，分析日志之后发现在tomcat刚启动的时候内存占用比较少，但是运行个几天之后内存占用越来越大，通过jmap命令可以查询到一些大对象引用没有被及时GC，这里就要求解决内存泄露的问题。

Java的内存泄露多半是因为对象存在无效的引用，对象得不到释放，如果发现Java应用程序占用的内存出现了泄露的迹象，那么我们一般采用下面的步骤分析：
1. 用工具生成java应用程序的heap dump（如jmap）
2. 使用Java heap分析工具（如MAT），找出内存占用超出预期的嫌疑对象
3. 根据情况，分析嫌疑对象和其他对象的引用关系。
4. 分析程序的源代码，找出嫌疑对象数量过多的原因。




### JVM参数

| 参数                                                    | 内容                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| **-Xms**                                                | 初始堆大小。如：-Xms256m                                     |
| -Xmx                                                    | 最大堆大小。如：-Xmx512m                                     |
| -Xss                                                    | JDK1.5+ 每个线程堆栈大小为 1M，一般来说如果栈不是很深的话， 1M 是绝对够用了的。 |
| -Xmn                                                    | 新生代大小。通常为 Xmx 的 1/3 或 1/4。新生代 = Eden + 2 个 Survivor 空间。实际可用空间为 = Eden + 1 个 Survivor，即 90% |
| -XX:NewSize / XX:MaxNewSize                             | 设置新生代大小/新生代最大大小                                |
| -XX:InitialTenuringThreshold / -XX:MaxTenuringThreshold | 设置老年代阀值的初始值和最大值                               |
| -XX:NewRatio                                            | 新生代与老年代的比例，如 –XX:NewRatio=2，则新生代占整个堆空间的1/3，老年代占2/3 |
| -XX:TargetSurvivorRatio                                 | 设置幸存区的目标使用率                                       |
| **-XX:SurvivorRatio**                                   | 新生代中 Eden(8) 与 Survivor(1+1) 的比值。默认值为 8。即 Eden 占新生代空间的 8/10，另外两个 Survivor 各占 1/10 |
| -XX:PermSize                                            | 永久代(方法区)的初始大小                                     |
| **-XX:MaxPermSize**                                     | 永久代(方法区)的最大值                                       |
| -XX:+PrintGCDetails                                     | 打印 GC 信息                                                 |
| -XX:+HeapDumpOnOutOfMemoryError                         | 让虚拟机在发生内存溢出时 Dump 出当前的内存堆转储快照，以便分析用 |
| -XX:-DisableExplicitGC                                  | 让System.gc()不产生任何作用                                  |
| -XX:PrintTenuringDistribution                           | 设置每次新生代GC后输出Survivor中对象年龄的分布               |

### 垃圾回收器

- Serial收集器，串行收集器是最古老，最稳定以及效率高的收集器，可能会产生较长的停顿，只使用一个线程去回收。
- ParNew收集器，ParNew收集器其实就是Serial收集器的多线程版本。
- Parallel收集器，Parallel Scavenge收集器类似ParNew收集器，Parallel收集器更关注系统的吞吐量。
- Parallel Old 收集器，Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法
- CMS收集器，CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。
- G1收集器，G1 (Garbage-First)是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征

### GC日志分析

摘录GC日志一部分（前部分为年轻代gc回收；后部分为full gc回收）：

```
2016-07-05T10:43:18.093+0800: 25.395: [GC [PSYoungGen: 274931K->10738K(274944K)] 371093K->147186K(450048K), 0.0668480 secs] [Times: user=0.17 sys=0.08, real=0.07 secs] 
2016-07-05T10:43:18.160+0800: 25.462: [Full GC [PSYoungGen: 10738K->0K(274944K)] [ParOldGen: 136447K->140379K(302592K)] 147186K->140379K(577536K) [PSPermGen: 85411K->85376K(171008K)], 0.6763541 secs] [Times: user=1.75 sys=0.02, real=0.68 secs]
复制代码
```

通过上面日志分析得出，PSYoungGen、ParOldGen、PSPermGen属于Parallel收集器。其中PSYoungGen表示gc回收前后年轻代的内存变化；ParOldGen表示gc回收前后老年代的内存变化；PSPermGen表示gc回收前后永久区的内存变化。young gc 主要是针对年轻代进行内存回收比较频繁，耗时短；full gc 会对整个堆内存进行回城，耗时长，因此一般尽量减少full gc的次数

### 调优命令

Sun JDK监控和故障处理命令有jps jstat jmap jhat jstack jinfo

- jps，JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。
- jstat，JVM statistics Monitoring是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
- jmap，JVM Memory Map命令用于生成heap dump文件
- jhat，JVM Heap Analysis Tool命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看
- jstack，用于生成java虚拟机当前时刻的线程快照。
- jinfo，JVM Configuration info 这个命令作用是实时查看和调整虚拟机运行参数。

### 其他命令

#### javap

> javap是 Java class文件分解器，可以反编译，也可以查看java编译器生成的字节码，从而对代码内部的执行逻辑进行分析。

语法：把java文件编译为class文件：javac  Test.java  (Test.java为java文件名) 生成对应的 .class 文件 Test.class

执行javap操作：javap 命令行 class文件名称（不加 .class后缀）

例如： javap -c Test

命令行

```ruby
　　-help 输出 javap 的帮助信息。
　　-l 输出行及局部变量表。
　　-b 确保与 JDK 1.1 javap 的向后兼容性。
　　-public 只显示 public 类及成员。
　　-protected 只显示 protected 和 public 类及成员。
　　-package 只显示包、protected 和 public 类及成员。这是缺省设置。
　　-private 显示所有类和成员。
　　-J[flag] 直接将 flag 传给运行时系统。
　　-s 输出内部类型签名。
　　-c 输出类中各方法的未解析的代码，即构成 Java 字节码的指令。
　　-verbose 输出堆栈大小、各方法的 locals 及 args 数,以及class文件的编译版本
　　-classpath[路径] 指定 javap 用来查找类的路径。如果设置了该选项，则它将覆盖缺省值或 CLASSPATH 环境变量。目录用冒号分隔。
-bootclasspath[路径] 指定加载自举类所用的路径。缺省情况下，自举类是实现核心 Java 平台的类，位于 jrelibt.jar 和 jrelibi18n.jar 中。
-extdirs[dirs] 覆盖搜索安装方式扩展的位置。扩展的缺省位置是 jrelibext。
```



### 调优工具

常用调优工具分为两类,jdk自带监控工具：jconsole和jvisualvm，第三方有：MAT(Memory Analyzer Tool)、GChisto。

- jconsole，Java Monitoring and Management Console是从java5开始，在JDK中自带的java监控和管理控制台，用于对JVM中内存，线程和类等的监控
- jvisualvm，jdk自带全能工具，可以分析内存快照、线程快照；监控内存变化、GC变化等。
- MAT，Memory Analyzer Tool，一个基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗
- GChisto，一款专业分析gc日志的工具

### 你知道哪些JVM性能调优

- 设定堆内存大小

-Xmx：堆内存最大限制。

- 设定新生代大小。 新生代不宜太小，否则会有大量对象涌入老年代

-XX:NewSize：新生代大小

-XX:NewRatio  新生代和老生代占比

-XX:SurvivorRatio：伊甸园空间和幸存者空间的占比

- 设定垃圾回收器 年轻代用  -XX:+UseParNewGC  年老代用-XX:+UseConcMarkSweepGC

## OOM 排查（内存溢出）

### 查看进程pid

Linux下

```ruby
ps -ef|grep java
```

Java下

```ruby
jps -l
```

登录linux服务器，获取tomcat的pid，命令：

![img](https://img-blog.csdn.net/20160907153739493)

### jmap命令

#### jmap -heap pid

输出当前进程 JVM 堆新生代、老年代、永久代等请情况，GC 使用的算法等信息
jmap -histo:live {pid} | head -n 10 输出当前进程内存中所有对象包含的大小
jmap -dump:format=b,file=/usr/local/logs/gc/dump.hprof {pid} 以二进制输出档当前内存的堆情况，然后可以导入 MAT 等工具进行

```ruby
jmap -heap pid
```

输出当前进程JVM堆新生代、老年代、持久代等情况，GC使用的算法等信息。

![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145609060-547180624.png)

#### Jmap -histo

```ruby
jmap -histo:live {pid} | head -n 10
```

输出当前进程内存中所有对象包含的大小

输出当前进程内存中所有对象实例数 (instances) 和大小 (bytes), 如果某个业务对象实例数和大小存在异常情况，可能存在内存泄露或者业务设计方面存在不合理之处。
![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145609413-1576871505.png)

#### jmap -dump

命令如下：
mkdir logs

```ruby
jmap -dump:format=b,file=/tmp/logs/dump.hprof {pid}
```

-dump:formate=b,file= 以二进制输出当前内存的堆情况至相应的文件，然后可以结合 MAT 等内存分析工具深入分析当前内存情况。
也可以通过JVM参数配置OOM时自动dump当前内存镜像文件。 -XX:+HeapDumpOnOutOfMemoryError 和-XX:HeapDumpPath所代表的含义就是当程序出现OutofMemory时，将会在相应的目录下生成一份dump文件，而如果不指定选项-XX:HeapDumpPath则在当前目录下生成dump文件。
确保应用发生 OOM 时 JVM 能够保存并 dump 出当前的内存镜像。
当然，如果你决定手动 dump 内存时，dump 操作占据一定 CPU 时间片、内存资源、磁盘资源等，因此会带来一定的负面影响。
此外，dump 的文件可能比较大 , 一般我们可以考虑使用 zip 命令对文件进行压缩处理，这样在下载文件时能减少带宽的开销。
下载 dump 文件完成之后，由于 dump 文件较大可将 dump 文件备份至制定位置或者直接删除，以释放磁盘在这块的空间占用。

### dump 日志分析

MAT(Memory Analyzer Tool)，一个基于 Eclipse 的内存分析工具，是一个快速、功能丰富的 JAVA heap 分析工具，它可以帮助我们查找内存泄漏和减少内存消耗。
使用内存分析工具从众多的对象中进行分析，快速的计算出在内存中对象的占用大小，看看是谁阻止了垃圾收集器的回收工作，并可以通过报表直观的查看到可能造成这种结果的对象。

直接看到下面Action窗口，有4种Action来分析heap profile，介绍其中最常用的2种:

\- **Histogram**：这个使用的最多，跟上面的jmap -histo 命令类似，只是在MAT里面可以用GUI来展示应用系统各个类产生的实例。

Shllow Heap排序后发现 Cms_Organization 这个类占用的内存比较多（没有得到及时GC），查看引用：

### jstack命令

> printf '%x\n' tid --> 10 进制至 16 进制线程 ID(navtive 线程) %d 10 进制
> jstack pid | grep tid -C 30 --color ps -mp 8278 -o THREAD,tid,time | head -n 40

某 Java 进程 CPU 占用率高，我们想要定位到其中 CPU 占用率最高的线程。
(1) 先利用top命令找到CPU占用高的进程pid
也可以通过ps -ef | grep 应用名 来快速定位自己应用的pid
![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145609683-64030502.png)

显示pid：29080
(2) 利用 top 命令可以查出占 CPU 最高的线程 pid (先找到该pid 29080下所有的线程数据)
![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145610005-889345981.png)
可以看到占用cpu资源最高的为29173

(3) 占用率最高的线程 ID 为29173，将其转换为 16 进制形式 (因为 java native 线程以 16 进制形式输出)

> printf '%x\n' 29173
> ![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145610294-1030547865.png)

(4) 利用 jstack 打印出 java 线程调用栈信息

jstack 29080 | grep '0x71f5' -A 50 --color
![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145610646-831537750.png)

可以看到这个线程是在做kafka相关的操作。因为这是测试，所以并不是因为CPU真的占用过高的情况。

### jinfo命令

jinfo可以用来查看正在运行的java运用程序的扩展参数。
查看pid对应的JVM参数，可以到 [PerfMa : http://xxfox.perfma.com/jvm/check](http://xxfox.perfma.com/jvm/check) 校验参数的正确性
jinfo -flags pid

![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145610953-1796316527.png)

拿到Command line后面的配置参数到perfma中验证查询：
![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145611251-430253782.png)

这里面好多功能可以去使用。

### jstat -gc命令

jstat：Java Virtual Machine statistics monitoring tool JDK自带的一个轻量级小工具。

#### jstat显示GC执行的情况

jstat -gc 12538 5000
即会每5秒一次显示进程号为12538的java进成的GC情况

![img](https://img2018.cnblogs.com/blog/799093/201812/799093-20181209145611536-412867147.png)

说明：
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
PC、PU：永久代容量和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时

显示内容说明如下（部分结果是通过其他其他参数显示的，暂不说明）：
S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
EU：年轻代中Eden（伊甸园）目前已使用空间 (字节)
OC：Old代的容量 (字节)
OU：Old代目前已使用空间 (字节)
PC：Perm(持久代)的容量 (字节)
PU：Perm(持久代)目前已使用空间 (字节)
YGC：从应用程序启动到采样时年轻代中gc次数
YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
FGC：从应用程序启动到采样时old代(全gc)gc次数
FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)

### 总结

一般分析CPU或者内存异常情况可以通过以下几步：

1. 查看日志
2. 查看CPU情况
3. 查看TCP情况
4. 查看java线程，jstack
5. 查看java堆，jmap
6. 通过MAT分析堆文件，寻找无法被回收的对象

# 内存泄漏和内存溢出

1、内存溢出：你申请了10个字节的空间，但是你在这个空间写入11或以上字节的数据，出现溢出。
2、内存泄漏：你用new申请了一块内存，后来很长时间都不再使用了（按理应该释放），但是因为一直被某个或某些实例所持有导致 GC 不能回收，也就是该被释放的对象没有释放。

轻微的内存泄漏一般不宜察觉。内存泄漏较严重，会造成应用速度变慢。更严重的就会造成内存溢出。

## 关系

内存泄露会最终会导致内存溢出。
相同点：都会导致应用程序运行出现问题，性能下降或挂起。
不同点：

1. 内存泄露是导致内存溢出的原因之一，内存泄露积累起来将导致内存溢出。

2. 内存泄露可以通过完善代码来避免，内存溢出可以通过调整配置来减少发生频率，但无法彻底避免。

## 内存溢出

java.lang.OutOfMemoryError，是指程序在申请内存时，没有足够的内存空间供其使用，出现OutOfMemoryError。

### 产生原因

产生该错误的原因主要包括：

JVM内存过小。
程序不严密，产生了过多的垃圾。
程序体现
一般情况下，在程序上的体现为：

内存中加载的数据量过于庞大，如一次从数据库取出过多数据。
集合类中有对对象的引用，使用完后未清空，使得JVM不能回收。
代码中存在死循环或循环产生过多重复的对象实体。
使用的第三方软件中的BUG。
启动参数内存值设定的过小。

### 解决方法

> 从根本上解决Java内存溢出的唯一方法就是修改程序，及时地释放没用的对象，释放内存空间。
>
> 主要思路就是避免程序体现上出现的情况。避免死循环，防止一次载入太多的数据，及时释放无用对象

- 增加JVM的内存大小
  - 对于tomcat容器，找到tomcat在bin目录中的catalina.bat，在linux环境下找到catalina.sh。编辑catalina.bat文件，找到JAVA_OPTS（具体来说是 set "JAVA_OPTS=%JAVA_OPTS% %LOGGING_MANAGER%"）这个选项的位置，这个参数是Java启动的时候，需要的启动参数。
  - 也可以在操作系统的环境变量中对JAVA_OPTS进行设置，因为tomcat在启动的时候，也会读取操作系统中的环境变量的值，进行加载。
  - 如果是修改了操作系统的环境变量，需要重启机器，再重启tomcat，如果修改的是tomcat配置文件，需要将配置文件保存，然后重启tomcat，设置就能生效了。

### 堆内存溢出

java.lang.OutOfMemoryError: Java heap space

#### jstat -gc 进程ID

查看堆内存占用情况

发现年老代内存使用率达近100%（OC和OU分别是年老代空间大小，和年老代已占用空间大小）。

jstat -gc 8052

```
 S0C     S1C       S0U    S1U     EC       EU        OC         OU       PC         PU    YGC     YGCT    FGC    FGCT     GCT  

138240.0 139392.0  0.0    0.0   769280.0 763627.3 5265408.0  5265408.0  262144.0 76198.7    851  135.059 16336 263263.054 263398.112
```

但是每次全量GC后，回收内存较少，还是90%多。初步判断是内存泄漏。

#### jmap -histo 进程ID

查看JVM中的对象数量和占用内存情况，如下： 

 num     #instances         #bytes class name

----------------------------------------------

   1:      40537430     1621497200 java.util.concurrent.ConcurrentHashMap$Segment

   2:      40543201     1297382432 java.util.concurrent.locks.ReentrantLock$NonfairSync

   3:      40537430      989891192 [Ljava.util.concurrent.ConcurrentHashMap$HashEntry;

   4:       5617075      553873688 [C

   5:       7661447      245166304 java.lang.String

   6:       2531665      222786520 org.apache.catalina.session.StandardSession

   7:       2533600      202687328 [Ljava.util.concurrent.ConcurrentHashMap$Segment;

从这段数据来看，内存占用排第一的对象是ConcurrentHashMap$Segment，这个其实就已经很不正常了。正常情况都是[B、[C、String、[I这些排在前面。而现在ConcurrentHashMap$Segment排在第一位，说明某个对象中有ConcurrentHashMap$Segment类型的变量。这个对象不能释放，导致ConcurrentHashMap$Segment越积越多。接着向下看，发现StandardSession对象排在第6位，对象数有253万个。这个是不正常的，应用中保留的Session太多了。查看StandardSession源码也发现，其确实有ConcurrentHashMap类型的变量：protected Map<String, Object> attributes = new ConcurrentHashMap();。看来问题就出在Session身上。查看tomcat的web.xml文件发现，设置的Session超时时间是24小时，这个太长了，改为2小时。同时了解到，客户端访问EPG时，没有带SessionID。导致每个请求，EPG服务器都会创建一个新的Session。

现在这个问题基本上是定位清楚了。但是现实的复杂总是超出你的想象。重启Tomcat，使用jmap -histo 进程ID查看JVM中的对象数量和占用内存情况。发现StandardSession对象还是200多万个，后来逐渐减少。预期情况是重启后StandardSession很少，后来逐渐增加。由于设置里2小时超时。后面GC时，就会释放掉一部分。最大数应该在20万左右。实际情况和预期完全不符。于是又上网搜索了Tomcat的session相关，才知道Tomcat有个配置控制是否持久化session，以防止重启Tomcat使session丢失。Tomcat默认是已经启用持久化配置，若要禁用持久化功能，则只需要在cof文件夹context.xml的<Context>节点里配置<Manager pathname="" />。

### 永久层内存溢

java.lang.OutOfMemoryError: PermGen space

出是启动JVM时没有调整JVM永久层内存大小时，经常发生。一般永久层内存大小默认是64M。如果JVM启动时，加载的类较多，占用内存大小超过64M，就会造成永久层内存溢出。

通过调整JVM的启动参数-XX:PermSize、 -XX:MaxPermSize，可以解决这个问题。

##### 场景

现网在某个新服务器上安装Tomcat时，现场操作人员经常忘记调整JVM的启动参数。然后就会向研发抱怨应用启动不了，或启动了但运行很慢。现在现场操作文档中增加了调整JVM启动参数的步骤。这种问题几乎就没有了。

有的时候堆内存溢出是由于应用一次性从数据库加载数据过大，构建对象过多造成。这种情况就要具体问题，具体分析了。 

## 内存泄露

### 原因

> 存在无效的引用！

### 介绍

Memory Leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存，迟早会被占光。
在Java中，内存泄漏就是存在一些被分配的对象，这些对象有下面两个特点：

1. 首先，这些对象是可达的，即在有向图中，存在通路可以与其相连；

2. 其次，这些对象是无用的，即程序以后不会再使用这些对象。

   如果对象满足这两个条件，这些对象就可以判定为Java中的内存泄漏，这些对象不会被GC所回收，然而它却占用内存。

### 解决

内存泄露是纯代码层面的问题。

### 案例

Full GC

## CPU高，内存占满

黑龙江现场反馈EPG程序访问无应答。远程登录服务器，查看EPG的java进程CPU使用率达100%。

### top -Hp 进程ID

查看有一个线程的CPU使用率尽100%。使用jstack 进程ID>>1.txt命令导出java堆栈，查看对应线程是GC线程。这说明java在拼命回收内存。

### jstat -gc 进程ID

命令查看内存使用情况，发现OC（年老代内存大小）、OU（年老代内存已使用大小）相等，内存果然被用光了。

通过以上信息，可以断定内存用完，java启动GC线程回收内存。但是由于内存对象都是活动状态，导致回收不掉。这个就出现了CPU高，内存占满的情况。CPU高不是主要问题，内存占满才是。内存占用率降下来，CPU使用率自然就会降下来。下面的工作就是找出内存占用率高的原因。

### jmap -histo 进程ID

命令查看java内存中的对象分布情况。如下（部分）：

num     #instances         #bytes  class name

----------------------------------------------

   1:      18906326     1570239408  [C

   2:      32879762     1415144440  [B

   3:      25681574      821810368  java.util.HashMap$Entry

   4:       7871687      639239680  [Ljava.util.HashMap$Entry;

   5:      18972768      607128576  java.lang.String

   6:       7863275      377437200  java.util.HashMap

   7:      10942392      350140768  [[B

   8:      10937478      262499472  com.mysql.jdbc.ByteArrayRow

   9:        101997       87368360  [Ljava.lang.Object;

  10:         57416       32392712  [I

从这个数据可以看出，[C和[B分别占了1.5G和1.4G，太多了。Java内存一共才5G。接着向下看com.mysql.jdbc.ByteArrayRow排在第8位，这个很不正常。ByteArrayRow是数据库查询时生成的对象。在前面导出的java堆栈文件1.txt中搜索ByteArrayRow，找到了好几处。都是查询连续剧剧头和分集对应关系的方法。使用mysql客户端登录mysql数据库，统计剧头和分集对应关系的数据有42万条。

排查到这里问题的原因已经很清楚了。剧头和分集对应关系的数据量较大，而且同时被调用了好几次。导致大量数据停留内存。Java虚拟机发现内存耗光，启动GC回收。由于都是活动对象，无法回收掉。

解决办法：之前想法是将剧头和分集对应关系全部查出，然后缓存起来，以便使用。现在看来数据量较大，不应该一次性全部查询出来。而且从应用场景看，这个关系不会全部都用到，只会用到极小的一部分。现改为按分集contentid查询对应关系，按需查询，并且缓存起来。这样数据量很小，而且很快。改好后，测试验证，没有问题了。