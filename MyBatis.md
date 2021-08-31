# MyBatis
* 半自动化的持久层框架，相比于 Hibernate 和 JPA，二者对长难复杂 SQL 处理不容易，内部自动生产的 SQL 不容易做特殊优化。基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难，导致数据库性能下降
* 对开发人员而言，核心 sql 还是需要自己优化。sql 和 Java 编码分开，一个专注业务，一个专注数据。

## 1. xml 全局配置文件
* MyBatis 提供了 xml 文件配置和 Configuration 类两种方式进行全局配置。框架内部，xml 配置最终也会转换为 Configuration 类型的对象
* xml 配置文件命名习惯有 mybatis-config.xml。全局配置文件除了配置数据连接和事务等基本属性，还可配置框架提供的一些特性功能
* 基本配置包括：环境、数据源、事务、属性、参数配置、映射引用
* 进阶配置包括：别名、类型处理器、对象工厂、对象封装工厂、反射工厂、数据库提供商、插件
* 配置以上元素需遵循先后顺序，原则是基本属性和参数设置放在前面

## 2. xml 映射文件配置
* 映射文件用来配置接口方法的 SQL 执行语句和返回结果类型的映射，在调用映射器接口方法进行数据操作时，MyBatis 框架会实际执行该配置中的 SQL 语句，并将数据库执行的结果转为映射配置的 Java 类型对象
* 映射文件一般和实体类相对应，用户类为 User，则映射文件为 UserMapper.xml
* &lt;select&gt; 基本查询配置
```xml
<select id="selectUser" resultType="com.wwb.User">
	select * from User where id = #{id}
</select>
```
* id 用来唯一标识 SQL 语句，值保持与接口类的方法名一致
* resultType 用来指定返回转换的 Java 对象类型，框架会据此配置将数据库查询结果自动转换
### (1). 参数替换方式与类型
* ${}：直接替换语句中的变量
* #{}：将传入的值当成字符串，会对 sql 语句进行预处理，将 sql 中的 #{} 替换为 ? ，调用 PreparedStatement 的 set 方法来赋值
### (2). &lt;resultMap&gt;
* 在 &lt;resultMap&gt; 下添加子标签实现对每个属性和字段的映射。子标签 &lt;id&gt; 用于配置属性和主键字段的映射，&lt;result&gt; 用于配置属性与一般表字段的映射
### (3). &lt;select&gt; 嵌套映射
* 使用 &lt;id&gt; 和 &lt;result&gt; 来配置字段和属性的映射时，映射的字段都是该数据表中存在的字段。字段和属性名称若不同则通过 SQL 的 as 关键字或定义 &lt;resultMap&gt; 进行手动栏位映射
* 若某个属性的值不是来自当前这张表的字段，而是从其他关联表查询出来的，则需在 resultMap 中使用嵌套的属性映射
* &lt;association&gt;：对应一对一的关系，将关联查询结果映射到一个 POJO 对象中
* &lt;collection&gt;：对应一对多的关系，将关联查询结果映射到一个 list 集合中
* &lt;discriminator&gt;：动态映射关系，可以根据条件返回不同类型的实例
* 注：MyBatis 支持 association 和 collection 的延迟加载，可通过 fetchType 设置 lazy（延迟加载）和 eager（立即加载）。延迟加载的原理是使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法
### (4). &lt;insert&gt; 插入数据，&lt;update&gt; 更新数据，&lt;delete&gt; 删除数据
### (5). &lt;sql&gt; 可重用代码段
* SQL 代码段通过 &lt;include&gt; 被其他的 &lt;select&gt; 等使用

## 3. MyBatis 核心接口和类
### (1). SqlSession（SQL 会话接口）
* MyBatis 执行数据操作的主要接口，定义了执行 SQL、获取映射器和管理事务的方法
* 框架内部，使用 TransactionFactory 创建 SqlSession 的 Connection 实例，也可以使用现有的 Connection 对象作为参数获取 SqlSession 实例。SqlSession 接口的默认实现类是 DefaultSqlSession
### (2). SqlSessionFactory（SQL 会话工厂接口）
* 定义了获取 SQL 会话对象和全局配置实例的方法。openSession() 用来获取 SqlSession 的实例。
* getConfiguration() 获取全局配置对象。默认实现类是 DefaultSqlSessionFactory
### (3). SqlSessionFactoryBuilder（SQL 会话工厂构建类）
* 提供了不同参数的 build() 重载方法用来创建 SQL 会话工厂实例
### (4). Configuration（配置类）
* SqlSessionFactoryBuilder 处理使用配置文件输入流作为参数，还可使用 Configuration 类型参数创建 SqlSessionFactory。框架内部，配置文件的输入流最终也会被转化为 Configuration 类型的实例。



## 4. MyBatis 内部运作
### (1). 会话工厂创建会话的流程
* SqlSessionFactory 的 openSession() 内部处理流程：
    - 从环境对象（Environment）获取事务工厂（TransactionFactory）
        - TransactionFactory 是事务工厂接口，根据不同的事务管理机制，MyBatis 提供两种实现类
        - JdbcTransactionFactory：JDBC 事务管理机制
        - ManagedTransactionFactory：MyBatis 本身不管理事务，让程序运行的容器（如 JBoss 和 WebLogic）管理
    - 通过事务工厂 TransactionFactory 创建 MyBatis 的事务（Transaction）对象
        - 对应两种事务工厂类型，提供两种事务接口实现：
        - JdbcTransaction：JDBC 类型事务
        - ManagedTransaction：容器管理的事务
    - 使用事务（Transaction）创建不同类型的执行器（Executor）
        - Executor 是实际用来执行数据操作和事务处理的方法，通过 Configuration 的 newExecutor()创建，一句不同执行器类型创建对应类型的执行器
    - SqlSession 在 Executor 基础上进行了更完善的封装，Executor 是在框架内部使用，MyBatis 提供以 Mapper 接口的方式来处理数据操作，事务相关操作通过 SqlSession 完成。
### (2). Mapper 映射器接口的运作方式
* 映射器接口是用户自定义的用于数据操作的 Java 接口，SqlSession 的 getMapper(Class&lt;T&gt;  type)使用映射器接口作为参数获取映射器，结合映射文件的配置调用该映射器接口中定义的方法，进行数据库操作并获得返回
* 框架对接口做一层代理，实际返回对象的类型是 org.apache.ibatis.binding.MapperProxy
* MapperProxy 代理类实现了 Java 反射接口 InvocationHandler，接口方法的最终执行都是通过 SqlSession 的数据操作来实现的
* 映射器代理（MapperProxy）通过映射器代理工厂（MapperProxyFactory）创建，映射器接口和MapperProxyFactory 之间的对应关系通过 MapperRegistry（映射器注册器）维护。映射器注册器作为Configuration  的一个成员变量，通过读取全局配置中的映射器配置初始化
### (3). SqlSession 和 Executor 如何执行 SQL 语句
* MyBatis 框架使用 Configuration 的 getMappedStatement()方法将字符串类型的 SQL 语句转换为 MappedStatement 类型的语句对象，MappedStatement（映射语句对象）表示要发往数据库执行的指令，可理解为 SQL 的抽象表示
* Executor 通过 MappedStatement 类型的参数执行实际的数据操作
* 执行器在执行数据操作时，会创建语句处理器。分为两类：
    - ParameterHandler：参数处理器，对 SQL 语句中的对象等类型参数进行处理，组装成最终执行的 SQL 语句
    - ResultSetHandler：结果集处理器，处理语句执行后产生的结果集，转换成配置的 Java 类型
    - 二者都会使用类型转换器（TypeHandler）实现数据库类型和 Java 类型的转换



## 5. 动态 SQL
* 支持使用逻辑语言定义 SQL 语句，这个表达式语言就是对象图导航语言（OGNL）
### (1). &lt;if&gt; 和 &lt;where&gt;
```xml
<select id="..." resultType="...">
	select * from User
	<where>
		<if test="name != null">
			name like '%${name}'
		</if>
	</where>
</select>
```
### (2). 多分支选择标签 &lt;choose&gt;、&lt;when&gt; 和 &lt;otherwise&gt;，相当于 switch
```xml
<select id="..." resultType="...">
	select * from User
	<where>
		<choose>
			<when test="name != null">
				AND name like '%${name}'
			</when>
			<when>
				...
			</when>
			<otherwise>
				AND id is not null
			</otherwise>
		</choose>
	</where>
</select>
```
### (3). 循环标签 &lt;foreach&gt;
* 根据 Java 集合类型对象循环产生 SQL 子句，collection 指令集合属性的名字，item 对应每次循环的值，open、close 分别是子句开始和结束附加的内容，separator 用来分隔每一项。
```xml
<select id="..." resultType="...">
	select * from User
	对 useridList 循环
	<foreach collection="useridList" item="user_id" open="where id in (" close=")" separator=",">
		#{user_id}
	</foreach>
</select>
```
```java
List<String> useridList = new ArrayList<String>()
useridList.add("1");
useridList.add("2");
//使用列表参数调用接口方法
list = mapper.findUserListWithForEach(useridList);
//最终组成 SQL 语句
select * from User where id in ('1','2');
```



## 6. MyBatis 日志
* 开启 MyBatis 调试（Debug）或追踪（Trace）级别的日志输出，可以打印转换的 SQL 语句及缓存的使用情况等。MyBatis 支持 SLF4J 和 Appache Commons Logging 标准的日志门面（标准接口，对其他日志实现库的封装），也支持直接使用 Log4j2 和Log4j 的日志实现




## 7. MyBatis 缓存
* MyBatis 将首次从数据库查询出来的数据写入内存中，后面再查询该数据则先从缓存中读取，不需要查询数据库。缓存可以加快数据查询的速度，但需要占据一定的内存，属于**空间换时间**。缓存内容以键值对的数据格式暂存，最简单的是 HashMap.
* MyBatis 提供了缓存键类（CacheKey）、缓存接口（Cache）等实现类，常用 PerpetualCache（永久缓存）
* MyBatis 支持一级和二级缓存。一级也称**本地缓存**，应用在 sqlSession 上。二级是**全局缓存**，维护在全局的 Configuration 对象中。一、二级都默认使用 MyBatis 自身缓存类实现，二级也支持第三方的缓存框架实现，如 redis。
### (1). 一级缓存
* SqlSession 对象创建时会创建关联的一级缓存，该 SqlSession 执行的查询语句及结果会被保存到一级缓存中。使用 SqlSession 的 clearCache() 可以清除。
#### 底层原理
* SqlSession 对执行器 Executor 进行了封装，一级缓存通过 BaseExecutor 成员变量 localCache 实现（变量类型为 PerpetualCache）。BaseExecutor 还有一个同类型的属性 localOutputParameterCache，用于缓存 CallableStatement 类型语句执行的存储过程的输出参数
* 底层使用 HashMap 存储缓存数据，使用 [namespace: sql: 参数] 作为 key，查询结果作为 value 保存
#### 一级缓存清除
* 使用该 SqlSession 执行增加、删除或修改并且提交事务的状况下，MyBatis 会清空该 SqlSession 的缓存数据。目的是保证缓存中的数据和数据库中的同步，避免脏读
* SqlSession 关闭时，缓存清空
* 调用 SqlSession 的 clearCache() 手动清除缓存
#### 作用机制
![MyBatis一级缓存作用机制](https://cdn.jsdelivr.net/gh/qtxsnwwb/image-hosting@master/20210823/MyBatis一级缓存作用机制.22mb72rwrv34.png)
> 注：若参数不同或调用方法不同，则会倒数据库中执行查询
> 不同的 SqlSession 即便方法和参数都相同，还是会到数据库中执行查询任务

#### 一级缓存范围
* SESSION 和 STATEMENT。默认是 SESSION，若不想使用一级缓存，可把范围指定为 STATEMENT，则每次执行完一个 Mapper 中的语句后都会将一级缓存清楚。通过属性 localCacheScope 进行修改设置。

### (2). 二级缓存
#### 缓存机制
* 二级缓存以映射配置文件的 namespace 为单位，多个 Session 调用同一个 Mapper 的查询方法查询时，会先到二级缓存查找该 Mapper 是否有对应的缓存，若有则返回。和一级一样，select() 结果会被缓存，执行 insert、update 和 delete 则会清除缓存

#### 底层原理
* 若开启了二级缓存，SqlSession 对象创建 Executor 对象时，会给执行器加上一个装饰对象 CachingExecutor。CachingExecutor 执行查询时会先查找二级缓存是否有需要的数据，若有就返回，没有则再将任务交给 Executor 对象。二级缓存通过 TransactionCacheManager 进行管理，二级缓存接口实现类为 TransactionCache，其 Map&lt;Object, Object&gt;类型的成员属性 entriesToAddOnCommit 保存缓存的查询语句和结果

#### 缓存补充
* 一级缓存是 SqlSession 级别，默认开启。二级缓存是 Mapper 级别，通过配置开启。查询数据的顺序为 二级缓存 ——> 一级缓存 ——> 数据库
* 在实现机制上，一个查询请求首先由 CachingExecutor 接收，进行二级缓存的查询，如果没命中就交给真正的 Executor 查询，执行器会到一级缓存中查询，若没命中，再到数据库查询，然后把查询结果返回 CachingExecutor 进行二级缓存，最后返回数据。
    - MyBatis 如何判断两次查询是相同的查询？
    - 判断的条件包括：
        - 查询语句的 ID
        - 组成查询的 SQL 语句字符串
        - java.sql.Statement 设置的参数
        - 查询的结果范围
    - MyBatis 内部使用缓存的键类是 CacheKey，包含一个哈希码和一个列表类型的 updateList，里面包含了 select 语句的 ID 和语句、环境等信息
    - 二级缓存应用场景
        - 适合数据查询请求多且对查询结果实时性要求不高的场景，比如统计分析类的需求
    - 缓存引用
        - 引用其他映射文件中定义的缓存，在映射文件中加入 &lt;cache-ref&gt;，namespace 为需要引用的缓存所在的命名空间
```xml
<cache-ref namespace="..." />
```
