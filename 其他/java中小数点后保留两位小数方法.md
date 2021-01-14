---
title: java中小数点后保留指定小数位数方法
date: 2019-07-01 14:53:42
tags: java
---

> 本文介绍java中几种保留小数点后指定小位数的方法,下面以保留两位为例

```java
package study;

import java.math.BigDecimal;
import java.text.DecimalFormat;
import java.text.NumberFormat;

public class Test {
    public static void main(String[] args) {
        double d = 123.456;

        //方法一：最简便的方法，调用DecimalFormat类
        DecimalFormat df = new DecimalFormat(".00");
        System.out.println(df.format(d));

        //方法二：直接通过String类的format函数实现
        System.out.println(String.format("%.2f", d));

        //方法三：通过BigDecimal类实现
        BigDecimal bg = new BigDecimal(d);
        double d3 = bg.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
        System.out.println(d3);

        //方法四：通过NumberFormat类实现
        NumberFormat nf = NumberFormat.getNumberInstance();
        nf.setMaximumFractionDigits(2);
        System.out.println(nf.format(d));

    }
}
```

