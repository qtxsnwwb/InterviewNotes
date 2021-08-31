# Spring
## 1. 概述
* Spring 是轻量级的开源 Java EE 框架
* Spring 可以解决企业应用开发的复杂性
* Spring 的两个核心部分：IOC 和 AOP
    - IOC：控制反转，把创建对象过程交给 Spring 管理
    - AOP：面向切面，不修改源代码进行功能增强
* Spring 特点
    - 方便解耦，简化开发
    - AOP 编程支持
    - 方便程序测试
    - 方便和其他框架整合
    - 方便事务操作
    - 降低 API 开发难度

## 2. IOC
* 什么是 IOC
    - 控制反转，把对象创建和对象之间的调用过程交给 Spring 进行管理
    - 目的：为了**耦合度降低**
* IOC 底层原理
    - 利用 xml 解析（注解同理）、**工厂模式**和**反射**完成类对象的创建
    - 第一步：xml 配置文件，配置创建的对象
    - 第二步：由 Service 和 Dao 类，创建工厂类
    - 第三步：调用对象

```xml
 <bean id="dao" class="com.atguigu.UserDao"></bean>
```

 ```java
class UserFactory {
	public static UserDao getDao() {
		String classValue = class属性值;
		Class clazz = Class.forName(classValue);   //利用反射创建对象
		return (UserDao)clazz.newInstance();
	}
}
 ```

```java
UserDao userDao = UserFactory.getDao();
```

* IOC 接口
    - IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂
    - Spring 提供 IOC 容器实现两种方式：
        * BeanFactory：IOC 容器基本实现，是 Spring 内部的使用接口，不提供开发人员进行使用（加载配置文件时不会创建对象，在获取对象（使用时）才会创建）
        * ApplicationContext：BeanFactory 接口的子接口，提供更多更强大的功能，一般由开发人员使用（加载配置文件时就会把配置对象创建）



## 3. IOC 操作 Bean 管理（xml）
* 什么是 Bean 管理
    - Spring 创建对象
    - Spring 注入属性
* Bean 管理操作两种方式
    * 基于 xml 配置文件方式实现
    * 基于注解方式实现
* Bean 管理（基于 xml 方式）
    - 基于 xml 方式创建对象
        - 在 Spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建
        - bean 标签属性
            - id 属性：唯一标识
            - class 属性：类全路径（包类路径）
        - 创建对象时，默认也是执行无参数构造方法完成对象创建
    - 基于 xml 方式注入属性
        - DI：依赖注入，就是注入属性，借助 property 标签原理为运行期由 Spring 根据配置文件，将其他对象的引用通过组件的提供的 setter 方法进行设定
        - 有参数构造注入属性：借助 constructor-arg 标签，通过反射机制调用有参构造方法实现
    - xml 注入其他类型属性
        - 字面量
        - 注入外部 bean
        - 注入内部 bean 和级联赋值
    - xml 注入集合属性
        - 数组 ——> <array>
        - 列表 ——> <list>
        - Map ——> <map>
        - Set ——> <set>
* 工厂 Bean
    - Spring 有两种类型的 bean
        - 普通 bean：在配置文件中定义 bean 类型就是返回类型
        - 工厂 bean：在配置文件中定义 bean 类型可以和返回类型不一样
    - 第一步：创建类，让这个类作为工厂 bean，实现接口 FactoryBean
    - 第二步：实现接口里的方法，在方法中定义返回的 bean 类型
* bean 作用域
    - bean 默认为单实例对象（singleton)，但可通过 scope 属性设置为 prototypt 改为多实例
    - singleton 和 prototype 区别：
    - 单实例对象在加载 Spring 配置文件时就会创建，多实例对象是在调用 getBean 方法时创建
* bean 生命周期
    - 通过构造器创建 bean 实例（无参数构造）
    - 为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）
    - 调用 bean 的初始化方法（需配置）
    - bean 使用（对象获取到了）
    - 当容器关闭时，调用 bean 的销毁方法（需配置）
 * xml 自动装配
     - 根据指定装配规则（属性名称或属性类型），Spring 自动将匹配的属性值进行注入
     - bean 标签属性 autowire，配置自动装配
     - autowire 属性常用两个值：
         - byName：根据属性名注入，注入值 bean 的 id 值和类属性名称一样
         - byType：根据属性类型注入



## 4. IOC 操作 Bean 管理（注解）
* 创建对象注解
    - @Component
    - @Service
    - @Controller
    - @Repository
    - 四个注解功能一样，都可以创建 bean 实例
    - 引入依赖   spring-aop.jar
    - 开启组件扫描。扫描打了注解类所在包
    - 创建类，打上注解
* 属性注入注解
    - @Autowired：根据属性类型进行自动装配
    - @Qualifier：根据名称进行注入，和 @Autowired 一起使用
    - 情景：接口有多个实现类，只用 @Autowired 则不知装配的是哪一个实现类，@Qualifier 可根据名称指定
    - @Resource：可以根据类型注入，也可根据名称注入
    - @Value：注入普通类型属性



## 5. AOP
* 面向切面编程，利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使业务逻辑各部分之间耦合度降低，提高程序的可重用性
* 即不通过修改源代码方式，在主干功能里添加新功能
* 底层原理
    - AOP 底层使用**动态代理**，动态代理分为两种情况
        - 有接口情况，使用 JDK 动态代理
            - 创建接口实现类代理对象，增强类的方法
        - 没有接口情况，使用 CGLIB 动态代理
            - 创建子类的代理对象，增强类的方法
* AspectJ
    - Spring 框架一般基于 AspectJ 实现 AOP 操作
    - AspectJ 不是 Spring 组成部分，独立 AOP 框架
    - 基于 AspectJ 实现 AOP 操作：
        - 基于 xml 配置文件实现
        - 基于注解方式使用
* 切入点表达式
    - 作用：知道对哪个类里面的哪个方法进行增强
    - 语法结构：
    - execution([权限修饰符][返回类型][类全路径][方法名称]([参数列表]))
    - 例：
    - execution(*com.atguigu.dao.BookDao.add(..))
        - 对 com.atguigu.dao.BookDao 里的 add 增强
    - execution(*com.atguigu.dao.BookDao.*(..))
        - 对 com.atguigu.dao.BookDao 里所有方法增强
    - execution(*com.atguigu.dao.*.*(..))
        - 对 com.atguigu.dao 包里所有类及类所有方法增强
* AspectJ 注解
    - 创建类，在类里定义方法
    - 创建增强类（编写增强逻辑）
    - 进行通知配置
        - 在 Spring 配置文件中开启注解扫描
        - 使用注解创建类和代理类对象
        - 在增强类上面添加注解 @Aspect
        - 在 Spring 配置文件中开启生成代理对象
    - 相同切入点抽取：@Pointcut
    - 有多个增强类对同一个方法增强，设置增强类优先级：在增强类上面添加 @Order（数字类型值），值越小，优先级越高



## 6. JdbcTemplate
* Spring 对 JDBC 进行封装，使用 JdbcTemplate 方便实现对数据库操作。可使用模板里的方法完成增删改查功能



## 7. Spring 事务管理
* 事务添加到 Java EE 三层结构里的 Service 层（业务逻辑层）
* Spring 事务管理操作有两种方式：
    - 编程式事务管理
    - 声明式事务管理（使用）
* 声明式事务管理
    - 基于注解方式（使用）
    - 基于 xml 配置文件方式
* 在 Spring 进行声明式事务管理，底层使用 AOP 原理
* Spring 事务管理 API
    - 提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类
* 参数配置
    - 在 service 类上面添加注解 @Transactional，在这个注解里可以配置事务相关参数
    - propagation：事务传播行为
        - 多事务方法直接进行调用，这个过程中事务是如何管理的
    - isolation：事务隔离级别
        - 事务有特性称为隔离性，多事务之间不会产生影响，不考虑隔离性产生三个读问题：脏读、不可重复读、虚（幻）读
    - timeout：超时时间
        - 事务需要在一定时间内进行提交，如何不提交进行回滚。默认值 -1，以秒为单位
    - readOnly：是否只读
        - 默认 false，可查、可增删改，为 true 时只可查
    - rollbackFor：回滚
        - 设置出现哪些异常进行事务回滚
    - noRollbackFor：不回滚
        - 设置出现哪些异常不进行事务回滚