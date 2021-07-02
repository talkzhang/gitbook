> 单位小伙伴贡献的文档，拿过来日后复习

# 认识Python

Python是一种高级语言，同时支持面向对象编程与函数式编程

Python是一种***解释型语言***	[参考](https://www.cnblogs.com/nanhe/p/13219165.html)

Python是一种***动态类型语言***

> 解释型语言：相比较**编译型语言**，优点是平台可移植性更高，缺点是执行效率低。
>
> 动态类型语言：相比及**静态类型语言**，代码的可复用性更强，代码更加灵活、简洁，缺点是编写代码时没有类型检查，bug更多，同时执行效率更低一点。

Python的其他一些特点：

1. 语法比较简单、易于学习、上手容易、可嵌入性
2. 非常丰富的库。Python本身的标准库非常强大，同时第三方库也非常丰富。不过这也算是一个缺点吧，可框架选择太多了。

Python的解释器：

默认的是CPython，为官方内置的解释器，其他还有一些JPython、IPython等。



# Python的数据类型

> 在 Python 中，变量就是变量，它没有类型，我们所说的"类型"是变量所指的内存中对象的类型。
>
> Python中，变量无须声明，但是在使用前必须先赋值，变量赋值后才会被创建。

6个标准的数据类型：

1. **不可变数据（3 个）：**Number（数字）、String（字符串）、Tuple（元组）
2. **可变数据（3 个）：**List（列表）、Dictionary（字典）、Set（集合）

对于以上6种数据类型，其中**Tuple、List、Dictionary、Set 为容器类型**

自定义的数据类型，是属于可变数据类型

> 注意点：
>
> bool类型，是Number类型中Int类型的子类



# Python的包管理

## PIP 包管理器

pip是Python的包管理器，提供对python包的查找、下载、安装、卸载的功能。

pip的安装命令默认如下，表示从PyPI（https://pypi.org/）下载按照werkzeug的最新版本：

```
pip install werkzeug
```

指定版本安装：
```
pip install PyjQuery==3.5.1.0
```

指定源安装：

```
pip install -i https://pypi.douban.com/simple PyjQuery
```

pip默认的源是https://pypi.org/simple，可以通过命令行修改，如下：

```
pip config set global.index-url www.baidu.com
```

其他一些Pip的常用命令，可以参考【[PIP 参考手册](https://pip.pypa.io/en/stable/)】

## requirements.txt

通常我们可以通过创建requirements.txt文件，记录所有的依赖包和版本，方便新环境的快速部署。

requirements.txt 文件可以通过手动创建，也可以通过命令自动生成，`自动生成的requirements.txt，包含当前环境下所有的包依赖`:

```
pip freeze > requirements.txt
```

通过 requirements.txt, 快速安装依赖：

```
pip install -r requirements.txt
```

## 虚拟环境

经常会有一些应用程序，需要使用特定版本的Python或者包。比如：A程序和B程序都引用了同一个Package，但是要求的版本不同，不管是安装那个版本的Package，都会导致另外一个程序不可用。此时我们可以通过为应用程序创建单独的虚拟环境。

实现虚拟环境的模块有[virtualenv](https://packaging.python.org/key_projects/#virtualenv)、[virtualenvwrapper](https://bitbucket.org/dhellmann/virtualenvwrapper)、[venv](https://docs.python.org/3/library/venv.html)、[pipenv](https://pypi.org/project/pipenv/). 下面我通过pipenv 演示，如何使用虚拟环境。

> 1. 安装pipenv
>
>    ```
>    pip install pipenv
>    # 安装完成之后，可以通过pipenv -h 查看pipenv命令帮助
>    ```
>
> 2. 针对应用程序安装一个虚拟环境
>
>    ```
>    # 进入到应用程序的根目录下面，安装一个Python3.9的虚拟环境（本地必须已经安装了Python3.9）
>    pipenv --python 3.9
>    #安装好了之后，可以进入虚拟环境：如果没有安装直接进入虚拟环境，会默认创建一个虚拟环境，基于本地的Python环境
>    pipenv shell
>    ```
>
> 3. 为应用程序安装包
>
>    ```
>    # 进入虚拟环境之后，操作和平常完全一样，使用pip进行操作
>    # 查看安装的包
>    pip list
>    # 安装包
>    pip install -r requirements.txt
>    ```
>
> 4. 使用虚拟环境
>
>    如果我们使用idea等IDE工具时，可以通过配置Python使用指定的虚拟环境，此时在Terminal中使用python和pip命令，默认是使用虚拟环境中的。
>
>    如果我们是通过cmd或者其他一些终端工具时，此时需要指定需要使用的Python



# Python的内存管理和垃圾回收

1. pymalloc 分配器和内存池的概念	[Python内存管理]( https://docs.python.org/zh-cn/3.9/c-api/memory.html)

   python为短生命周期的小对象（小于或等于 512 字节），直接在python维护的内存池进行分配。

   pymalloc 分配器会预先申请一块连续的空间（默认256KB）用于小对象的内存分配，当小对象释放时，内存会还给内存池，而不是直接释放。

2. Python 默认采用引用计数法进行垃圾回收

   > 一旦没有引用立即回收，处理回收内存的时间分摊到了平时。
   >
   > ***问题点：`循环引用问题`***
3. 标记清除 	[CPython 垃圾收集器的设计](https://devguide.python.org/garbage_collector/#collecting-the-oldest-generation)

   > CPython维护两个`双向链表`，一个为可达对象列表，一个为不可达对象列表，为了实现这个功能，对象头需要加上**PyGC_Head**
   >
   > 具体的过程如下：
   >
   > 1. 每个对象都会额外增加gc_ref字段，该字段被初始化为该对象的引用计数
   > 2. 遍历第一个列表中的所有容器对象，并将容器引用的任何其他对象的 gc_ref字段减一
   > 3. gc_ref为 0的对象标记为“暂时不可到达”，然后将它们暂时移动到不可达的列表中
   > 4. gc_ref大于0的对象，它使用 tp_traverse 遍历其引用，以查找从它可到达的所有对象，将它们移动到可到达对象列表的末尾，并将其 gc_ref 字段设置为1，为了避免访问一个对象两次，GC 将标记所有已经访问过一次的对象（取消 prev_mask_collect 标志）
   > 5. 遍历完所有的对象后，对于不可达的列表的对象进行垃圾回收尝试
   >
   > ***注意点：如果一个元组或者字段，内部只有不可变对象时，GC不会对这些对象进行跟踪***

5. 分代回收	[Python GC](https://docs.python.org/zh-cn/3.9/library/gc.html?highlight=get_threshold#module-gc)
   
    > weak generational hypothesis：假设大多数对象的生命周期非常短，因此可以在创建后不久收集，一个对象存活的越久，则其继续存活更长的时间。
    >
    > 为了限制每次垃圾收集所需的时间，所有容器对象都被隔离成三个空间/代。每个新对象都从第一代(第0代)开始分配。越靠后的代里面的对象则越少被检查和回收。
    >
    > 分代垃圾回收的阈值: 可以通过gc模块的gc.get_threshold()方法获取，默认是（700,10,10），代表（对象的个数，0代回收10次执行一次1代回收，1代回收10次进行一次2代回收），可以通过gc.set_threshold()修改
    >
    > 新增对象的数量/现有对象的数量 的比值高于给定值(固定25%) ，也则会触发2代的完整GC
    >
    > ***思考：跨代引用如何处理？***

Python默认开启了自动垃圾回收，用于引用计数无法回收的对象进行垃圾回收，如果你确定你的程序不会产生循环引用，你可以关闭回收器。可以通过调用 `gc.disable()` 关闭自动垃圾回收。

可以通过gc.collect(i)手动触发指定代的垃圾回收

