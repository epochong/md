---
title: 多个catch异常的捕获顺序
date: 2019-06-01 14:59:59
tags: java
---

##### try块中产生异常的时候

**按照 catch 块从上往下匹配执行**

当它匹配某一个 catch 块的时候，他就直接进入到这个 catch 块里面去了，后面在再有 catch 块的话，它不做任何处理，直接跳过去，全部忽略掉。如果有匹配的 catch，它就会忽略掉这个 catch 后面所有的 catch。如果有 finally 的话进入到 finally 里面继续执行。

##### 先看一下继承关系

- ```java
  - java.lang.Object  
  - - java.lang.Throwable  
    - - java.lang.Exception  
      - - java.io.IOException  
        - - java.io.FileNotFoundException  
  ```

#####  父类异常在子类异常前面

```java
public static void getCustomerInfo(){
    try {
        FileInputStream fileInputStream = new FileInputStream("F:\\Test\\test.txt");
    } catch (java.lang.Exception e) {
        System.out.println("Exception");
    } catch (java.io.FileNotFoundException e) {
        System.out.println("FileNotFoundException");
    } catch (java.io.IOException e) {
        System.out.println("IOException" + e);
    }
}
public static void main(String[] args) {
    getCustomerInfo();
}
```

一但抛出 `IOException`，它首先进入到 `catch (java.lang.Exception e) {}` 里面，先和 `Exception` 匹配，由于 `FileNotFoundException extends Exception`, 根据多态的原则，`FileNotFoundException` 是匹配 `Exception` 的，所以程序就会进入到 `catch (Exception e) {}` 里面，进入到第一个 `catch` 后，后面的 `catch` 都不会执行了，所以 catch `(FileNotFoundException e) {}` 永远都执行不到。

**就会产生以下运行时异常，先处理了父类异常就不需要在后面处理子类异常**

![Exception](http://psb7ug9p7.bkt.clouddn.com/exception.png)

##### 那么如果父类异常在子类异常后面又是如何呢

```java
public static void getCustomerInfo(){
    try {
        FileInputStream fileInputStream = new FileInputStream("F:\\Test\\test.txt");
    } catch (java.io.FileNotFoundException e) {
        System.out.println("FileNotFoundException");
    } catch (java.io.IOException e) {
        System.out.println("IOException" + e);
    }catch (java.lang.Exception e) {
        System.out.println("Exception");
    }
}

public static void main(String[] args) {
    getCustomerInfo();
}
```

![exception1](http://psb7ug9p7.bkt.clouddn.com/exception1.png)

从运行结果可以看出，捕获子类异常后，后面的catch块就不会执行了，如果产生的是`IOException`，则会达到第二个`catch`块不会执行第三个。

