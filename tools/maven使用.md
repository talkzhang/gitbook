# maven scope作用域

maven作为比较流程的java项目依赖管理，对java开发者来讲dependency的scope标签并不罕见，例如我们经常会在项目的pom文件中看到类似这样的写法：
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
</dependency>
```
maven的scope，这是很常见的junit依赖，可以看到它使用了`scope`来声明它的作用域是`test`，那maven有几种作用域，分别是什么？

- scope作用域
1. compile， 缺省值，适用于所有阶段，即编译、测试、运行、打包。
2. provided，类似compile，期望JDK、容器或使用者会提供这个依赖，在编译和测试时使用，在容器启动时如果包含了这个依赖，不会冲突，而是把provided作用域的exclude动作。如servlet-api-2.3.jar。
3. runtime， 在运行时使用，如JDBC驱动，适用运行和测试阶段。   如plexus-utils-1.1.jar
4. test，   只在测试时使用，用于编译和运行测试代码。不会随项目发布。如Junit-3.8.1.jar
5. system， 类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

# maven如何构建父子级项目

在多模块的maven项目中 , 如果需要部署某个子模块 , 单独构建则会报错 , 如果构建整个项目 , 又会非常耗时 . 

maven为自定义构建部分项目提供了支持 : 

```bash
-pl, --projects
    构建指定的模块，模块间用逗号分隔；适合无依赖的项目
-am, --also-make (常用)
    同时构建所列模块的依赖模块，比如A依赖B，B依赖C，构建B，同时构建C
-amd, --also-make-dependents
        同时构建依赖于所列模块的模块，比如A依赖B，B依赖C,构建B，同时构建A
```

首先切换到maven父项目目录 , 单独构建web-a , 同时会构建 web-a 依赖的其他模块

```bash
mvn install -pl web-a -am
```

单独构建common , 同时构建依赖于common的其他模块

```bash
mvn install -pl common -am -amd
```