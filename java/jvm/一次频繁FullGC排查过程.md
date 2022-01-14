先说什么是FullGC，它是发生在jvm虚拟机上面，一种用于清理内存的行为，这里不说jvm理论，重点提一下full gc发生时，会让整个jvm堆产生stop the world，这对系统以及用户体验影响很大，此次现象是某个服务频繁fullgc但没有发生oom这类异常，部署方式是两台实例，最高达到两台实例一天200多次fullgc。

## 问题分析

这种情况下首先考虑的应该是服务项目内频繁有大对象创建导致堆内存吃紧，无法容纳所以只能通过fullgc来满足该大对象的容纳空间，这样的话，只要找到某个大对象基本可以定位问题所在。

那如何找到这个大对象？这里需要拉取项目的dump文件来进行分析，能够更快更准确的定位，基本思路就是如此。

那说白了，就两个问题，怎么获取项目的dump文件？获取下来后怎么分析？

### 获取dump文件

由于服务在生产环境运行，理想状态是在服务运行过程中获取想要的dump文件，java提供了jinfo命令来进行临时配置获取，无需重启vm，即时生效，dump文件生成后，清除VM参数，通常fullgc 会频繁发生，不需要一直导出dump，所以拿到一次的dump采样后， 即可清除，然后慢慢分析dump文件即可。

第一步，通过jps获得java程序的pid（jps,ps等方法），我们通过docker部署，进程号默认就是1，也可以通过jps 获取java某个服务的进程，没什么区别，这一步就是为了确认pid

第二步，调用jinfo命令设置VM参数

```bash
#jinfo -flag +HeapDumpBeforeFullGC 1

#jinfo -flag +HeapDumpAfterFullGC 1
```

使用 #jinfo -flags pid 可以检查该配置有没有生效，该配置顾名思义，主要是告诉jvm在fullgc之前和之后生成dump文件。

生成后注意下路径问题，默认路径就是java进程的工作目录，该目录和代码`System.getProperty("user.dir")`应该是一致的（未测试过，根据现象推测）。

生成文件后，将该文件发送到本地，同时采样已完成，避免频繁生成dump文件，将



参考连接：https://blog.csdn.net/kevin_mails/article/details/103404883