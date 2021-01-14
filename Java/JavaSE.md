### Arrays.sort实现原理

- 元素个数 < 47 : 插入排序
- 47 <= 元素个数 < 286 : 快速排序
- 元素个数 > 286 ：归并排序

**1.对于小数组来说，插入排序效率更高，每次递归到小于47的大小时，用插入排序代替快排，明显提升了性能。
2.双轴快排使用两个pivot，每轮把数组分成3段，在没有明显增加比较次数的情况下巧妙地减少了递归次数。
3.pivot的选择上增加了随机性，却没有带来随机数的开销。
4.对重复数据进行了优化处理，避免了不必要交换和递归。**

#### 算法步骤

1.对于很小的数组（长度小于47），会使用插入排序。
2.选择两个点P1,P2作为轴心，比如我们可以使用第一个元素和最后一个元素。
3.P1必须比P2要小，否则将这两个元素交换，现在将整个数组分为四部分：
（1）第一部分：比P1小的元素。
（2）第二部分：比P1大但是比P2小的元素。
（3）第三部分：比P2大的元素。
（4）第四部分：尚未比较的部分。
在开始比较前，除了轴点，其余元素几乎都在第四部分，直到比较完之后第四部分没有元素。
4.从第四部分选出一个元素a[K]，与两个轴心比较，然后放到第一二三部分中的一个。
5.移动L，K，G指向。
6.重复 4 5 步，直到第四部分没有元素。
7.将P1与第一部分的最后一个元素交换。将P2与第三部分的第一个元素交换。
8.递归的将第一二三部分排序。

**双轴快排**：

- **快速排序**使用的是分治思想，将原问题分成若干个子问题进行递归解决。选择一个元素作为轴(pivot)，通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比轴元素小，另外一部分的所有数据都比轴元素大，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。
- **双轴快排**(DualPivotQuicksort)，顾名思义有**两个轴元素**pivot1，pivot2，且pivot ≤
    pivot2，**将序列分成三段**：x < pivot1、pivot1 ≤ x ≤ pivot2、x >pivot2，然后分别对三段进行递归。这个算法通常会比传统的快排效率更高，也因此被作为Arrays.java中给基本类型的数据排序的具体实现。

以JDK1.8中Arrays对int型数组的排序为例来介绍其中使用的双轴快排：

1.判断数组的长度是否大于286，大于则使用归并排序(merge sort)，否则执行2。

```java
 // Use Quicksort on small arrays
    if (right - left < QUICKSORT_THRESHOLD) {
            sort(a, left, right, true);
            return;
    }
    // Merge sort
```

2.判断数组长度是否小于47，小于则直接采用插入排序(insertion sort)，否则执行3。

```java
 // Use insertion sort on tiny arrays
    if (length < INSERTION_SORT_THRESHOLD) {
    // Insertion sort
    ......
    }
```

3.用公式length/8+length/64+1近似计算出数组长度的1/7。

```java
 // Inexpensive approximation of length / 7
    int seventh = (length >> 3) + (length >> 6) + 1;
```

4.取5个根据经验得出的等距点。

```java
 /*
     * Sort five evenly spaced elements around (and including) the
     * center element in the range. These elements will be used for
     * pivot selection as described below. The choice for spacing
     * these elements was empirically determined to work well on
     * a wide variety of inputs.
     */
    int e3 = (left + right) >>> 1; // The midpoint
    int e2 = e3 - seventh;
    int e1 = e2 - seventh;
    int e4 = e3 + seventh;
    int e5 = e4 + seventh;
```

5.将这5个元素进行插入排序

```java
// Sort these elements using insertion sort
    if (a[e2] < a[e1]) { int t = a[e2]; a[e2] = a[e1]; a[e1] = t; }
    if (a[e3] < a[e2]) { int t = a[e3]; a[e3] = a[e2]; a[e2] = t;
    if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
    }
    if (a[e4] < a[e3]) { int t = a[e4]; a[e4] = a[e3]; a[e3] = t;
        if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
            if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
        }
    }
    if (a[e5] < a[e4]) { int t = a[e5]; a[e5] = a[e4]; a[e4] = t;
        if (t < a[e3]) { a[e4] = a[e3]; a[e3] = t;
            if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
                if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
            }
        }
    }
```

6.选取a[e2]，a[e4]分别作为pivot1，pivot2。由于步骤5进行了排序，所以必有pivot1 <=pivot2。定义两个指针less和great，less从最左边开始向右遍历，一直找到第一个不小于pivot1的元素，great从右边开始向左遍历，一直找到第一个不大于pivot2的元素。

```java
 /*
         * Use the second and fourth of the five sorted elements as pivots.
         * These values are inexpensive approximations of the first and
         * second terciles of the array. Note that pivot1 <= pivot2.
         */
        int pivot1 = a[e2];
        int pivot2 = a[e4];
        /*
         * The first and the last elements to be sorted are moved to the
         * locations formerly occupied by the pivots. When partitioning
         * is complete, the pivots are swapped back into their final
         * positions, and excluded from subsequent sorting.
         */
        a[e2] = a[left];
        a[e4] = a[right];
        /*
         * Skip elements, which are less or greater than pivot values.
         */
        while (a[++less] < pivot1);
        while (a[--great] > pivot2);

```

7.接着定义指针k从less-1开始向右遍历至great，把小于pivot1的元素移动到less左边，大于pivot2的元素移动到great右边。这里要注意，我们已知great处的元素小于pivot2，但是它于pivot1的大小关系，还需要进行判断，如果比pivot1还小，需要移动到到less左边，否则只需要交换到k处。

```java
/*
 * Partitioning:
 *
 *   left part           center part                   right part
 * +--------------------------------------------------------------+
 * |  < pivot1  |  pivot1 <= && <= pivot2  |    ?    |  > pivot2  |
 * +--------------------------------------------------------------+
 *               ^                          ^       ^
 *               |                          |       |
 *              less                        k     great
 *
 * Invariants:
 *
 *              all in (left, less)   < pivot1
 *    pivot1 <= all in [less, k)     <= pivot2
 *              all in (great, right) > pivot2
 *
 * Pointer k is the first index of ?-part.
 */
        outer:
        for (int k = less - 1; ++k <= great; ) {
            int ak = a[k];
            if (ak < pivot1) { // Move a[k] to left part
                a[k] = a[less];
                /*
                 * Here and below we use "a[i] = b; i++;" instead
                 * of "a[i++] = b;" due to performance issue.
                 */
                a[less] = ak;
                ++less;
            } else if (ak > pivot2) { // Move a[k] to right part
                while (a[great] > pivot2) {
                    if (great-- == k) {
                        break outer;
                    }
                }
                if (a[great] < pivot1) { // a[great] <= pivot2
                    a[k] = a[less];
                    a[less] = a[great];
                    ++less;
                } else { // pivot1 <= a[great] <= pivot2
                    a[k] = a[great];
                }
                /*
                 * Here and below we use "a[i] = b; i--;" instead
                 * of "a[i--] = b;" due to performance issue.
                 */
                a[great] = ak;
                --great;
            }
        }

```

8.将less-1处的元素移动到队头，great+1处的元素移动到队尾，并把pivot1和pivot2分别放到less-1和great+1处。

```java
// Swap pivots into their final positions
        a[left]  = a[less  - 1]; a[less  - 1] = pivot1;
        a[right] = a[great + 1]; a[great + 1] = pivot2;

```

9.至此，less左边的元素都小于pivot1，great右边的元素都大于pivot2，分别对两部分进行同样的递归排序。

```java
// Sort left and right parts recursively, excluding known pivots
        sort(a, left, less - 2, leftmost);
        sort(a, great + 2, right, false);

```

10.对于中间的部分，如果大于4/7的数组长度，很可能是因为重复元素的存在，所以把less向右移动到第一个不等于pivot1的地方，把great向左移动到第一个不等于pivot2的地方，然后再对less和great之间的部分进行递归排序。

```java
/*
         * If center part is too large (comprises > 4/7 of the array),
         * swap internal pivot values to ends.
         */
        if (less < e1 && e5 < great) {
            /*
             * Skip elements, which are equal to pivot values.
             */
            while (a[less] == pivot1) {
                ++less;
            }
            while (a[great] == pivot2) {
                --great;
            }
        }
        ......
        // Sort center part recursively
        sort(a, less, great, false);
```

#### 具体细节

事实上Collections.sort方法底层就是调用的array.sort方法，而且不论是Collections.sort或者是Arrays.sort方法，

```java
public static void main(String[] args) {
                 List<String> strings = Arrays.asList("6", "1", "3", "1","2");             
             Collections.sort(strings);//sort方法在这里

             for (String string : strings) {

                 System.out.println(string);
             }
         }
```
collections.sort方法调用的list.sort


list里面有个sort方法，但是list是一个接口，肯定是调用子类里面的实现，这里我们demo使用的是一个Arrays.asList方法，所以事实上我们的子类就是arraylist了。OK，看arraylist里面sort实现，选择第一个，为什么不选择第二个呢？（可以看二楼评论，解答得很正确，简单说就是用Arrays.sort创建的ArrayList对象）


发现里面调用的Arrays.sort(a, c); a是list,c是一个比较器，我们来看一下这个方法

我们没有写比较器，所以用的第二项，LegacyMergeSort.userRequested这个bool值是什么呢？
跟踪这个值，我们发现有这样的一段定义：

  > Old merge sort implementation can be selected (for
  > compatibility with broken comparators) using a system property.
  > Cannot be a static boolean in the enclosing class due to
  > circular dependencies. To be removed in a future release.

  反正是一种老的归并排序，不用管了现在默认是关的
我们走的是sort(a)这个方法，接着进入这个

接着看我们重要的sort方法

```java
static void sort(Object[] a, int lo, int hi, Object[] work, int workBase, int workLen) {
                 assert a != null && lo >= 0 && lo <= hi && hi <= a.length;           
			int nRemaining  = hi - lo;
             if (nRemaining < 2)
                 return;  // array的大小为0或者1就不用排了

             // 当数组大小小于MIN_MERGE(32)的时候，就用一个"mini-TimSort"的方法排序，jdk1.7新加
             if (nRemaining < MIN_MERGE) {
                //这个方法比较有意思，其实就是将我们最长的递减序列，找出来，然后倒过来
                 int initRunLen = countRunAndMakeAscending(a, lo, hi);
                 //长度小于32的时候，是使用binarySort的
                 binarySort(a, lo, hi, lo + initRunLen);
                 return;
             }
            //先扫描一次array，找到已经排好的序列，然后再用刚才的mini-TimSort，然后合并，这就是TimSort的核心思想
             ComparableTimSort ts = new ComparableTimSort(a, work, workBase, workLen);
             int minRun = minRunLength(nRemaining);
             do {
                 // Identify next run
                 int runLen = countRunAndMakeAscending(a, lo, hi);

                 // If run is short, extend to min(minRun, nRemaining)
                 if (runLen < minRun) {
                     int force = nRemaining <= minRun ? nRemaining : minRun;
                     binarySort(a, lo, lo + force, lo + runLen);
                     runLen = force;
                 }

                 // Push run onto pending-run stack, and maybe merge
                 ts.pushRun(lo, runLen);
                 ts.mergeCollapse();

                 // Advance to find next run
                 lo += runLen;
                 nRemaining -= runLen;
             } while (nRemaining != 0);

             // Merge all remaining runs to complete sort
             assert lo == hi;
             ts.mergeForceCollapse();
             assert ts.stackSize == 1;
     }
```
回到5，我们可以看到当我们写了比较器的时候就调用了TimSort.sort方法，源码如下

```java
static <T> void sort(T[] a, int lo, int hi, Comparator<? super T> c,
                                  T[] work, int workBase, int workLen) {
                 assert c != null && a != null && lo >= 0 && lo <= hi && hi <= a.length;             
						 int nRemaining  = hi - lo;
             if (nRemaining < 2)
                 return;  // Arrays of size 0 and 1 are always sorted

             // If array is small, do a "mini-TimSort" with no merges
             if (nRemaining < MIN_MERGE) {
                 int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
                 binarySort(a, lo, hi, lo + initRunLen, c);
                 return;
             }

             /**
              * March over the array once, left to right, finding natural runs,
              * extending short natural runs to minRun elements, and merging runs
              * to maintain stack invariant.
              */
       TimSort<T> ts = new TimSort<>(a, c, work, workBase, workLen);
            int minRun = minRunLength(nRemaining);
            do {
                // Identify next run
                int runLen = countRunAndMakeAscending(a, lo, hi, c);

                // If run is short, extend to min(minRun, nRemaining)
                if (runLen < minRun) {
                    int force = nRemaining <= minRun ? nRemaining : minRun;
                    binarySort(a, lo, lo + force, lo + runLen, c);
                    runLen = force;
                }

                // Push run onto pending-run stack, and maybe merge
                ts.pushRun(lo, runLen);
                ts.mergeCollapse();

                // Advance to find next run
                lo += runLen;
                nRemaining -= runLen;
            } while (nRemaining != 0);

            // Merge all remaining runs to complete sort
            assert lo == hi;
            ts.mergeForceCollapse();
            assert ts.stackSize == 1;
  }
```
和上面的sort方法是一样的，其实也就是TimSort的源代码

3、总结
不论是Collections.sort方法或者是Arrays.sort方法，底层实现都是TimSort实现的，这是jdk1.7新增的，以前是归并排序。TimSort算法就是找到已经排好序数据的子序列，然后对剩余部分排序，然后合并起来 JAVA基础

## 面向对象和面向过程的区别

- **面向过程** ：**面向过程性能比面向对象高。** 因为类调用时需要实例化，开销比较大，比较消耗资源，所以当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发。但是，**面向过程没有面向对象易维护、易复用、易扩展。**
- **面向对象** ：**面向对象易维护、易复用、易扩展。** 因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，**面向对象性能比面向过程低**。

参见 issue :  面向过程 ：面向过程性能比面向对象高？？

> 这个并不是根本原因，面向过程也需要分配内存，计算内存偏移量，Java性能差的主要原因并不是因为它是面向对象语言，而是Java是半编译语言，最终的执行代码并不是可以直接被CPU执行的二进制机械码。
>
> 而面向过程语言大多都是直接编译成机械码在电脑上执行，并且其它一些面向过程的脚本语言性能也并不一定比Java好。

## Java 语言有哪些特点?

1. 简单易学；
2. 面向对象（封装，继承，多态）；
3. 平台无关性（ Java 虚拟机实现平台无关性）；
4. 可靠性；
5. 安全性；
6. 支持多线程（ C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持）；
7. 支持网络编程并且很方便（ Java 语言诞生本身就是为简化网络编程设计的，因此 Java 语言不仅支持网络编程而且很方便）；
8. 编译与解释并存；

## 关于 JVM JDK 和 JRE 最详细通俗的解答

### JVM

Java虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。

**什么是字节码?采用字节码的好处是什么?**

> 在 Java 中，JVM可以理解的代码就叫做`字节码`（即扩展名为 `.class` 的文件），它不面向任何特定的处理器，只面向虚拟机。Java 语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以 Java 程序运行时比较高效，而且，由于字节码并不针对一种特定的机器，因此，Java程序无须重新编译便可在多种不同操作系统的计算机上运行。

**Java 程序从源代码到运行一般有下面3步：**

![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87H9SpUp8QZPBanQId5y5P37u77zyuSUNkGQbzkA2mibqvDnP889Ejm78w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Java程序运行过程

我们需要格外注意的是 .class->机器码 这一步。在这一步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的(也就是所谓的热点代码)，所以后面引进了 JIT 编译器，而JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们为什么经常会说 Java 是编译与解释共存的语言。

> HotSpot采用了惰性评估(Lazy Evaluation)的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码（热点代码），而这也就是JIT所需要编译的部分。JVM会根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度就越快。JDK 9引入了一种新的编译模式AOT(Ahead of Time Compilation)，它是直接将字节码编译成机器码，这样就避免了JIT预热等各方面的开销。JDK支持分层编译和AOT协作使用。但是 ，AOT 编译器的编译质量是肯定比不上 JIT 编译器的。

**总结：**

Java虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM  实现是 Java 语言“一次编译，随处可以运行”的关键所在。

### JDK 和 JRE

JDK是Java Development Kit，它是功能齐全的Java SDK。它拥有JRE所拥有的一切，还有编译器（javac）和工具（如javadoc和jdb）。它能够创建和编译程序。

JRE 是 Java运行时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java虚拟机（JVM），Java类库，java命令和其他的一些基础构件。但是，它不能用于创建新程序。

如果你只是为了运行一下 Java 程序的话，那么你只需要安装 JRE 就可以了。如果你需要进行一些 Java 编程方面的工作，那么你就需要安装JDK了。但是，这不是绝对的。有时，即使您不打算在计算机上进行任何Java开发，仍然需要安装JDK。例如，如果要使用JSP部署Web应用程序，那么从技术上讲，您只是在应用程序服务器中运行Java程序。那你为什么需要JDK呢？因为应用程序服务器会将 JSP 转换为 Java servlet，并且需要使用 JDK 来编译 servlet。

## Oracle JDK 和 OpenJDK 的对比

可能在看这个问题之前很多人和我一样并没有接触和使用过  OpenJDK 。那么Oracle和OpenJDK之间是否存在重大差异？下面我通过收集到的一些资料，为你解答这个被很多人忽视的问题。

对于Java 7，没什么关键的地方。OpenJDK项目主要基于Sun捐赠的HotSpot源代码。此外，OpenJDK被选为Java 7的参考实现，由Oracle工程师维护。关于JVM，JDK，JRE和OpenJDK之间的区别，Oracle博客帖子在2012年有一个更详细的答案：

> 问：OpenJDK存储库中的源代码与用于构建Oracle JDK的代码之间有什么区别？
>
> 答：非常接近 - 我们的Oracle JDK版本构建过程基于OpenJDK 7构建，只添加了几个部分，例如部署代码，其中包括Oracle的Java插件和Java WebStart的实现，以及一些封闭的源代码派对组件，如图形光栅化器，一些开源的第三方组件，如Rhino，以及一些零碎的东西，如附加文档或第三方字体。展望未来，我们的目的是开源Oracle JDK的所有部分，除了我们考虑商业功能的部分。

**总结：**

1. Oracle JDK大概每6个月发一次主要版本，而OpenJDK版本大概每三个月发布一次。但这不是固定的，我觉得了解这个没啥用处。详情参见：https://blogs.oracle.com/java-platform-group/update-and-faq-on-the-java-se-release-cadence。
2. OpenJDK 是一个参考模型并且是完全开源的，而Oracle JDK是OpenJDK的一个实现，并不是完全开源的；
3. Oracle JDK 比 OpenJDK 更稳定。OpenJDK和Oracle JDK的代码几乎相同，但Oracle JDK有更多的类和一些错误修复。因此，如果您想开发企业/商业软件，我建议您选择Oracle JDK，因为它经过了彻底的测试和稳定。某些情况下，有些人提到在使用OpenJDK 可能会遇到了许多应用程序崩溃的问题，但是，只需切换到Oracle JDK就可以解决问题；
4. 在响应性和JVM性能方面，Oracle JDK与OpenJDK相比提供了更好的性能；
5. Oracle JDK不会为即将发布的版本提供长期支持，用户每次都必须通过更新到最新版本获得支持来获取最新版本；
6. Oracle JDK根据二进制代码许可协议获得许可，而OpenJDK根据GPL v2许可获得许可。

## Java和C++的区别?

我知道很多人没学过 C++，但是面试官就是没事喜欢拿咱们 Java 和 C++ 比呀！没办法！！！就算没学过C++，也要记下来！

- 都是面向对象的语言，都支持封装、继承和多态
- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
- Java 有自动内存管理机制，不需要程序员手动释放无用内存

## 什么是 Java 程序的主类 应用程序和小程序的主类有何不同?

一个程序中可以有多个类，但只能有一个类是主类。在 Java 应用程序中，这个主类是指包含 main（）方法的类。而在 Java 小程序中，这个主类是一个继承自系统类 JApplet 或 Applet 的子类。应用程序的主类不一定要求是 public 类，但小程序的主类要求必须是 public 类。主类是 Java 程序执行的入口点。

## Java 应用程序与小程序之间有哪些差别?

简单说应用程序是从主线程启动(也就是 `main()` 方法)。applet 小程序没有 `main()` 方法，主要是嵌在浏览器页面上运行(调用`init()`或者`run()`来启动)，嵌入浏览器这点跟 flash 的小游戏类似。

## 字符型常量和字符串常量的区别?

1. 形式上: 字符常量是单引号引起的一个字符; 字符串常量是双引号引起的若干个字符
2. 含义上: 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)
3. 占内存大小 字符常量只占2个字节; 字符串常量占若干个字节(至少一个字符结束标志) (**注意：char在Java中占两个字节**)

> java编程思想第四版：2.2.2节![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HL9POicmsX4w2kicHVoHuFO449fibPIw1mxNJulN1U1mCjB0pRo8NqaUog/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 构造器 Constructor 是否可被 override?

在讲继承的时候我们就知道父类的私有属性和构造方法并不能被继承，所以 Constructor 也就不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

## 重载和重写的区别

- **重载：** 发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同，发生在编译时。
- **重写：**  发生在父子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类；如果父类方法访问修饰符为 private 则子类就不能重写该方法。

## Java 面向对象编程三大特性: 封装 继承 多态

### 封装

封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。

### 继承

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码。

**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

### 多态

所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

在Java中有两种形式可以实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。

## String StringBuffer 和 StringBuilder 的区别是什么? String 为什么是不可变的?

**可变性**

简单的来说：String 类中使用 final 关键字修饰字符数组来保存字符串，`private　final　char　value[]`，所以 String 对象是不可变的。而StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串`char[]value` 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

StringBuilder 与 StringBuffer 的构造方法都是调用父类构造方法也就是 AbstractStringBuilder 实现的，大家可以自行查阅源码。

AbstractStringBuilder.java

```
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    AbstractStringBuilder() {
    }
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
```

**线程安全性**

String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。

**性能**

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据: 适用String
2. 单线程操作字符串缓冲区下操作大量数据: 适用StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据: 适用StringBuffer

## 自动装箱与拆箱

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

## 在一个静态方法内调用一个非静态成员为什么是非法的?

由于静态方法可以不通过对象进行调用，因此在静态方法里，不能调用其他非静态变量，也不可以访问非静态变量成员。

## 在 Java 中定义一个不做事且没有参数的构造方法的作用

Java 程序在执行子类的构造方法之前，如果没有用 `super()`来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 `super()`来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。

## import java和javax有什么区别？

刚开始的时候 JavaAPI 所必需的包是 java 开头的包，javax 当时只是扩展 API 包来使用。然而随着时间的推移，javax 逐渐地扩展成为 Java API 的组成部分。但是，将扩展从 javax 包移动到 java 包确实太麻烦了，最终会破坏一堆现有的代码。因此，最终决定 javax 包将成为标准API的一部分。

所以，实际上java和javax没有区别。这都是一个名字。

## 接口和抽象类的区别是什么？

1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
2. 接口中除了static、final变量，不能有其他变量，而抽象类中则不一定。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。
4. 接口方法默认修饰符是public，抽象方法可以有public、protected和default这些修饰符（抽象方法就是为了被重写所以不能使用private关键字修饰！）。
5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。

备注：在JDK8中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现两个接口，接口中定义了一样的默认方法，则必须重写，不然会报错。(详见issue:https://github.com/Snailclimb/JavaGuide/issues/146)

## 成员变量与局部变量的区别有哪些？

1. 从语法形式上看:成员变量是属于类的，而局部变量是在方法中定义的变量或是方法的参数；成员变量可以被 public,private,static 等修饰符所修饰，而局部变量不能被访问控制修饰符及 static 所修饰；但是，成员变量和局部变量都能被 final 所修饰。
2. 从变量在内存中的存储方式来看:如果成员变量是使用`static`修饰的，那么这个成员变量是属于类的，如果没有使用`static`修饰，这个成员变量是属于实例的。而对象存在于堆内存，局部变量则存在于栈内存。
3. 从变量在内存中的生存时间上看:成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。
4. 成员变量如果没有被赋初值:则会自动以类型的默认值而赋值（一种情况例外:被 final 修饰的成员变量也必须显式地赋值），而局部变量则不会自动赋值。

## 创建一个对象用什么运算符?对象实体与对象引用有何不同?

new运算符，new创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。一个对象引用可以指向0个或1个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有n个引用指向它（可以用n条绳子系住一个气球）。

## 什么是方法的返回值?返回值在类的方法里的作用是什么?

方法的返回值是指我们获取到的某个方法体中的代码执行后产生的结果！（前提是该方法可能产生结果）。返回值的作用:接收出结果，使得它可以用于其他的操作！

## 一个类的构造方法的作用是什么? 若一个类没有声明构造方法，该程序能正确执行吗? 为什么?

主要作用是完成对类对象的初始化工作。可以执行。因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。

## 构造方法有哪些特性？

1. 名字与类名相同。
2. 没有返回值，但不能用void声明构造函数。
3. 生成类的对象时自动执行，无需调用。

## 静态方法和实例方法有何不同

1. 在外部调用静态方法时，可以使用"类名.方法名"的方式，也可以使用"对象名.方法名"的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。
2. 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制。

## 对象的相等与指向他们的引用相等,两者有什么不同?

对象的相等，比的是内存中存放的内容是否相等。而引用相等，比较的是他们指向的内存地址是否相等。

## 在调用子类构造方法之前会先调用父类没有参数的构造方法,其目的是?

帮助子类做初始化工作。

## == 与 equals(重要)

**==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

**举个例子：**

```
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        String aa = "ab"; // 放在常量池中
        String bb = "ab"; // 从常量池中查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，非同一对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

**说明：**

- String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
- 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。

## hashCode 与 equals (重要)

面试官可能会问你：“你重写过 hashcode 和 equals 么，为什么重写equals时必须重写hashCode方法？”

### hashCode（）介绍

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

### 为什么要有 hashCode

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals（）`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的Java启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

通过我们可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个int整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**`hashCode()`在散列表中才有用，在其它情况下没用。**在散列表中hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

### hashCode（）与equals（）的相关规定

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个对象分别调用equals方法都返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

推荐阅读：Java hashCode() 和 equals()的若干问题解答

## 为什么Java中只有值传递？

为什么Java中只有值传递？

## 简述线程、程序、进程的基本概念。以及他们之间关系是什么?

**线程**与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

**程序**是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就是说程序是静态的代码。

**进程**是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如CPU时间，内存空间，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系统载入内存中。线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

##  线程有哪些基本状态?

Java 线程在运行的生命周期中的指定时刻只可能处于下面6种不同状态的其中一个状态（图源《Java 并发编程艺术》4.1.4节）。

![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HZJera9R54IyeVL6HNRoa3EV9B6plxKjZkXHeKewkYsQ4FZKOQDOWJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Java线程的状态

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示（图源《Java 并发编程艺术》4.1.4节）：

![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HBYJpCx3CzMYFYnibMNsEic9D8GRuIvUDGswN7l3oc4icZpUUtDicaBRYicg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Java线程状态变迁

由上图可以看出：

线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 cpu 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。

> 操作系统隐藏 Java虚拟机（JVM）中的 READY 和 RUNNING 状态，它只能看到 RUNNABLE 状态（图源：HowToDoInJava：Java Thread Life Cycle and Thread States），所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。

![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HtfibYicrB7mPnjoNq2e6dwZ9EE69gic6hqS1kribWdEpgsNSBDNefM5cBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)RUNNABLE-VS-RUNNING

当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）**状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIME_WAITING(超时等待)** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到**TERMINATED（终止）** 状态。

## 关于 final 关键字的一些总结

final关键字主要用在三个地方：变量、方法、类。

1. 对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
2. 当用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。
3. 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

## Java 中的异常处理

### Java异常类层次结构图

![img](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HLn1PzU27cPnrlR1EpZibcd18tNRjc0QQ6BsLbyXAEnzMibgQZpQ0jCicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Java异常类层次结构图

在 Java 中，所有的异常都有一个共同的祖先java.lang包中的 **Throwable类**。Throwable：有两个重要的子类：**Exception（异常）** 和 **Error（错误）** ，二者都是 Java 异常处理的重要子类，各自都包含大量子类。

**Error（错误）:是程序无法处理的错误**，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。

这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。

**Exception（异常）:是程序本身可以处理的异常**。Exception 类有一个重要的子类**RuntimeException**。RuntimeException 异常由Java虚拟机抛出。**NullPointerException**（要访问的变量没有引用任何对象时，抛出该异常）、**ArithmeticException**（算术运算异常，一个整数除以0时，抛出该异常）和 **ArrayIndexOutOfBoundsException** （下标越界异常）。

**注意：异常和错误的区别：异常能被程序本身处理，错误是无法处理。**

### Throwable类常用方法

- **public string getMessage()**:返回异常发生时的简要描述
- **public string toString()**:返回异常发生时的详细信息
- **public string getLocalizedMessage()**:返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以声称本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与getMessage（）返回的结果相同
- **public void printStackTrace()**:在控制台上打印Throwable对象封装的异常信息

### 异常处理总结

- **try 块：** 用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。
- **catch 块：** 用于处理try捕获到的异常。
- **finally 块：** 无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return 语句时，finally语句块将在方法返回之前被执行。

**在以下4种特殊情况下，finally块不会被执行：**

1. 在finally语句块第一行发生了异常。因为在其他行，finally块还是会得到执行
2. 在前面的代码中用了System.exit(int)已退出程序。exit是带参函数 ；若该语句在异常语句之后，finally会执行
3. 程序所在的线程死亡。
4. 关闭CPU。

下面这部分内容来自issue:https://github.com/Snailclimb/JavaGuide/issues/190。

**注意：** 当try语句和finally语句中都有return语句时，在方法返回之前，finally语句的内容将被执行，并且finally语句的返回值将会覆盖原始的返回值。如下：

```
    public static int f(int value) {
        try {
            return value * value;
        } finally {
            if (value == 2) {
                return 0;
            }
        }
    }
```

如果调用 `f(2)`，返回值将是0，因为finally语句的返回值覆盖了try语句块的返回值。

## Java序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用transient关键字修饰。

transient关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

## 获取用键盘输入常用的两种方法

方法1：通过 Scanner

```
Scanner input = new Scanner(System.in);
String s  = input.nextLine();
input.close();
```

方法2：通过 BufferedReader

```
BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```

## Java 中 IO 流

### Java 中 IO 流分为几种?

- 按照流的流向分，可以分为输入流和输出流；
- 按照操作单元划分，可以划分为字节流和字符流；
- 按照流的角色划分为节点流和处理流。

Java Io流共涉及40多个类，这些类看上去很杂乱，但实际上很有规则，而且彼此之间存在非常紧密的联系， Java I0流的40多个类都是从如下4个抽象类基类中派生出来的。

- InputStream/Reader: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

按操作方式分类结构图：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87HpdHWTnJyEkboh6l8IXoXcQ8EkKeIpdmquXtticfeXlibste3ticGAplbg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)IO-操作方式分类

按操作对象分类结构图：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/iaIdQfEric9Twb1OteH4T5vjlibvNnnr87H2kBaRhA9qR0hdjX9JXb26DU81RfsrQjXL5wccXzGmlia7f3cVEuNYSQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)IO-操作对象分类

### 既然有了字节流,为什么还要有字符流?

问题本质想问：**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

回答：字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

### BIO,NIO,AIO 有什么区别?

- **BIO (Blocking I/O):** 同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。
- **NIO (New I/O):** NIO是一种同步非阻塞的I/O模型，在Java 1.4 中引入了NIO框架，对应 java.nio 包，提供了 Channel , Selector，Buffer等抽象。NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。NIO提供了与传统BIO模型中的 `Socket` 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发
- **AIO (Asynchronous I/O):** AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。

## 关键字总结:static,final,this,super

详见笔主的这篇文章: https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/Basis/final、static、this、super.md

## Collections 、Arrays 方法总结

详见笔主的这篇文章: https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/Basis/Arrays,CollectionsCommonMethods.md

## 反射

**框架：**半成品软件。可以在框架的基础上进行软件开发，简化编码

**反射：**将类的各个组成部分封装为其他对象，这就是发射机制。

#### Java代码在计算机中经历的三个阶段

- **Source源代码阶段：**`javac`编译`.java`文件成`.class`字节码文件
    - 属性
    - 构造方法
    - 方法

- Class类阶段
    - 成员变量：Filed[]  fields
    - 构造方法：Constructor[]  cons
    - 成员方法：Method[]  methods
- Runtime运行阶段

#### 反射好处

> 编译器的对象方法代码提示的实现

1. 可以在程序运行中，操作类的对象
2. 可以解耦，提高程序的可扩展性

### Java中的反射

- "反"的操作核心的处理就在于Object类的一个方法:，取得Class对象: 

    ```java
    public final native Class<?> getClass();
    ```

- java中使用Class类描述java中的类的信息

    - ==Class类描述接口与类组成，Class对象由JVM在第一次加载类时产生，并且全局唯一==

- 三种获得Class对象的方法

    1. 任何类的实例化对象可以通过Object类中的getClass()方法取得Class类对象。 

    2. "类.class":直接根据某个具体的类来取得Class类的实例化对象。 

    3. 使用Class类提供的方法:public static Class<?> forName(String className) throws  ClassNotFoundException 

        ```java
        public class Test {
        	public static void main(String[] args) throws ClassNotFoundException{ 		Class<?> cls = Class.forName("java.util.Date") ; 					  System.out.println(cls.getName()); 
        	} 
        }
        ```

    - ==基本类型：int.class-->int，包装类型：Interge.class--->Class java.util.Integer==
    - ==多线程模式下，synchronized(Test.class)全局锁，因为Class对象唯一，所以可以锁==

#### Class类方法解释

> 利用反射可以做出一个对象具备的所有操作行为，最为关键的是这一切的操作都可以基于Object进行。

##### 实例化类对象方法 -   `newInstance()`

```java
public T newInstance() throws InstantiationException, IllegalAccessException
```

- 通过该方法实例化对象

    ```java
    //?可用Date替换
    Class<?> cls = Class.forName("java.util.Date") ; 
    Object obj = cls.newInstance() ;
    // 实例化对象，等价于 new java.util.Date() ;
    ```

    - ==通过反射可以破坏类的封装性，实例化私有构造方法==

- java实例化类对象的方式

    1. 直接new
    2. 通过工厂模式
    3. 通过反射

    - ==通过反射可以解决传统的工厂模式带来的增加产品要修改工厂的弊端==

##### 取得父类信息的方法
- 取得类的包名称

     - 方法声明

         ```java
         public Package getPackage()
         //Package是一个类，用来描述包信息的类
         ```

     - 使用方法

         ```java
         Class<?> cls = Date.class ;
         //不加getName()前面显示类型，package
         System.out.println(cls.getPackage().getName());//只显示包名称
         ```

- 取得父类Class对象

    	- 方法声明
     
        	```java
        // 本地方法
        public native Class<? super T> getSuperclass();
        ```
        
    - ==java 的一个设计理念是，与泛型相关的异常最好是在编译期间就被发现，因此设计了 extends 与 super 这两种方式==
      
        - **List<? extends T>** 表示该集合中存在的都是类型 T 的子类，包括 T 自己。
        - **List<? super T>** 表示该集合中存的都是类型 T 的父类，包括 T 自己。
    
- 取得父类实现的接口们

    - 方法声明
    
    	```java
    	public Class<?>[] getInterfaces()
    	```
    
    - 方法应用
    
        ```java
        // 取得CLS类的Class类对象
        Class<?> cls = CLS.class ; 
        // 取得实现的父接口对象 
        Class<?>[] iClass = cls.getInterfaces() ; 
        for (Class<?> class1 : iClass) { 
        	System.out.println(class1.getName()); 
        }
        ```
    

#### 获取构造方法

- 取得指定参数类型的public构造

    ```java
    public Constructor<T> getConstructor(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException
    ```

- 取得类中的所有public构造

    ```java
    public Constructor<?>[] getConstructors() throws SecurityException
    ```

    - 只能取得类中所有public的权限的
    - 本类，不可为父类，与继承无关

- 取得类中的所有构造方法（不限权限）

    ```java
    public Constructor<?>[]getDeclaredConstructors()throwsSecurityException
    ```

##### 关于返回值Constructor类

- **重要方法，**根据参数实例化对象的方法

    ```java
    public T newInstance(Object ... initargs)
    ```

    - 方法应用

        ```java
        Class<?> cls = Person.class ; // 取得指定参数类型的构造方法对象 
        Constructor<?> cons = cls.getConstructor(String.class,int.class) ; System.out.println(cons.newInstance("epochong",29));
        ```

- **getName()：**返回包名.类名

- **toSting()：**才是真正的返回构造方法的信息：权限、参数列表

==Class类通过反射实例化类对象（cls.newInstance()）的时候，只能够调用类中的无参构造。如果现在类中没有无参构造则无法使用Class类调用，只能够通过Constructor对象，明确的构造调用实例化处理。==

#### 获取普通方法

- 取得全部普通方法

    ```java
    public Method[] getMethods() throws SecurityException
    ```

- 取得指定的普通方法

    ```java
    //方法名称，这个方法的参数类型的class对象，按照参数顺序，依次填写对应的class对象
    //如果该方法没有参数，只填写方法的名称即可
    public Method getMethod(String name, Class<?>... parameterTypes)
    ```

##### 关于两个方法实现机制

以上两个方法返回的类型是`java.lang.reflflect.Method`类的对象，在此类中提供有一个调用方法的支持:

- 调用

    ```java
    public Object invoke(Object obj, Object... args)throws IllegalAccessException,
    IllegalArgumentException,InvocationTargetException
    ```
    
- 应用

    ```java
    // 任何时候调用类中的普通方法都必须有实例化对象 
    Class<?> cls = Class.forName("www.epochong.Person") ; 
    // 取得setName这个方法的实例化对象,设置方法名称与参数类型 
    Object obj = cls.newInstance() ; 
    Method setMethod = cls.getMethod("setName", String.class) ; 
    // 随后需要通过Method类对象调用指定的方法，调用方法需要有实例化对象,同时传入参数 
    
    // 相当于Person对象.setName("epochong") ; 
    setMethod.invoke(obj, "epochong") ; 
    Method getMethod = cls.getMethod("getName") ; 
    // 相当于Person对象.getName() ; 
    Object result = getMethod.invoke(obj) ; 
    System.out.println(result) ;
    ```

    ==此类操作的好处是:不再局限于某一具体类型的对象，而是可以通过Object类型进行所有类的方法调用。==

#### 反射调用类中的属性


> 不带Declared只能访问public的变量，带Declared不考虑修饰符，但是在访问私有属性的时候要通过暴力反射`setAccessible(true)`的方式获取才能操作

- 获取成员变量和设置值

    - `File[]  getFields()`：获取该类和父类所有public修饰的变量

    - `Field[] getField(String name)`：获取该类指定名称的public变量

        ```java
        Class personClass = Person.class;
        Field a = personClass.getField("a");
        Person p = new Person();
        //设置属性值
        a.set(p,"epochong");
        //获取属性值
        Object value = a.get(p);
        ```

    - `Field[] getDeclaredFields()`：不考虑修饰符，获取该类所有属性

    - `Field[] getDeclaredField(String name)`：不考虑修饰符，获取指定属性

        - 忽略访问权限修饰符的安全检查（暴力反射）

            ```java
            Field d = personClass.getDeclaredField("d");
            d.setAccessible(true);
            ```

    
    - 应用
    
        ```java
        class Person { 
        	private String name ; 
        }
        public class Test { 
        	public static void main(String[] args) throws Exception {
        		Class<?> cls=Class.forName("com.epochong.Person") ; // 实例化本类对象 
        		Object obj = cls.newInstance() ; // 操作name属性 
        		Field nameField = cls.getDeclaredField("name") ; 					nameField.set(obj, "epochong") ; // 相当于对象.name = "epochong" 		 
                System.out.println(nameField.get(obj)); // 取得属性 
        		} 
        }
        ```
    
    - Field类中的一个有用的方法
    
        ```java
        public Class<?> getType()
        ```
    
        - 应用
    
            ```java
            public class Person {
                private Integer id;
                protected String name ;
                public int age ;
            }
            ```
    
            ```java
            public class Test1 {
                public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchFieldException {
                    Class<?> cls = Class.forName("com.epochong.Person");
                    Object obj = cls.newInstance();
                    Field idField = cls.getDeclaredField("id");
                    System.out.println(idField.getType());
                    System.out.println(idField.getType().getName());
                    System.out.println(idField.getType().getSimpleName());
            
                }
            }
            ```
    
        - 输出结果
    
            ```java
            class java.lang.Integer
            java.lang.Integer
            Integer
            ```
    
        - 如果是基本类型结果为
    
            ```java
            int
            int
            int
            ```

##### 动态设置封装（暴力反射）

> 获得该类的私有属性，并对其赋值

- 方法

    ```java
    public void setAccessible(boolean flag) throws SecurityException
    ```

    - 应用

        ```java
        class Person { 
            private String name ;
        }
        public class Test { 
            public static void main(String[] args) throws Exception { 
            Class<?> cls = Class.forName("www.bit.java.testthread.Person") ; // 实例化本类对象 
            Object obj = cls.newInstance() ; // 操作name属性 
            Field nameField = cls.getDeclaredField("name") ; // 取消封装 
            nameField.setAccessible(true) ; 
            // ---------------------------- 
            nameField.set(obj, "epochong") ; // 相当于对象.name = "epochong" 
            System.out.println(nameField.get(obj)); // 取得属性
        	}
        }
        ```

泛型

#### 写在前面

本文主要讲述JDK1.5以后的新特性，重点讲解泛型

> - 可变参数
> - foreach循环
> - 静态导入
> - 泛型、通配符

<!--more-->

从JDK1.0开始，几乎每个版本都会提供新特性。例如在JDK中有以下代表性版本： 

- JDK1.2: 推出了轻量级的界面包：Swing 
- JDK1.5: 推出新程序结构的设计思想。 
- JDK1.8: Lambda表达式、接口定义加强 

现在已经接触了新特性：自动拆装箱、switch对String的支持

#### 可变参数

- 方法定义格式

  ```
  public [static] [final] 返回值 方法名称([参数类型 参数名称][参数类型 ... 参数名称]){}
  ```

- 实例

  ```java
  public class Test { 
  	public static void main(String[] args) { 
  		System.out.println(add("Hello")); 
  		System.out.println(add("Hello",1,4,5,6)); 
  		System.out.println(add("Hello",new int[]{1,2,3})); 
      }
  	public static int add(String msg,int ... data) { 
  		int result = 0 ; 
  		for (int i = 0; i < data.length; i++) { 
  			result += data[i] ; 
  		}
  		return result ; 
  	} 
  }
  ```

<p><font color = red>注意点：如果要传递多类参数，可变参数一定放在最后，并且只能设置一个可变参数


#### foreach循环

<p><font color = red>通过此方式可以很好的避免数组越界的问题，但是这种数组的操作只适合简单输出模式。不能在循环内直接对元素进行删除操作，否则会出现fail-fast（在我的博客Java-集合类，中有探讨）


#### 静态导入

> 从JDK1.5开始，如果类中方法含有static方法，则可以直接把这个类的方法导入进来，这样就好比像在主类中定义 的方法那样，可以被主方法直接调用静态导入类的静态方法，但不能调用非静态方法

- 定义类StaticClass

  ```java
  package com.epochong;
  
  public class StaticClass {
      public static void a() {
          System.out.println("a");
      }
      public static void b() {
          System.out.println("b");
      }
      public void c() {
          System.out.println("c");
      }
  }
  ```

- 定义类StaticImport

  ```java
  package com.epochong;
  
  //导入方式，加static
  import static com.epochong.StaticClass.*;
  
  public class StaticImport {
      public static void main(String[] args) {
          a();
          b();
          //c();不可调用
      }
  }
  ```

## 泛型

> 类定义的时候并不会设置类中的属性或方法中的参数的具体类型。
>
> 类使用时再进行定义。 

<p><font color=red> java中只有一种类型可以保存所有类型：Object型，这样存在类型转换异常ClassCastException的隐患，所以引入泛型对语法进行限制，在设置内容的时候就检查是否正确


- 泛型类

  ```java
  class MyClass<T> {
  T value1; 
  }
  
  class MyClass<T,E> {
      T value1; E value2; 
  }
  ```

  > 尖括号 <> 中的 T 被称作是**类型参数**，用于指代任何类型。

<p><font color = red> 注意：泛型只能接受类，所有的基本数据类型必须使用包装类！</font></p>
<p><font color = red>  JDK1.7 以后new后边可以不写参数，由前面的决定：</font></p>

```java
List<String> set = new ArrayList<>();
```

<p><font color = red>引入泛型后，如果明确设置了类型，则为设置类型；如果没有设置类型，
则默认为Object类型。</font></p>

- 定义泛型方法

  ```java
  class MyClass{
  //<T> 中的 T 被称为类型参数，而方法中的 T 被称为参数化类型，它不是运行时真正的参数。
  	public <T> void testMethod(T t) { 
  		System.out.println(t); 
  	} 
  }
  ```

- 泛型类和泛型方法共存

  - 泛型类中的普通方法

    > 可使用泛型类的泛型

  - 泛型类中的泛型方法

    > 泛型方法时钟以自己定义的类型参数为准，即使泛型符号相同，如果想使用泛型类的类型，应使用不同的泛型名称定义

#### 泛型接口

<p><font color = red>泛型接口的实现类有两种方式</font></p>

```java
public interface GenericityInterface<T> {
     void print(T t);
}
```

1. ##### 在子类定义时继续使用泛型

   ```java
   //泛型类必须和接口的泛型名称都是T
   public class ImplementsImpl<T> implements GenericityInterface<T> {
       @Override
       public void print(T t) {
   
       }
   }
   ```

2. ##### 在子类定义时给出具体类型

   ```java
   public class ImplementsImpl implements GenericityInterface<String> {
   
       @Override
       public void print(String s) {
   
       }
   }
   ```

   ```java
   //不指定类型默认为Object，因为接口<T> ==> <T extends Object>
   public class ImplementsImpl implements GenericityInterface {
       @Override
       public void print(Object o) {
   
       }
   }
   ```

#### 类型擦除

<p><font color = red>泛型信息只存在于代码编译阶段，在进入JVM之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除。即泛型类和普通类在 java 虚拟机内是没有什么特别的地方


- 检验类型擦除

  ```java
  public class Demo<T> {
      private T t;
  
      public T method(T t) {
          return t;
      }
  
      public static void main(String[] args) {
          Demo<String> demo = new Demo <>();
          Demo<Integer> demo1 = new Demo <>();
          //true
          System.out.println(demo.getClass().equals(demo1.getClass()));
      }
  }
  ```

```
title: 接口、抽象类
date: 2019-06-01 15:03:27
tags: java
```

### 接口抽象类

#### 接口中的固定修饰

|   类型   | 修饰符（默认不写）  |
| :------: | :------------------ |
|   接口   | abstract            |
| 接口方法 | public static final |
| 接口属性 | public abstract     |

##### 解释

- java 接口的修饰符：`public abstract`（interface 本身就是抽象的，加不加 abstract 都一样）

必须用`public`修饰，因为接口就是用来实现的，无法实例化对象，设置为其他权限修饰是毫无意义的

```java
 public interface BinTree<E> {
 }
```

- 接口中字段的修饰符：`public static final`（默认不写）

  ###### 含义

  **public:** 使接口的实现类可以使用该常量；

  **static：**接口不涉及和任何具体实例相关的细节，因此接口没有构造方法，不能被实例化，没有实例变量，只有静态变量。

  > （static 修饰就表示它属于类的，随的类的加载而存在的，当 JVM 把字节码加载进 JVM 的时候，static 修饰的成员已经在内存中存在了。如果是非 static 的话，就表示属于对象的，只有建立对象时才有它，而接口是不能建立对象的，所以接口的常量必须定义为 static。）

  **final：**接口中不可以定义变量，即定义的变量前都要加上 final 修饰，使之成为常量，且必须赋初始值！（final 修饰就是保证接口定义的常量不能被实现类去修改，如果没有 final 的话，由子类随意去修改的话，接口建立这个常量就没有意义了。

  ```java
   public  interface BinTree<E> {
       int size;
   }
  ```

- 接口中方法的修饰符:`public abstract`（默认不写）

  接口方法仅仅描述方法能做什么，但是不指定如何去做，所以接口中的方法都是抽象的

  ```java
   public interface BinTree<E> {
       boolean add(E e);
       int size();
       E getMax();
       E getMin();
       boolean contains(E e);
   }
  ```

  ```java
   interface A {
       void method();
       default void defaultMethod() {
           System.out.println("defaultMethod");
       }
       static void staticMethod(){
           System.out.println("staticMethod");
       }
   }
   class B implements A {
       public static void main(String[] args) {
           // 可以直接调用静态方法
           A.staticMethod();
           B b = new B();
           // default方法必须通过子类对象调用，不能直接在实现类中调用default()
           b.defaultMethod();
       }
   
       /**
        * 继承必须实现的方法
        */
       @Override
       public void method() {
           
       }
   }
  ```

  规定静态方法必须有方法体（普通抽象方法是不允许有方法体的）

  ##### 2. 允许接口定义静态方法

  default只能在本包下使用，观察JDK源码可以发现，Java库中的许多接口都有default方法，但是这些方法不是为开发人员准备的而是，JDK源码实现使用的，学习者很容易对这点迷惑，至少我是学习的时候是这样的

  > 原先的 jdk7 之类的，它们接口中的方法都是抽象方法，没有具体的实现。那么，如果面向接口编程，大家已经根据自己需要通过继承接口的方式来实现了自己的功能，突然有一天，产品提需求了，你需要给所有接口的实现类都添加一个新的功能即一个新的方法实现，而且这个方法可能大家都是一样的，那咋办？
  >
  > jdk8 以前的做法肯定是现在接口中定义这个抽象方法，然后所有实现类必须实现这个方法，如果实现类比较多，那改起来会很麻烦，这种情况下是不利于维护的。
  >
  > 那么我们在 jdk8 中就有了好的解决方式，就是在接口中加一个默认方法，这个默认方法有具体实现，这样就不用去修改实现类啦，很省事。

  ##### 1. 允许接口使用 `default` 关键字

  #### JDK8接口增强

  - 成员变量
    - 接口只能为 public final static 修饰，都是常量
    - 抽象类则没有限制，可以有一般的变量
  - 方法
    - 接口只能是public abstract 的抽象方法
    - 抽象类既可以有抽象方法也可以有一般方法（抽象类可以没有抽象方法）
  - 继承
    - 普通类只能继承一个抽象类
    - 普通类可以实现多个接口

  #### 接口和抽象类区别

## 通配符

> 有了泛型的定义后，避免了 ClassCastException 的问题，但是又会产生新的情况：参数的统一问题。 

- <? extends T>：是指 “上界通配符（Upper Bounds Wildcards）”
- <? super T>：是指 “下界通配 符（Lower Bounds Wildcards）”

<!--more-->

##### 问题引出 

- 观察以下程序

  ```java
  public class MyDemo<T> {
      T t;
  
      public T getT() {
          return t;
      }
  
      public void setT(T t) {
          this.t = t;
      }
  }
  class MyDemo1 {
      public static void main(String[] args) {
          MyDemo<Integer> myDemo = new MyDemo <>();
          //无法通过
          method(myDemo);
      }
      public static void method(MyDemo<String> myDemo) {
  
      }
  }
  ```

<p><font color = red>由上面的程序可以看出，如果指定参数的类型为确定的类型，则不能传不同类型的参数，使用通配符？可以很好的解决这一问题


- 将method改为以下方式即可，因为不知道具体类型只能获取值，不能修改值

  ```java
  public static void method(MyDemo<?> myDemo) {
      //无法修改值，报错
      myDemo.setT(11);
      //可以获取值
      System.out.println(myDemo.getT());
  }
  ```

#### 泛型边界

> 泛型有上界和下届，通过下面程序讲解

- 定义类

  ```java
  public class Person {
  }
  
  class Student extends Person {
  
  }
  
  class SmallStudnet extends Student {
  
  }
  
  class PersonObject<T> {
      T person;
  
      public T getPerson() {
          return person;
      }
  
      public void setPerson(T person) {
          this.person = person;
      }
  }
  ```

##### 泛型上界

- <? extends T>：是指 “上界通配符（Upper Bounds Wildcards）”

  - 不能往里存，因为不确定具体存的是父类还是子类
  - 只能往外取，因为一定是T或T的子类，只能用T或者T的父类接收

  ```java
  @Test
  public void testExtends() {
      PersonObject<? extends Person> personObject = new PersonObject <>();
      Person person = new Person();
      Student student = new Student();
      SmallStudnet smallStudnet = new SmallStudnet();
      //不能设置值
      personObject.setPerson(person);
      //可以获取值
      Person person1 = personObject.getPerson();
      System.out.println(person1);
  }
  ```

##### 泛型下界

- <? super T>：是指 “下界通配 符（Lower Bounds Wildcards）”

  - 不影响往里存，因为T已经确定，元素一定是T或者T的父类，那往里存粒度比 T小的都可以
  - 可以往外取，但只能放在 Object 对象里，由于T的父类可用有很多也可能是默认的Object类，所以不知道是哪一个，所以必须要用Object来接收

  ```java
  @Test
  public void testSuper() {
      PersonObject<? super Person> personObject = new PersonObject <>();
      Person person = new Person();
      Student student = new Student();
      SmallStudnet smallStudnet = new SmallStudnet();
      //下面都是合法的
      personObject.setPerson(person);
      personObject.setPerson(student);
      personObject.setPerson(smallStudnet);
      Object person1 = personObject.getPerson();
      System.out.println(person1);
  }
  ```



