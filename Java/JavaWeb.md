# Servlet

## 单实例多线程模式

Servlet如何处理多个请求访问？
Servlet容器默认是采用单实例多线程的方式处理多个请求的：
1.当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)；
2.容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的<Connector>设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等。
3.当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者；
4.线程执行Servlet的service方法；
5.请求结束，放回线程池，等待被调用；
（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）

### 为什么这样设计

第一：Servlet单实例，减少了产生servlet的开销；
第二：通过线程池来响应多个请求，提高了请求的响应时间；
第三：Servlet容器并不关心到达的Servlet请求访问的是否是同一个Servlet还是另一个Servlet，直接分配给它一个新的线程；如果是同一个Servlet的多个请求，那么Servlet的service方法将在多线程中并发的执行；
第四：每一个请求由ServletRequest对象来接受请求，由ServletResponse对象来响应该请求；

Servlet/JSP技术和ASP、PHP等相比，由于其多线程运行而具有很高的执行效率。由于Servlet/JSP默认是以多线程模式执行的，所以，在编写代码时需要非常细致地考虑多线程的安全性问题。 

### JSP的中存在的多线程问题： 

当客户端第一次请求某一个JSP文件时，服务端把该JSP编译成一个CLASS文件，并创建一个该类的实例，然后创建一个线程处理CLIENT端的请求。如果有多个客户端同时请求该JSP文件，则服务端会创建多个线程。每个客户端请求对应一个线程。以多线程方式执行可大大降低对系统的资源需求,提高系统的并发量及响应时间. 

对JSP中可能用的的变量说明如下: 
实例变量: 实例变量是在堆中分配的,并被属于该实例的所有线程共享，所以不是线程安全的. 
JSP系统提供的8个类变量 
JSP中用到的OUT,REQUEST,RESPONSE,SESSION,CONFIG,PAGE,PAGECONXT是线程安全的(因为每个线程对应的request，respone对象都是不一样的，不存在共享问题), APPLICATION在整个系统内被使用,所以不是线程安全的. 

局部变量: 局部变量在堆栈中分配,因为每个线程都有它自己的堆栈空间,所以是线程安全的. 
静态类: 静态类不用被实例化,就可直接使用,也不是线程安全的. 

外部资源: 在程序中可能会有多个线程或进程同时操作同一个资源(如:多个线程或进程同时对一个文件进行写操作).此时也要注意同步问题. 

使它以单线程方式执行,这时，仍然只有一个实例，所有客户端的请求以串行方式执行。这样会降低系统的性能 

问题 
问题一. 说明其Servlet容器如何采用单实例多线程的方式来处理请求 
问题二. 如何在开发中保证servlet是单实例多线程的方式来工作(也就是说如何开发线程安全的servelt)。 

### Servlet容器如何同时来处理多个请求 

Java的内存模型JMM（Java Memory Model） 
JMM主要是为了规定了线程和内存之间的一些关系。根据JMM的设计，系统存在一个主内存(Main Memory)，Java中所有实例变量都储存在主存中，对于所有线程都是共享的。每条线程都有自己的工作内存(Working Memory)，工作内存由缓存和堆栈两部分组成，缓存中保存的是主存中变量的拷贝，缓存可能并不总和主存同步，也就是缓存中变量的修改可能没有立刻写到主存中；堆栈中保存的是线程的局部变量，线程之间无法相互直接访问堆栈中的变量。根据JMM，我们可以将论文中所讨论的Servlet实例的内存模型抽象为图所示的模型。 

 

工作者线程Work Thread:执行代码的一组线程。 
调度线程Dispatcher Thread：每个线程都具有分配给它的线程优先级,线程是根据优先级调度执行的。 

Servlet采用多线程来处理多个请求同时访问。servlet依赖于一个线程池来服务请求。线程池实际上是一系列的工作者线程集合。Servlet使用一个调度线程来管理工作者线程。 

当容器收到一个Servlet请求，调度线程从线程池中选出一个工作者线程,将请求传递给该工作者线程，然后由该线程来执行Servlet的service方法。当这个线程正在执行的时候,容器收到另外一个请求,调度线程同样从线程池中选出另一个工作者线程来服务新的请求,容器并不关心这个请求是否访问的是同一个Servlet.当容器同时收到对同一个Servlet的多个请求的时候，那么这个Servlet的service()方法将在多线程中并发执行。 
Servlet容器默认采用单实例多线程的方式来处理请求，这样减少产生Servlet实例的开销，提升了对请求的响应时间，对于Tomcat可以在server.xml中通过<Connector>元素设置线程池中线程的数目。 

就实现来说： 
调度者线程类所担负的责任如其名字，该类的责任是调度线程，只需要利用自己的属性完成自己的责任。所以该类是承担了责任的，并且该类的责任又集中到唯一的单体对象中。而其他对象又依赖于该特定对象所承担的责任，我们就需要得到该特定对象。那该类就是一个单例模式的实现了。 

注意：服务器可以使用多个实例来处理请求，代替单个实例的请求排队带来的效益问题。服务器创建一个Servlet类的多个Servlet实例组成的实例池，对于每个请求分配Servlet实例进行响应处理，之后放回到实例池中等待下此请求。这样就造成并发访问的问题。 
此时,局部变量(字段)也是安全的，但对于全局变量和共享数据是不安全的，需要进行同步处理。而对于这样多实例的情况SingleThreadModel接口并不能解决并发访问问题。 SingleThreadModel接口在servlet规范中已经被废弃了。

### 如何开发线程安全的Servlet 

　　1、实现 SingleThreadModel 接口 
　　该接口指定了系统如何处理对同一个Servlet的调用。如果一个Servlet被这个接口指定,那么在这个Servlet中的service方法将不会有两个线程被同时执行，当然也就不存在线程安全的问题。这种方法只要将前面的Concurrent Test类的类头定义更改为：

```
Public class Concurrent Test extends HttpServlet implements SingleThreadModel { 
………… 
} 
```

　　2、同步对共享数据的操作 

　　使用synchronized 关键字能保证一次只有一个线程可以访问被保护的区段，在本论文中的Servlet可以通过同步块操作来保证线程的安全。同步后的代码如下：

```
………… 
Public class Concurrent Test extends HttpServlet { ………… 
Username = request.getParameter ("username"); 
Synchronized (this){ 
Output = response.getWriter (); 
Try { 
Thread. Sleep (5000); 
} Catch (Interrupted Exception e){} 
output.println("用户名:"+Username+"<BR>"); 
} 
} 
} 
```

　　3、避免使用实例变量 

　　本实例中的线程安全问题是由实例变量造成的，只要在Servlet里面的任何方法里面都不使用实例变量，那么该Servlet就是线程安全的。 

　　修正上面的Servlet代码，将实例变量改为局部变量实现同样的功能，代码如下：

```
…… 
Public class Concurrent Test extends HttpServlet {public void service (HttpServletRequest request, HttpServletResponse 
Response) throws ServletException, IOException { 
Print Writer output; 
String username; 
Response.setContentType ("text/html; charset=gb2312"); 
…… 
} 
} 
```

对上面的三种方法进行测试，可以表明用它们都能设计出线程安全的Servlet程序。但是，如果一个Servlet实现了SingleThreadModel接口，Servlet引擎将为每个新的请求创建一个单独的Servlet实例，这将引起大量的系统开销。SingleThreadModel在Servlet2.4中已不再提倡使用；同样如果在程序中使用同步来保护要使用的共享的数据，也会使系统的性能大大下降。这是因为被同步的代码块在同一时刻只能有一个线程执行它，使得其同时处理客户请求的吞吐量降低，而且很多客户处于阻塞状态。另外为保证主存内容和线程的工作内存中的数据的一致性，要频繁地刷新缓存,这也会大大地影响系统的性能。所以在实际的开发中也应避免或最小化 Servlet 中的同步代码；在Serlet中避免使用实例变量是保证Servlet线程安全的最佳选择。从Java 内存模型也可以知道，方法中的临时变量是在栈上分配空间，而且每个线程都有自己私有的栈空间，所以它们不会影响线程的安全。
更加详细的说明：

1，变量的线程安全：这里的变量指字段和共享数据(如表单参数值)。 
a，将 参数变量 本地化。多线程并不共享局部变量.所以我们要尽可能的在servlet中使用局部变量。 
例如：String user = ""; 
user = request.getParameter("user"); 

b，使用同步块Synchronized，防止可能异步调用的代码块。这意味着线程需要排队处理。在使用同板块的时候要尽可能的缩小同步代码的范围，不要直接在sevice方法和响应方法上使用同步，这样会严重影响性能。 

2,属性的线程安全：ServletContext，HttpSession，ServletRequest对象中属性。 
ServletContext：（线程是不安全的） 
ServletContext是可以多线程同时读/写属性的，线程是不安全的。要对属性的读写进行同步处理或者进行深度Clone()。所以在Servlet上下文中尽可能少量保存会被修改（写）的数据，可以采取其他方式在多个Servlet中共享，比方我们可以使用单例模式来处理共享数据。 
HttpSession：（线程是不安全的） 
HttpSession对象在用户会话期间存在，只能在处理属于同一个Session的请求的线程中被访问，因此Session对象的属性访问理论上是线程安全的。 
当用户打开多个同属于一个进程的浏览器窗口，在这些窗口的访问属于同一个Session，会出现多次请求，需要多个工作线程来处理请求，可能造成同时多线程读写属性。这时我们需要对属性的读写进行同步处理：使用同步块Synchronized和使用读/写器来解决。 
ServletRequest：（线程是安全的） 
对于每一个请求，由一个工作线程来执行，都会创建有一个新的ServletRequest对象，所以ServletRequest对象只能在一个线程中被访问。ServletRequest是线程安全的。注意：ServletRequest对象在service方法的范围内是有效的，不要试图在service方法结束后仍然保存请求对象的引用。 

4，不要在Servlet中创建自己的线程来完成某个功能。 
Servlet本身就是多线程的，在Servlet中再创建线程，将导致执行情况复杂化，出现多线程安全问题。 

5，在多个servlet中对外部对象(比方文件)进行修改操作一定要加锁，做到互斥的访问。 

6，javax.servlet.SingleThreadModel接口是一个标识接口，如果一个Servlet实现了这个接口，那Servlet容器将保证在一个时刻仅有一个线程可以在给定的servlet实例的service方法中执行。将其他所有请求进行排队。 


PS：
Servlet并非只是单例的. 当container开始启动,或是客户端发出请求服务时,Container会按照容器的配置负责加载和实例化一个Servlet（也可以配置为多个，不过一般不这么干）.不过一般来说一个servlet只会有一个实例。
1) Struts2的Action是原型，非单实例的；会对每一个请求,产生一个Action的实例来处理。 
2) Struts1的Action,Spring的Ioc容器管理的bean 默认是单实例的. 

Struts1 Action是单实例的，spring mvc的controller也是如此。因此开发时要求必须是线程安全的，因为仅有Action的一个实例来处理所有的请求。单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心。Action资源必须是线程安全的或同步的。 
Spring的Ioc容器管理的bean 默认是单实例的。
Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题。（实际上，servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题）。
当Spring管理Struts2的Action时，bean默认是单实例的，可以通过配置参数将其设置为原型。(scope="prototype ）

### Servlet的生命周期

大致分为4部：Servlet类加载-->实例化-->服务-->销毁

> 1、Web Client向Servlet容器(Tomcat)发出Http请求。
>
> 2、Servlet容器接收Client端的请求。
>
> 3、Servlet容器创建一个HttpRequest对象，将Client的请求信息封装到这个对象中。
>
> 4、Servlet创建一个HttpResponse对象。
>
> 5、Servlet调用HttpServlet对象的service方法，把HttpRequest对象和HttpResponse对象作为参数传递给HttpServlet对象中。
>
> 6、HttpServlet调用HttpRequest对象的方法，获取Http请求，并进行相应处理。
>
> 7、处理完成HttpServlet调用HttpResponse对象的方法，返回响应数据。
>
> 8、Servlet容器把HttpServlet的响应结果传回客户端。

​    其中的3个方法说明了Servlet的生命周期：

> 1、init()：负责初始化Servlet对象。
>
> 2、service()：负责响应客户端请求。
>
> 3、destroy()：当Servlet对象退出时，负责释放占用资源。

Servlet的生命周期由Servlet容器管理；

三个概念的理解：

Servlet容器<Web容器<应用服务器？

Servlet容器的主要任务就是管理Servlet的生命周期；

Web容器也称之为web服务器，主要任务就是管理和部署web应用的；

应用服务器的功能非常强大，不仅可以管理和部署web应用，也可以部署EJB应用，实现容器管理的事务等等。


Web服务器就是跟基于HTTP的请求打交道，而EJB容器更多是跟数据库，事务管理等服务接口交互，所以应用服务器的功能是很多的。

常见的web服务器就是Tomcat,但Tomcat同样也是Servlet服务器；

常见的应用服务器有WebLogic，WebSphere，但都是收费的；

没有Servlet容器，可以用Web容器直接访问静态Html页面，比如安装了apache等；如果需要显示Jsp/Servlet，就需要安装一个Servlet容器；但是光有servlet容器也是不够的，它需要被解析为html显示，所以仍需要一个web容器；所以，我们常把web容器和Servlet容器视为一体，因为他们两个容器都有对方的功能实现了，都没有独立的存在了，比如tomcat！

### 实际问题

1、如果不同的2个用户同时对这个网站的不同业务同时发出请求（如注册和登陆），那容器里有几个servlet呢？

容器里有2个servlet（当然，这是在“一个servlet对应一种业务请求”的前提下，如果你要把两个业务逻辑写在同一个servlet内另当别论了）

2、不同的用户同时对同一个业务（如注册）发出请求，那这个时候容器里产生的有是几个servlet实例呢？

只有一个servlet实例。一个servlet是在第一次被访问时加载到内存并实例化的。同样的业务请求共享一个servlet实例。不同的业务请求一般对应不同的servlet。如果一个网站要被几千万人同时登录，如果创建几千万个实例的话服务器还怎么跑得动？

答案:

引子：一个web容器，可以有多个servlet。 对提交到同一个servlet类的多个业务请求，共享一个servlet对象（即这个servlet类只被实例化一次，但别忘了，请求还可以从一个servlet forward到另一个servlet,因此一个请求是可以产生多个servlet的，但是由不同的servlet类实例化的，每个servlet类都只被实例化一次，直到应用程序终止或服务器shutdown

## SimpleDateFormat不是线程安全的

多线程情况下，如web，一个请求就是一个线程，使用同一个SimpleDateFormat会报错（也不是每次都报错，概率比较低的。）。解决办法是不使用同一个。每个线程要格式化日期时，自己新建一个SimpleDateFormat对象。

## Listener,Filter,Servlet执行顺序和生命周期

Listener、Filter、Servlet定义：

### Listener:

首先定义一个Listener，实现以下接口：

HttpSessionListener(用来监控session的创建，销毁等)

ServletRequestListener(用于监控servlet上下文request)

ServletRequestAttributeListener(用于监控request中的attribute的操作)

```java
package com.xy.web.listener;
import javax.servlet.ServletRequestAttributeEvent;
import javax.servlet.ServletRequestAttributeListener;
import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class TestListener implements HttpSessionListener,ServletRequestListener,ServletRequestAttributeListener {
    private Logger logger = LoggerFactory.getLogger(TestListener.class);
    
    //sessionListener start!
    public void sessionCreated(HttpSessionEvent arg0) {
        logger.info(".......TestListener sessionCreated().......");
    }
 
    public void sessionDestroyed(HttpSessionEvent arg0) {
        logger.info(".......TestListener sessionDestroyed().......");
    }
    //sessionListener end!
    
    //requestListener start!
    public void requestInitialized(ServletRequestEvent arg0) {
        logger.info("......TestListener requestInitialized()......");
    }
 
    public void requestDestroyed(ServletRequestEvent arg0) {
        logger.info("......TestListener requestDestroyed()......");
    }
    //requestListener end!
    
    //attributeListener start!
    public void attributeAdded(ServletRequestAttributeEvent srae) {
        logger.info("......TestListener attributeAdded()......");
    }
 
    public void attributeRemoved(ServletRequestAttributeEvent srae) {
        logger.info("......TestListener attributeRemoved()......");
    }
 
    public void attributeReplaced(ServletRequestAttributeEvent srae) {
        logger.info("......TestListener attributeReplaced()......");
    }
    //attributeListener end!
}
```

### Filter

```
package com.xy.web.filter;
 
import java.io.IOException;
 
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class TestFilter implements Filter {
	private Logger logger = LoggerFactory.getLogger(TestFilter.class);
 
	public void destroy() {
		logger.info("..............execute TestFilter destory()..............");
	}
 
	public void doFilter(ServletRequest arg0, ServletResponse arg1,FilterChain arg2) throws IOException, ServletException {
		logger.info("..............execute TestFilter doFilter()..............");
		arg2.doFilter(arg0, arg1);
	}
 
	public void init(FilterConfig arg0) throws ServletException {
		logger.info("..............execute TestFilter  init()..............");
	}
}
```

### Servlet

```java
package com.xy.web.servlet;
 
import java.io.IOException;
 
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public class TestServlet extends HttpServlet {
 
	private Logger logger = LoggerFactory.getLogger(TestServlet.class);
 
	/**
	 * 
	 */
	private static final long serialVersionUID = -4263672728918819141L;
 
	@Override
	public void init() throws ServletException {
		logger.info("...TestServlet init() init..........");
		super.init();
	}
	
	@Override
	public void destroy() {
		logger.info("...TestServlet init() destory..........");
		super.destroy();
	}
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		this.doPost(req, resp);
	}
	
	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		logger.info("...TestServlet doPost() start..........");
		//操作attribute
		request.setAttribute("a", "a");
		request.setAttribute("a", "b");
		request.getAttribute("a");
		request.removeAttribute("a");
		//操作session
		request.getSession().setAttribute("a", "a");
		request.getSession().getAttribute("a");
		request.getSession().invalidate();
		logger.info("...TestServlet doPost() end..........");
	}
}
```

**从启动日志来看，**

**启动的顺序为listener->Filter->servlet.**

**简单记为：理(Listener)发(Filter)师(servlet).**

执行的顺序不会因为三个标签在配置文件中的先后顺序而改变。

### 生命周期

日志：
访问项目路径：http://localhost/MyWebProject/common/test.do，访问action两次，打断点后查看日志情况：
第一次访问：

2016-01-14 00:03:03,991  INFO TestListener:26 - ......TestListener requestInitialized()......
2016-01-14 00:03:04,001  INFO TestServlet:24 - ...TestServlet init() init..........
2016-01-14 00:03:04,011  INFO TestFilter:23 - ..............execute TestFilter doFilter()..............
2016-01-14 00:03:15,275  INFO TestServlet:42 - ...TestServlet doPost() start..........
2016-01-14 00:03:16,255  INFO TestListener:36 - ......TestListener attributeAdded()......
2016-01-14 00:03:16,853  INFO TestListener:44 - ......TestListener attributeReplaced()......
2016-01-14 00:03:18,561  INFO TestListener:40 - ......TestListener attributeRemoved()......
2016-01-14 00:03:20,065  INFO TestListener:16 - .......TestListener sessionCreated().......
2016-01-14 00:03:22,908  INFO TestListener:20 - .......TestListener sessionDestroyed().......
2016-01-14 00:03:25,624  INFO TestServlet:52 - ...TestServlet doPost() end..........
2016-01-14 00:03:27,746  INFO TestListener:30 - ......TestListener requestDestroyed()......

![image-20200318110905468](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318110905468.png)

第二次访问：

2016-01-14 00:04:08,908  INFO TestListener:26 - ......TestListener requestInitialized()......
2016-01-14 00:04:08,909  INFO TestFilter:23 - ..............execute TestFilter doFilter()..............
2016-01-14 00:04:14,385  INFO TestServlet:42 - ...TestServlet doPost() start..........
2016-01-14 00:04:14,778  INFO TestListener:36 - ......TestListener attributeAdded()......
2016-01-14 00:04:14,974  INFO TestListener:44 - ......TestListener attributeReplaced()......
2016-01-14 00:04:15,342  INFO TestListener:40 - ......TestListener attributeRemoved()......
2016-01-14 00:04:15,904  INFO TestListener:16 - .......TestListener sessionCreated().......
2016-01-14 00:04:17,354  INFO TestListener:20 - .......TestListener sessionDestroyed().......
2016-01-14 00:04:17,815  INFO TestServlet:52 - ...TestServlet doPost() end..........
2016-01-14 00:04:19,044  INFO TestListener:30 - ......TestListener requestDestroyed()......

![image-20200318110926921](/Users/wangchong/Library/Application Support/typora-user-images/image-20200318110926921.png)

关闭项目，打印日志如下：

2016-1-14 0:40:15 org.apache.coyote.http11.Http11Protocol pause
信息: Pausing Coyote HTTP/1.1 on http-80
2016-1-14 0:40:16 org.apache.catalina.core.StandardService stop
信息: Stopping service Catalina
2016-1-14 0:40:16 org.apache.catalina.core.ApplicationContext log
信息: SessionListener: contextDestroyed()
2016-1-14 0:40:16 org.apache.catalina.core.ApplicationContext log
信息: ContextListener: contextDestroyed()
2016-01-14 00:40:16,561  INFO TestServlet:30 - ...TestServlet init() destory..........
.......
2016-01-14 00:40:22,091  INFO TestFilter:19 - ..............execute TestFilter destroy()..............

********************************************

结论：
从启动，结束和运行时候的日志看：

Listener生命周期：一直从程序启动到程序停止运行。

ServletRequestListener：每次访问一个Request资源前，都会执行requestInitialized()方法，方法访问完毕，都会执行requestDestroyed()方法。

HttpSessionListener：每次调用request.getSession()，都会执行sessionCreated()方法，执行session.invalidate()方法，都会执行sessionDestroyed()方法。

ServletRequestAttributeListener：每次调用request.setAttribute()都会执行attributeAdded()方法，如果set的key在request里面存在，就会执行attributeReplacerd()方法，调用request.removeAttribute()方法，都会执行attributeRemoved()方法。



Filter生命周期：程序启动调用Filter的init()方法(永远只调用一次,具体看启动日志)，程序停止调用Filter的destroy()方法(永远只调用一次，具体看关闭日志)，doFilter()方法每次的访问请求如果符合拦截条件都会调用(程序第一次运行，会在servlet调用init()方法以后调用，不管第几次，都在调用doGet(),doPost()方法之前)。



Servlet生命周期：程序第一次访问，会调用servlet的init()方法初始化(只执行一次，具体看日志)，每次程序执行都会根据请求调用doGet()或者doPost()方法，程序停止调用destory()方法(具体看结束日志)。


