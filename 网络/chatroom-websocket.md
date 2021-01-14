---
title: chatroom_websocket
date: 2019-07-27 10:51:22
tags: [project,JavaWeb,MySQL]
---

> 项目讲解，基于websocket实现的聊天室

<!--more-->

#### 业务

> 开发从dao层开始写

##### dao：数据库CURD

1. 获取数据源
2. 获取连接
3. sql语句
4. 关闭

- 各个数据操作只是sql语句不同，1,2,4为公共操作封装在父类BaseDao中
- 数据库操作在子类AccountDao中实现

##### service：处理业务

> 调用dao层

- 登录验证
- 注册验证

##### controller：

- 调用service获取数据返回给客户端
- 从客户端获得数据调用业务处理

#### JDBC（Java程序连接数据库标准）

##### JDBC:Java操作数据库的接口(OS)

所有数据库厂商必须实现此接口
Java DataBase Connector

##### JDBC操作格式

- 数据源

    > 类似线程池，创建好已连接的数据库直接拿来使用

    - druid（阿里数据源）
    - c3p0
    - Hakari

    ```properties
    driverClassName=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/chatroom_websocket?charset=utf8&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
    username=root
    password=root
    filters=stat
    initialSize=5//初始化数量
    maxActive=30//最大数量
    maxWait=60000//创建后等待时间
    timeBetweenEvictionRunsMillis=60000
    minEvictableIdleTimeMillis=300000
    validationQuery=SELECT 1
    testWhileIdle=true
    testOnBorrow=false
    testOnReturn=false
    poolPreparedStatements=false
    ```

    1. 加载驱动或加载数据源
    Class.forName("com.mysql.jdbc.Driver");

    2. 获取连接 Connection

    - **url：**jdbc:mysql://localhost:3306/db_name
    - **username**
    - **password**

    3. 使用连接执行SQL CURD

    - 执行sql Statement
    - 获取查询返回值 ResultSet

    ```java
    select : statement.executeQuery(sql) : ResultSet
    insert,delete,update
    statement.executeUpdate(sql) : int
    ```

    4. 关闭资源

    - Connection
    - Statement
    - ResultSet


##### 配置文件方式

- 将数据库的配置信息存放到配置文件(*.properties)
- 将信息从配置文件读到程序中 

##### Junit单元测试 : 白盒测试

- 创建测试类
    1. 创建测试文件夹,test与main同目录,将test文件夹标记为测试文件夹 
    2. 要测试的类选中，ctrl+shift+T 创建测试类
    3. 使用断言Assert

##### SQL注入漏洞

> `Statement`语句通过字符串拼接，黑客只需改变参数的值即可改变sql语句的正确性

- userName为正确值password拼接为 or 1 = 1成为一个永真式就可以获得这个用户的连接

    ```java
    connection = getConnection();
    String sql = "select * from user where username = ' " + username + " ' and" + "password = '" + password "'";
    statement = connection.createStatement();
    resultSet = statement.executeQuery(sql);
    ```

- 通过PreparedStatement用通配符替换字符串拼接可以检查语法错误

    ```java
    connection = getConnection();
    String sql = "select * from user where username = ? and" + "password = ?";
    statement = connection.prepareStatement(sql);
    statement.setString(1, username);
    statement.setString(2, DigestUtils.md5Hex(password));
    resultSet = statement.executeQuery();
    ```

#### WebSocket

##### 单边通信 - http协议

- 浏览器
- 服务端

##### 全双工通信（应用层协议） - tcp/ip

- websocket -（c/s） 

- 多线程聊天

> 服务端可以主动向浏览器发送信息

##### 协议

- **http协议：**http://域名:端口号

- **websocket：**//域名:端口号

    > 浏览器版本的socket

#### 模板引擎 - FreeMarker

> 获取后端传的参数，也可以用jsp，html，但是这个比较方便，要想用模板引擎后端必须要加载模板引擎

- 配置tomcat加载ftl页面路径
- 配置监听器，项目启动的时候监听器就执行了，所以可以将全局配置放在里面，各个Servlet都能用了

```java
package com.epochong.chatroom.config;

import freemarker.template.Configuration;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
import java.io.File;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * @author epochong
 * @date 2019/8/6 10:03
 * @email epochong@163.com
 * @blog epochong.github.io
 * @describe 预加载配置，相当于类的static静态块
 * WebListener：这个类具备监听器的能力
 */
@WebListener
public class FreeMarkerListener implements ServletContextListener {
    /**
     * 读取配置的时候通过k,v的方式
     * k：_template_
     * v：cfg （freemarker.template.Configuration）
     */
    public static final String TEMPLATE_KEY = "_template_";

    /**
     * 当项目启动的时候tomcat会自动调用
     * @param sce
     */
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        //配置版本
        Configuration cfg = new Configuration(Configuration.VERSION_2_3_0);
        //配置加载ftl路径
        try {
            cfg.setDirectoryForTemplateLoading(new File("F:\\Program\\Java\\Maven\\chatroom_websocket\\src\\main\\webapp"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 配置页面编码
        cfg.setDefaultEncoding(StandardCharsets.UTF_8.displayName());
        //将配置写入上下文中，设置k,v
        sce.getServletContext().setAttribute(TEMPLATE_KEY,cfg);
    }

    /**
     * 项目快要终止的时候调用
     * 可以放一些全局的资源释放
     * @param sce
     */
    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```

