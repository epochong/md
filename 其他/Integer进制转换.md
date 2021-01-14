---
title: Integer进制转换
date: 2019-07-05 01:01:46
tags: java
---

- 十进制转成十六进制：   


```java
Integer.toHexString(int i)   
```

- 十进制转成八进制   


```java
Integer.toOctalString(int i)   
```

- 十进制转成二进制   


```java
Integer.toBinaryString(int i)   
```

- 十六进制转成十进制   


```java
Integer.valueOf("FFFF",16).toString()   
```

- 八进制转成十进制   


```java
Integer.valueOf("876",8).toString()   
```

- 二进制转十进制   


```java
Integer.valueOf("0101",2).toString()  
```

