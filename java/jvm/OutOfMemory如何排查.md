最近线上项目有开始出现oom类型错误，为了方便下次排查，所以对java项目如何发生oom应该如何处理步骤流程大致梳理一下，方便日后使用。

# 如何能快速查看到异常堆栈信息

任何java项目，在运行过程中可以通过命令来实时获取该项目的堆栈详细数据信息，同时也可以设置在发生`OutOfMemory`时自动生成dump文件来供我们本地分析问题。

## 运行时获取dump文件

- 首先通过命令行找到当前运行项目在服务器上的pid，也就是进程号，获取方式很多种，如果你是linux系统，可以通过`ps -f |grep xxx.jar` 来定位，或者通过`jps`来查看。

- 获取到pid之后，就可以通过`jmap`命令来进行导出堆文件的导出。示例：

```
jmap -dump:file=javaDump.dump,format=b 24552
```

这样会获取到一个名为javaDump.dump的文件。

## 发生oom异常时自动生成dump文件

这个可以通过jvm启动参数来进行获取。

```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./
```

如上，会在发生oom时，将dump文件输出到根目录下。

# 如何分析dump文件

dump文件搞定了，接下来是如何分析呢？如何通过工具或者别的方式来对dump出来的文件进行一个深入解读？

## jhat分析

jdk中有自带jhat工具来对文件进行在线分析，通过`jhat xxx.dump/hprof`，之后，本地访问`http://localhost:7000`，即可在浏览器看到访问效果，如图：

![通过jhat启动查看dump](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103487331830.png)

![jhat页面1](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103490572886.png)

![jhat页面2](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1610349010325.png)

![jhat页面3](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103492726863.png)

页面大致是长这个样子，重点查看 Other Queries目录下的SHow heap histogram，可以比较容易观察到对象的引用次数及占用内存的大小。

使用jhat优点是非常方便，通过命令行即可查看视图，缺点是如果文件过大，容易导致浏览器卡崩，而且都是通过纯文本描述，阅览上并不够直观。

## jvisualvm分析

jdk的另一个工具，visualvm也可以对dump文件进行可视化分析，且功能比较强大，可以定位到具体运行的线程错误（代码行），虽然分析jvm内存是一件非常复杂工作，即使定位到代码行也不一定就是该位置导致的oom，但是和jhat相比，jvisualvm有很大优势。

找到所在操作系统内的`$JAVA_HOME/bin/jvisualvm.exe`，双击即可启动，启动后左上角`文件->装入`，选择指定的要分析的文件即可。

![jvisualvm界面图](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103500291910.png)

综合来看还是使用jvisualvm来进行dump文件的查询分析更直观、更利于定位问题。

## 测试案例

本地编写一块测试代码验证一下流程，方便记忆。

```java
public class Test {

    public static void main(String[] args) throws IOException {
        List<String> list = new ArrayList<>();
        int cnt = 0;
        for (; ;) {
            list.add(new String("i am couter this is " + cnt));
        }
    }
}
```

编写如上代码，且设置一下jvm启动参数：`-Xmx20m -Xms20m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./`，在IDEA内设置参数非常方便：

![IDEA设置jvm启动参数](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103505327086.png)

![IDEA设置jvm启动参数](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103507065766.png)

这里需要注意，在测试时，不要设置-Xms太小，如果太小会在启动时报错：
```
Error occurred during initialization of VM 
GC triggered before VM initialization comA1eted. Try increasing NewSize, current value 1536K . 
```

设置完之后启动main方法，在发生oom之后会自动导出dump文件到当前目录下。

![oom控制台信息](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1610351281678.png)

![oom文件](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103514014791.png)

将这个文件通过分析工具来定位查看信息，通过[概要]项可以定位哪个该线程触发了错误信息，且户定位到代码行。

![jvva visual VM page 1](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1610351484622.png)

通过[类]项可以更清晰的看到对象引用记录。

![jvva visual VM page 2](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103516355191.png)

可以看到引用最多的是char数组，其实就是string，因为string的底层结构就是char[]，双击char[]进去之后，可以详细看到究竟是谁在引用它。

![jvva visual VM page 3](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16103519112087.png)

可以看到，引用该类实例的是ArrayList持续装载，且可以定位到具体文本内容。

# 扩展补充 OutOfMemoryError的几种类型

- **OutOfMemoryError: GC overhead limit exceeded**

    这是指程序基本耗尽了所有的内存 GC都清理不了，意味着GC占用了大部分CPU周期，大多数意味着98％或99％，并且在每次运行中它释放的内存非常少1-2％对别的线程影响很大影响整个系统吞吐量，比如一个线程占用98%内存无法释放，其他线程执行过程中由于内存限制就会抛出这个错误，甚至会导致连接数据库等这种操作变的缓慢，出现类似一个请求处理好几分钟的现象。



- **OutOfMemoryError: Java heap space**

    相当于将xl型号的东西往只有s号空间内塞的意思，意味着应用程序在某些处理期间内存不足。


- **OutOfMemoryError: Permgen space | OutOfMemoryError: Metaspace**

    在java8之前存在用永久代来实现方法区，这个永久代归堆管理，但是java8之后，将方法区的实现放在元空间，那么方法区内存不足时，在java8之前会出现：permgen space的oom错误，java8之后，也就是方法区通过元空间来实现，这个区域内存不足时会报：Metaspace


# 总结

在排查oom问题时，排查步骤如下：

1. 设置jvm启动参数，`-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./`，在发生oom时自动生成dump文件。

2. 如果是生产服务器，可以将文件发送到本地，通过jvisualvm进行排查。