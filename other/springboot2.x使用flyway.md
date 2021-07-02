# 什么是flyway

flyway是一个数据库管理工具，具体介绍详见官网。

官网地址：[https://flywaydb.org/](https://flywaydb.org/)

# 为什么使用flyway

什么？听过代码版本管理工具git、svn，数据库还管理，它管的什么，解决了什么问题？

我把它总结如下：

1. 自己写的SQL忘了在所有环境执行；
2. 别人写的SQL我们不能确定是否都在所有环境执行过了；
3. 有人修改了已经执行过的SQL，期望再次执行；
4. 需要新增环境做数据迁移；
5. 每次发版需要手动控制先发DB版本，再发布应用版本；

# flyway如何使用

我们项目使用springboot2.x，所以在本地测试时使用springboot2.x来进行本地验证，话不多说，上代码。

## 导入依赖

我们使用maven进行版本管理，所以在pom中导入如下依赖，关于其他依赖就不贴了，如果还不熟悉maven的可以自行搜索学习。
```xml
<!-- https://mvnrepository.com/artifact/org.flywaydb/flyway-core -->
		<dependency>
			<groupId>org.flywaydb</groupId>
			<artifactId>flyway-core</artifactId>
			<version>5.2.4</version>
		</dependency>
```

## application配置

```
# 数据库配置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=your datasource url
spring.datasource.username=username
spring.datasource.password=password

# 启用或禁用 flyway
spring.flyway.enabled=true
# 如果没有 flyway_schema_history 这个 metadata 表， 在执行 flyway migrate 命令之前, 必须先执行 flyway baseline 命令
# 设置为 true 后 flyway 将在需要 baseline 的时候, 自动执行一次 baseline。
spring.flyway.baseline-on-migrate=true
# SQL 脚本的目录,多个路径使用逗号分隔 默认值 classpath:db/migration
spring.flyway.locations=classpath:flyway
# 是否检查脚本
spring.flyway.check-location=true
```

关于该配置，有如下注意事项：

1. 数据库配置成正常在使用的。
2. `spring.flyway.locations`配置的路径是`resources`目录下;
3. 在目录下放置的SQL脚本，命名规则需要注意，是`V`开始后面写一些数字或者下划线（单下划线）都可以，然后分隔符号`__`（双下划线），后面跟`xx.sql`，例如：`V1.2__init.sql`。

为了更直观，目录结构如下图：

![项目目录结构](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16122349286773.png)

下图中的sql脚本信息：

```sql
CREATE TABLE `test_edu_student` (
  `stu_id` varchar(16) NOT NULL COMMENT '学号',
  `stu_name` varchar(20) NOT NULL COMMENT '学生姓名',
  PRIMARY KEY (`stu_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='学生表';
```

# 遇到的问题

现在问题是在第一次使用时，数据库还不存在`flyway_schema_history`这张表，在使用如上配置之后，只是初始化创建了一条数据，但是并没有执行指定目录下的SQL，有一条默认数据，只有把它删掉之后再重启服务才会继续执行我的脚本，如果不删这条数据，而重启服务的话，在控制台输入如下：

![控制台异常输出](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16122372654554.png)

贴一下数据库默认的这条数据：

![数据库默认数据](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16122366361509.png)

只有删掉这条数据之后，在重启服务才会执行我的脚本，不知道这是我的问题，还是flyway的设计本就如此。

![成功执行数据](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16122370184208.png)

![成功执行控制台](https://gitee.com/hongqigg/imgs-bed/raw/master/image/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16122374662292.png)


---
