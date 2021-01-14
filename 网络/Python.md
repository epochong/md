## Python和Java的区别

#### 1. 变量的使用

从类型上说，Java是静态类型语言，Python是动态类型语言。所谓静态类型就是变量需要先声明再使用，动态类型是不需要事先声明变量的类型。
例如在Java中声明变量：

```java
int var = 0;
```

我们需要先确定变量的类型，再为变量赋值。而在Python中，变量无需事先声明：

```python
var = 0
```

可以说是拿起来就能用，正是因此Python 的语法要比Java 更灵活。

#### 2. 忘掉分号

Java中语句的结束强制以";"为结尾，Python中我们当然也可以用分号，但并不建议这样用。通常在Python 中我们用换行表示语句的结束。

#### 3. 输出语句

如果你有过在Java 代码中大量拼接字符串的体验，那么你可能会爱上Python的语法。在python中打印变量需要使用占位符，如：

```python
print("This is a %s"% ("dog"))
```

#### 4. 数组和列表

Java 中的数组是很很实用的数据结构，Python 中同样有类似的数据结构。我们用代码对比两个语言的差异：

- java:

```java
int[] array={1, 2, 3, 4, 5};
```

- Python:

```python
list = [1, 2, 3, 4, 5 ]
```

不过由于Python 是动态数据类型 ，所以在list中的元素可以是不同的数据类型：

```python
list=[1, 2, "a", "b", "c"]
```

### 其他

**一、**python虚拟机没有java强，java虚拟机是java的核心，python的核心是可以很方便地使用c语言函数或c++库。


**二、**python是全动态性的，可以在运行时自己修改自己的代码，java只能通过变通方法实现。python的变量是动态的，而java的变量是静态的，需要事先声明，所以java ide的代码提示功能优于python ide。


三，python的产生几十年了，几十年前面向过程是主流，所以用python有好多程序用的是面向过程设计方法，很多概念从c语言过来的，class在python中是后加入的，而java是为了实现没有指针的c++（当年com组件用的引用记数，java用的虚拟机），主要采用面向对象的设计方法，很多概念是oop的概念。面向过程，相对简洁直观，但容易设计出面条程序，面向对象，相对抽象优雅，但容易过度抽象。


四，在实际使用的python入门简单，但要学会用python干活，需要再学习python各种库，pyhton的强大在于库，为什么python的库强大，原因是python的库可以用python，c语言,c++等设计，再提供给python使用，所以无论gpu运行，神经网络，智能算法，数据分析，图像处理，科学计算，各式各样的库在等着你用。而java没有python那么多的开源库，很多库是商业公司内部使用，或发布出来只是一个jar包，看不到原始代码。python虚拟机因为编译性没有java的支持的好（或者说故意这么设计的），一般直接使用源码（linux），或源码简单打个包（如pyexe）。


五、python有很多虚拟机实现，如cython,Pyston,pypy,jython, IronPython等等，适合用于业务语言，或插件语言，或面向领域语言，而java因为虚拟机巨大，很少用于插件语言，发布也不方便。


六、java主要用于商业逻辑强的领域，如商城系统，erp，oa,金融，保险等传统数据库事务领域，通过类似ssh框架事务代码，对商业数据库，如oralce,db2,sql server等支持较好，软件工程理念较强，适合软件工程式的多人开发模式。python主要用于web数据分析，科学计算，金融分析，信号分析，图像算法，数学计算，统计分析，算法建模，服务器运维，自动化操作，快速开发理念强，适合快速开发团队或个人敏捷模式。


七、java的商业化公司支持多，如sap,oracle,ibm等，有商业化的容器，中间件，企业框架ejb。python的开源组织支持多，如qt,linux,google,很多开源程序都支持python， 如pyqt,redis,spark等。


八、python用途最多的是脚本，java用途最多的是web，pyhotn是胶水，可以把各类不相关的东西粘在一起用，java是基佬，可以通过软件工程组成几百个人的团队和你pk，商业化气息重。不过我认为还是python强大，因为可以方便调用c或c++的库，但软件工程和商业化运作没有java好，适合快捷开发。


九，关于钱。
如果你想写程序卖软件用java，可用上ibm服务器，上oracle数据库，上EMC存储,价格高，商业采购公司喜欢这种高大上。如果你要直接用程序生成金钱用python，python可以实现宽客金融，数据回测，炒股，炒期权，炒黄金，炒比特币，对冲套利，统计套利，有很多开源库，数据分析库，机器学习库可以参考。


十、java和python，都可以运行于linux操作系统，但很多linux可以原生支持python,java需要自行安装。java和python强于c#的原因大于支持linux，支持osx，支持unix，支持arm。java和python比c++受欢迎的原因在于不需要指针。


十一、对于移动互联网，python只能通过运行库运行于安卓或ios，java原生支持安卓开发，但不能用ios中。


十二、对于大数据，hadoop用java开的, spark用Scala开发，用python调用spark再分析更方便。

## 多线程

“**对于任何Python程序，不管有多少的处理器，任何时候都总是只有一个线程在执行**”，即Python中的多线程是“**假的多线程**”

### 解释型语言

编译性语言例如c语言：用c语言开发了程序后，需要通过编译器把程序编译成机器语言（即计算机识别的二进制文件，因为不同的操作系统计算机识别的二进制文件是不同的），所以c语言程序进行移植后，要重新编译。（如windows编译成ext文件，linux编译成erp文件）。

Java与Python都是解释型语言，而相对的,解释性语言编写的程序不进行预先编译，以文本方式存储程序代码。在发布程序时，看起来省了道编译工序。但是，在运行程序的时候，解释性语言必须先解释再运行。例如java语言，java程序首先通过编译器编译成class文件，如果在windows平台上运行，则通过windows平台上的java虚拟机（VM）进行解释。如果运行在linux平台上，则通过linux平台上的java虚拟机进行解释执行。所以说能跨平台，前提是平台上必须要有相匹配的java虚拟机。如果没有java虚拟机，则不能进行跨平台。

前者由于程序执行速度快，同等条件下对系统要求较低，因此像开发操作系统、大型应用程序、数据库系统等时都采用它，像C/C++、Pascal/Object Pascal（Delphi）等都是编译语言，而一些网页脚本、服务器脚本及辅助开发接口这样的对速度要求不高、对不同系统平台间的兼容性有一定要求的程序则通常使用解释性语言，如Java、JavaScript、VBScript、Perl、Python、Ruby、MATLAB 等等。

### 造成Python没有真正意义上的多线程的原因

作为解释型语言，Python的解释器须要做到既安全又高效。多线程编程，解释器要留意的是避免在不同的线程操作内部共享的数据，同时它还要保证在管理用户线程时保证总是有最大化的计算资源。那么，不同线程同时访问时，数据的保护机制是解释器全局锁。这是一个加在解释器上的全局（从解释器的角度看）锁。

（但是这也导致了上面提到的问题：对于任何Python程序，不管有多少的处理器，任何时候都总是只有一个线程在执行（这点上Python是不同于Java的，对于多核的情况，Java可以同时开启多个线程进行处理，具体例子可以参考下面例子）。

所以，对于计算密集型的，我还是建议不要使用python的多线程而是使用多进程方式，而对于IO密集型的，还是建议使用多进程方式，因为使用多线程方式出了问题

### Python多进程库multiprocessing

基于Process的多进程
multiprocessing模块提供process类实现新建进程。下述代码是新建一个子进程。

```python
from multiprocessing import Process

def f(name):
    print 'hello', name

if __name__ == '__main__':
    p = Process(target=f, args=('bob',)) # 新建一个子进程p，目标函数是f，args是函数f的参数列表
    p.start() # 开始执行进程
    p.join() # 等待子进程结束
```


上述代码中p.join()的意思是等待子进程结束后才执行后续的操作，一般用于进程间通信。例如有一个读进程pw和一个写进程pr，在调用pw之前需要先写pr.join()，表示等待写进程结束之后才开始执行读进程。

基于进程池Pool的多进程
如果要同时创建多个子进程可以使用multiprocessing.Pool类。该类可以创建一个进程池，然后在多个核上执行这些进程。

```python
import multiprocessing
import time

def func(msg):
    print multiprocessing.current_process().name + '-' + msg

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=4) # 创建4个进程
    for i in xrange(10):
        msg = "hello %d" %(i)
        pool.apply_async(func, (msg, ))
    pool.close() # 关闭进程池，表示不能在往进程池中添加进程
    pool.join() # 等待进程池中的所有进程执行完毕，必须在close()之后调用
    print "Sub-process(es) done."
```

结果：

Sub-process(es) done.
PoolWorker-34-hello 1
PoolWorker-33-hello 0
PoolWorker-35-hello 2
PoolWorker-36-hello 3
PoolWorker-34-hello 7
PoolWorker-33-hello 4
PoolWorker-35-hello 5
PoolWorker-36-hello 6
PoolWorker-33-hello 8
PoolWorker-36-hello 9

Pool可以使用apply()函数或者map函数进行数据的处理协程

#### 协程概念

又称微线程，纤程，英文名Coroutine。协程的作用，是在执行函数A时，可以随时中断，去执行函数B，然后中断继续执行函数A（可以自由切换）。但这一过程并不是函数调用（没有调用语句），这一整个过程看似像多线程，然而协程只有一个线程执行。

#### 优势

- 执行效率极高，因为子程序切换（函数）不是线程切换，由程序自身控制，没有切换线程的开销。所以与多线程相比，线程的数量越多，协程性能的优势越明显。
- 不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在控制共享资源时也不需要加锁，因此执行效率高很多。

　　*说明：协程可以处理IO密集型程序的效率问题，但是处理CPU密集型不是它的长处，如要充分发挥CPU利用率可以结合多进程+协程。*

　　以上只是协程的一些概念，可能听起来比较抽象，那么我结合代码讲一讲吧。这里主要介绍协程在Python的应用，Python2对协程的支持比较有限，生成器的yield实现了一部分但不完全，gevent模块倒是有比较好的实现；Python3.4以后引入了asyncio模块，可以很好的使用协程。

### 协程VS多线程

当线程越来越多时，多线程主要的开销花费在线程切换上，而协程是在一个线程内切换的，因此开销小很多，这也许就是两者性能的根本差异之处吧。（个人观点）

### 异步爬虫

也许关心协程的朋友，大部分是用其写爬虫（因为协程能很好的解决IO阻塞问题），然而我发现常用的urllib、requests无法与asyncio结合使用，可能是因为爬虫模块本身是同步的（也可能是我没找到用法）。那么对于异步爬虫的需求，又该怎么使用协程呢？或者说怎么编写异步爬虫？
给出几个我所了解的方案：

- grequests （requests模块的异步化）
- 爬虫模块+gevent（比较推荐这个）
- aiohttp （这个貌似资料不多，目前我也不太会用）
- asyncio内置爬虫功能 （这个也比较难用）