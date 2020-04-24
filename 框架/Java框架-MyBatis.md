Spring Boot+MyBatis+Druid的使用是基础，在项目中贯穿。
# 1. MyBatis简介
答：MyBatis是一个**半ORM**的框架，**内部封装了JDBC，通过xml文件或注解配置信息**。通过Java对象和statement的SQL参数映射执行SQL语句生成Java对象返回。

## 1.1 ORM是什么
答：ORM(Object Relational Mapping)对象关系映射，解决关系型数据库数据和简单Java对象(POJO)的映射关系的技术。

MyBatis是半ORM，因为还需手动编写sql查询关联对象。
## 1.2 MyBatis和JDBC的区别
答：总结为：
- MyBatis代码量少，开发速度快，移植性好，缺点是sql语句自己写；
- JDBC更灵活但存在硬编码和数据库连接开启关闭频繁。

# 2. 核心组件
答：MyBatis的核心组件包括SqlSessionFactoryBuilder，SqlSessionFactory，SqlSession和Mapper。
## 2.1 SqlSessionFactoryBuilder
SqlSessionFactoryBuilder是一个构造器，通过配置文件或者java代码获得资源构造工厂类。可以构造多个工厂。生命周期在方法内部，用完就回收。
```java
SqlSessionFactory  factory  = SqlSessionFactoryBuilder.build(inputStream);
```
## 2.2 SqlSessionFactory
SqlSessionFactory的作用就是创建SqlSession，一个会话。用单例模式创建SqlSession。生命周期贯穿整个MyBatis。
```java
SqlSession sqlSession = SqlSessionFactory.openSession();
```
## 2.3 SqlSession
SqlSession就是一个会话，作用是发送SQL语句和获取Mapper接口。生命周期是请求数据库处理事务的过程中。注意每次SqlSession使用完后需要关闭，减少资源消耗。
## 2.4 Mapper
Mapper映射器，给出对应的SQL和映射规则。作用是发送SQL去执行并返回结果。
```java
XXMapper xxMapper = sqlSession.getMapper(XXMapper.class);
```
# 3. 运行原理
## 3.1 工作原理
![MyBatis原理](D2654FACB2F2470FAD0F527D23BF8C52)
1. 读取MyBtis配置文件；
2. 加载映射文件，也就是写了SQL的文件；
3. 构造会话工厂；
4. 创建会话对象；
5. Executor执行器，操作数据库；
6. MappedStatement对象：对映射信息的封装，存储sql的id参数等信息；
7. 输入参数映射；
8. 输出结果映射；

整个架构就是：加载配置——>解析SQL——>执行SQL——>结果映射
## 3.2 Executor执行器
答：MyBatis中有三种基本Executor执行器。
- SimpleExecutor：每执行一次update或select，就开启一个statement，用完立马关闭；
- ReuseExecutor：重复使用statement，用一个hashmap存储；
- BatchExecutor：重复使用+批量更新。

## 3.3 延迟加载
懒加载，即**先查询主表信息，若用到从表数据，再去查询从表信息，不用到，就不查询**。

### 3.3.1 原理
答：使用CGLIB创建代理，当调用目标方法时，进入拦截器，若子表信息为空，则调用子表的sql获得信息，之后继续执行。

eg.调用a.getB().getName()，拦截器发现getB()是null，就单独运行原本存储的关联B的SQL，得到查询结果再调用setB()，最后继续执行原来的命令
。
### 3.3.2 操作步骤
1. 开启延迟加载
在全局配置文件中，添加setting。
```java
<settings>
  //打开延迟加载的开关，默认为true
  <setting name="lazyLoadingEnabled" value="true"/>
  //积极的懒加载，默认是true，设置为false时，懒加载生效
  <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```
2. 编写主表的查询映射
比如orders，一对一查询，user。
在association中以select属性选取从表查询。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200210213234438.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
3. 正常编写从表的查询映射
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200210213337690.png)


# 4. 映射器
## 4.1 MyBatis中#和$的区别
答：**能用#就用#** 。区别在：
- #符号相当于占位符，$符号相当于拼接SQL串；
- #符号将传入的数据都当成一个字符串，再把#{}替换为?，$符号将传入的数据直接原样输出；
- #符号有预编译，能防止SQL注入，$符号是直译，不能防止注入。

## 4.2 模糊查询like怎么写
答：
- '%${ming}%'：不能防止注入；
- "%"#{ming}"%"
- concat('%',#{ming},'%')：使用concat()函数；
- bind标签：
```java
<bind name="pattern" value="'%' + username + '%'" />
select id,sex,age,username,password from person where username LIKE #{pattern}
```
## 4.3 mapper传递参数
答：四种方式。
- 顺序传参：一般不用，不清晰。username=#{0}
- @Param注解传参：参数不多，用这个。username=#{userName}
- Map传参：参数多，用这个。#{}里的名称对应Map的key，值对应value。
- Java Bean传参：逆向工程时常用。一般就用Bean实体做对应，如果有特殊需要就再封装个POJO。

## 4.4 接口绑定实现方式
答：接口绑定就是在MyBatis中定义接口，然后把接口里的方法和SQL语句绑定。
- 通过注解绑定。在方法上添加@Select、@Update注解 + SQL语句；
- 通过xml里写SQL绑定。注意mapper的开发规范。

开发规范：
- 接口的全路径名要与映射文件中的namespace相同；
- 方法名要与映射文件中的statement的id相同；
- 方法参数为传递给SQL的参数；
- 返回值要与映射文件中的resultType或resultMap一致。

## 4.5 Dao接口的工作原理
答：Dao接口就是Mapper接口。工作原理是**JDK的动态代理**，为Mapper接口生成代理对象，代理对象会拦截接口方法，执行MapperStatement对应的SQL，将结果返回。
### 4.5.1 接口方法可以重载吗？
答：不能重载，因为接口内部使用 **全限定名+方法名** 保存和查找。
### 4.5.2 不同的xml文件中id能重复吗？
答：xml文件如果配置了namespace，就可以重复；没有配置，不能重复。

## 4.6 xml和内部数据结构的映射关系
答：所有的xml信息都封装在Configuration内部。
- <parameterMap>标签会被解析为ParameterMap对象，其每个子元素会被解析为ParameterMapping对象。
- <resultMap>标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象。
- 每一个<select>、<insert>、<update>、<delete>标签均会被解析为MappedStatement对象，标签内的sql会被解析为BoundSql对象。


# 5. 高级查询
## 5.1 一对一和一对多查询
答：联合查询和嵌套查询。
- 联合查询：几个表联合，只查一次。
- 嵌套查询：先查一个表，得到id，再去查其他表。
- association一对一，collection一对多。

## 5.2 动态SQL
答：在写xml时能用if、where、foreach等标签，**根据表达式的值完成逻辑判断并动态拼接SQL语句**。
总共提供了9种标签：
- if：单条件分支
- choose、when、otherwise：多条件分支
- foreach：循环遍历
- trim、where、set：辅助元素，实现SQL拼接
- bind：模糊查询
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvODc0NzEwLzIwMTcwNC84NzQ3MTAtMjAxNzA0MTYxNTM1NDQ4MDYtMTI0NzI4NzA5NS5wbmc?x-oss-process=image/format,png)
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvODc0NzEwLzIwMTcwNC84NzQ3MTAtMjAxNzA0MTYxNTUzNTYwNTYtMTcwODU2MDczNi5wbmc?x-oss-process=image/format,png)
我们还能把动态SQL提取出来，用id标志。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200210143958170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

# 6. MyBatis的缓存机制
答：MyBatis分为一级和二级缓存。
- 一级缓存：**SqlSession级别**的缓存，基于HashMap实现，默认开启；
- 二级缓存：**Mapper级别**的缓存，多个SqlSession共用，基于HashMap实现。默认不开启，开启后，一级缓存失效。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200210144110154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)
