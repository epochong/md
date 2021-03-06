---
title: Linux基础
date: 2019-07-07 09:42:49
tags:
---

## 统一磁盘整体情况，包括磁盘大小，已使用，可用

1.查看当前目录
命令

df -h

结果：

![image-20200313122610751](/Users/wangchong/Library/Application Support/typora-user-images/image-20200313122610751.png)

统一每个目录下磁盘的整体情况

2.查看指定目录
在命令后直接放目录名,比如查看“usr”目录使用情况：

df -h /usr/
结果：

统一了指定目录一使用情况，及分配的最大空间

第二：具体查看文件夹的占用情况。
1.查看当前目录每个文件夹的情况。
命令：

du --max-depth=1 -h 

结果如下：

最后一行统计整体占用多少磁盘

2.指定目录
只要在命令后直接根目录名，以目录“/usr”为例
命令如下:

du --max-depth=1 -h  /usr/
结果如下：


第三：计算文件夹大小
为了快算显示，同时也只是想查看目录整体占用大小。可以直接使用du -sh 命令，如果想查看指定目录，直接在命令后根目录即可。
命令：

du -sh /usr/
结果如下：

第四：总结
其中df -h和du -sh使用的比较多，一个统计整体磁盘情况，一个看单独目录点用情况，而命令du --max-depth=1 -h查看了目录下文件夹占用情况，使用比较少，可以用du -sh代替，而且命令较长，当然并不是说它没用。

## Linux查看程序端口占用情况

使用命令：
ps -aux | grep tomcat
发现并没有8080端口的Tomcat进程。
使用命令：netstat –apn
查看所有的进程和端口使用情况。发现下面的进程列表，其中最后一栏是PID/Program name 
发现8080端口被PID为9658的Java进程占用。
进一步使用命令：ps -aux | grep java，或者直接：ps -aux | grep pid 查看
就可以明确知道8080端口是被哪个程序占用了！然后判断是否使用KILL命令干掉！
方法二：直接使用 netstat   -anp   |   grep  portno
即：netstat -anp|grep 8080

## 网络端口号相关

Netstat 用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。
通过netstat可以查看网络连接、端口号等占用情况
查看TCP/UDP端口：netstat -tuoln
查看进程名运行的端口号：netstat -anp|grep 进程名
当需要监控某个服务的端口号时可以首先获取该服务的监听端口号，如tomact进程
netstat -anp|grep tomcat
根据端口查看运行的进程：
netstat -anp|grep 端口号 或 lsof -i:端口号
通常用于查看某个端口号下建立的连接数，如8083端口号下的连接数统计：
netstat -anp|grep：8083|wc -l 
netstat -tln 查看服务监听端口 
查看进程  ps aux
查看指定服务的进程号,如tomcat服务： ps aux|grep tomcat
结束进程  kill -15 pid 立即释放资源； kill -9 pid 不会立即释放资源

## 用top、ps命令查看进程中的线程

### 方法一：PS

ps命令能提供一份当前进程的快照

在ps命令中，“-T”选项可以开启线程查看。下面的命令列出了由进程号为`<pid>`的进程创建的所有线程。

```ruby
ps -T -p <pid>
```

### 方法二： Top

状态可以自动刷新

top命令可以实时显示各个线程情况。要在top输出中开启线程查看，请调用top命令的“-H”选项，该选项会列出所有Linux线程。在top运行时，你也可以通过按“H”键将线程查看模式切换为开或关。

```
top -H
```

要让top输出某个特定进程<pid>并检查该进程内运行的线程状况：

```
top -H -p <pid>
```

### 方法三： Htop

一个对用户更加友好的方式是，通过htop查看单个进程的线程，它是一个基于ncurses的交互进程查看器。该程序允许你在树状视图中监控单个独立线程。

要在htop中启用线程查看，请开启htop，然后按<F2>来进入htop的设置菜单。选择“设置”栏下面的“显示选项”，然后开启“树状视图”和“显示自定义线程名”选项。按<F10>退出设置。

### 如何理解awk？

**答：**awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。

简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

## linux系统中如何查看日志？

**答：**

> tail -f -n 5 /var/log/syslog
>
> cat /var/log/syslog
>
> more  /var/log/syslog
>
> less  /var/log/syslog

## 统计文件中“abc”有多少个？

grep abc **.log | wc -l

# 解决线上问题

## 先排查运行环境

首先要强调的是，有些问题不是疑难问题，或是伪疑难问题。其实就是些运行环境的问题，磁盘空间、内存大小、CPU占用、数据库连接、用户权限等问题。如果有人向我反馈某个软件启动不了、启动后运行很慢、启动了但整体功能都不正常等问题，首先会使用df -h命令查磁盘空间，使用free -m命令查内存使用情况，使用top命令查看CPU占用情况，使用mysql命令登录数据库show processlist命令查看数据库连接情况。不要以为这样做无意义。通过这些命令解决了问题，就知道这样做多重要。比如说磁盘满了的问题，一般是软件开始时运行是正常的。但是过了一段时间，比如几个月或几年，突然就出现运行慢、重启不了等怪问题了。

某年春节过后，四川移动报告MA（媒资）系统运行异常缓慢。登录到MA服务器排查，发现MA服务器一切正常。由于MA需要数据库，而数据库在另一台服务器上。先使用mysql命令登录数据库show processlist命令查看数据库连接情况。发现有大量链接处于“query end”状态。上网搜索“mysql query end”，说有可能是磁盘空间满了。然后登录数据库服务器，使用df -h命令查磁盘空间，果然满了。Mysql做同步时，会产生大量的二进制日志。不及时清理就会造成磁盘空间占满。Mysql的配置expire_logs_days，可以设置保留日志的天数。设置为30，就不用再为Mysql的日志过多而苦恼了。

从上面的例子可以看出，这些运行环境引起的问题，排查和解决都不难。这类问题解决的重点是要往运行环境上想。如果不查环境，而是看软件日志、查源码什么的。看了大半天也看不出问题。然后才想起查一下环境。然后发现果然是环境问题。那不是要吐血。

但是问题来了，什么样的问题是运行环境引起的问题呢？被定位了是运行环境引起的问题，就是运行环境引起的问题。也就是说运行环境引起的问题没什么特别的现象。所以正如前面强调过的，遇到整体性的软件问题，先查一下环境。

## 应用的CPU占用过高问题

这种问题的定位过程，网上说的比较多。对一般情况，本文只简单列一下定位过程。1、使用top命令查询出CPU使用率较高的进程ID。2、使用top -H -p 进程ID -c命令查询出CPU使用率较高的线程ID，注意是线程ID。一般情况只会有一个线程的CPU使用率较高。3、使用JDK带的jstack 进程ID >> java.txt命令，查出java进程的线程栈方法调用、运行情况。这个命令的结果记得要保存到文件中，而且最好重复三次，保存到3个文件中。4、由于文件中的线程号是十六进制，需要将第二步得到的线程ID转为十六进制，然后在文件中搜索，找到对应的线程栈信息。查看对应的方法调用、运行情况。大概都是应用不小心死循环了、排序算法效率不高、从数据库获取的数据太多等问题引起。

这种一般问题还是比较好定位的，解决的难易，就只能各由天命了。下面说说不一般的情况。上面第二步提到，一般情况只会有一个线程的CPU使用率较高。如果是好几个线程的CPU使用率不是特别高，但是几个加起来就很高的情况，怎么办？如某个Java进程CPU使用率达200%，而查线程时发现，前5个线程的CPU使用率都是30%多。这种情况，我只遇到过一次。我不清楚是否还有其它原因会导致这种情况。在这里，将我遇到的问题的定位、解决描述一下，其它情况用这个思路估计也有帮助。

2017年元旦后上班，发现公司有一台服务器上的tomcat进程占用CPU很高。重启tomcat也不行。使用top -H -p 进程ID -c命令查询，发现有好几个线程的CPU使用率比较高。后来测试也发现别的服务器有此现象。现场客户那里也发现了相同现象。用strace -p 进程ID命令查询系统调用，结果如图：

查了一下，果然在2017年1月1日，闰秒了。重启服务器可以解决。如果不想重启服务器，执行/etc/init.d/ntpd stop; date -s now。

## 数据库慢查询紧急解决办法

某现场报应用运行很慢，登录都要好几分钟。检查应用运行环境，一切正常。这个现场有个特殊情况是好几个应用的数据库都放到了一台数据库服务器上。使用mysql命令行客户端登录到数据库服务器的mysql上，使用show processlist查看。发现另一个应用的查询命令执行了十几分钟了，还在执行。这是非常不正常的。这个慢查询占用大量服务器资源，导致其它链接的查询变得很慢。其实查询超过一秒的，就算很慢了。

因为是现场应用，需要快速恢复，所以要kill掉慢查询的链接。先使用show full processlist命令，查询出慢查询的全部sql语句，并保存到文件中。然后使用kill 链接ID命令，杀掉慢查询的链接。链接ID就是show processlist命令中的Id列。注意，这个kill不是linux操作系统的，而是Mysql的，要在Mysql的命令行下运行。杀掉这个慢查询的链接后，所有的应用的运行速度都正常了。

## Full GC造成应用停止

对于要求实时响应客户请求的应用，JVM的FGC，会停止应用（stop-the-world）。这会造成用户请求超时。可以通过增加如下参数，将JVM年老代的GC改为CMS并发收集。

-XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=2 -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70

-XX:+UseConcMarkSweepGC  设置年老代为CMS并发收集。并行GC方式在进行Full GC时，会停止整个应用，会造成外部请求无响应的情况。此CMS并发GC方式可减少应用的暂停时间，主要适合场景是对响应时间的重要性需求大于对吞吐量的要求。

-XX:CMSFullGCsBeforeCompaction=5 由于CMS并发收集器不对内存空间进行压缩,整理,所以运行一段时间以后会产生"碎片",使得运行效率降低.此值设置运行多少次GC以后对内存空间进行压缩,整理.

-XX:+UseCMSCompactAtFullCollection 在FULL GC的时候， 对年老代的压缩

-XX:CMSInitiatingOccupancyFraction=70 年老代内存使用率到70％后开始CMS收集

这里需要强调的是，这样设置并不是将FGC变为了CMS，FGC还是存在的。如果JVM判断有必要的话，还是会启动FGC。Tomcat的某些版本有个设置，每隔一个小时调用一次System.gc()，这个会触发FGC。可使用如下操作之一：

1、增加JVM参数-XX:+DisableExplicitGC，这个参数会使显示的调用System.gc()空转，不会执行垃圾回收

2、不增加JVM参数-XX:+DisableExplicitGC，换成增加-XX:+ExplicitGCInvokesConcurrent，使FULL GC使用并发垃圾回收器CMS，提高回收效率（CMS并发GC，stop-the-world时间较短）

3、修改server.xml配置，gcDaemonProtection参数改为false，默认是true

 <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"

gcDaemonProtection="false"/> 

# Linux中的nohup与2>&1 &

nohup是Linux的一个常用命令，当你想要在退出账户或者关闭终端后进程仍在运行时，就可以使用nohup命令。nohup就是不挂断的意思（no hang up）。

nohup的一般形式为：

```
nohup command>file 2>&1 &
```

如果不将 nohup 命令的输出重定向，输出将附加到当前目录的 nohup.out 文件中，否则就是自己指定的文件。

尾部的`&`是把该命令以后台的job的形式运行，那么`2>&1`是什么意思？

## 基本符号与含义

- 0 表示stdin标准输入
- 1 表示stdout标准输出
- 2 表示stderr标准错误

## command>file

这个命令其实是一个缩写，实际上是command产生的**标准输出**重定向到file中，也就是说相当于执行了`command 1 > file`。

## 2>&1

2是标准错误，1是标准输出，&的意思是**等效于**。实际就是把标准错误也重定向到file中，那么这样写和分别重定向有什么区别呢？

### command>a 2>&1与command>a 2>a的区别

经过上面的分析，`command>a 2>&1`这条命令，等价于`command 1>a 2>&1`，也就是说执行command产生的标准输入重定向到文件a中，标准错误也重定向到文件a中，那么是否等价于`command 1>a 2>a`呢？其实不是，区别在于前者只打开一次文件a，后者会打开两次并导致标准输出被标准错误**覆盖**。

`&1`的含义就可以理解为用标准输出的引用，引用的就是重定向标准输出产生打开的a。从IO效率上来讲，`command 1>a 2>&1`比`command 1>a 2>a`的效率更高。

# VIM

1、使用vim删除换行符

vim输入命令：%s/\n//g

# 日志

## 查询时间范围

sed -n '/2021-02-01 10:59:40/,/2021-02-01 10:59:41/p' request.log
