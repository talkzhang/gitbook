好文地址：https://www.javazhiyin.com/27630.html#m

https://www.cnblogs.com/rickiyang/p/11368932.html

https://tech.meituan.com/2019/11/07/java-dynamic-debugging-technology.html

# 小结
1. jdk5时，提供了permain方法，顾名思义，可以在目标类执行前做的事情。如果premain(String args, Instrumentation inst)和premain(String args)同时存在时，优先使用前者。其中方法参数args即命令中的options，类型为String（注意不是String[]），因此如果需要多个参数，需要在方法中自己处理（比如用”;”分割多个参数之类）；inst是运行时由VM自动传入的Instrumentation实例，可以用于获取VM信息。
2. jkd6时，提供了agentmain 方法，参数和上面的premain一样，只不过方法名发生了变化，它的特性就是可在main方法执行过程中进行某些操作，已达到目的。
3. 无论是premain或者是agentmain，这两个方法在如果存在多个时，即(String args)或者(String args, Instrumentation inst)，带有Instrumentation参数的执行优先级更高。
4. MANIFEST.MF，需要在该文件中声明premain-class、agent-class的路径，MANIFEST.MF的路径在src/java/resources目录下。
5. 使用原理：JVMTI（Java Virtual Machine Tool Interface）是一套本地编程接口集合，它提供了一套”代理”程序机制，可以支持第三方工具程序以代理的方式连接和访问 JVM，并利用 JVMTI 提供的丰富的编程接口，完成很多跟 JVM 相关的功能。java.lang.instrument 包的实现，也就是基于这种机制的：在 Instrumentation 的实现当中，存在一个 JVMTI 的代理程序，通过调用 JVMTI 当中 Java 类相关的函数来完成 Java 类的动态操作。具体类图如下：![](https://cdn.jsdelivr.net/gh/talkzhang/imgs-bed@master/image/javaagent%E4%B8%BB%E8%A6%81%E5%B7%A5%E4%BD%9C%E7%B1%BB.png)
6. 现有工具如lombok、arthas监控等都是利用该技术实现。