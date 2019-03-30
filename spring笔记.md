**spring解决的问题**
解决了业务逻辑层和其他层的松耦合问题，将面向接口的编程思想贯彻整个系统应用

**springBean的生命周期**

* 首先看一下bean的作用域scope

  Singleton:单例，整个应用只创建一个bean的实例

  Propotype：每次注入、通过时spring上下文获取的时候都会创建一个bean的实例

  Session：每个回话创建一个bean实例

  Request：每个请求创建一个bean实例

* 他们是什么时候创建的:
  一个单例的bean,而且lazy-init属性为false(默认),在Application Context创建的时候构造
  一个单例的bean,lazy-init属性设置为true,那么,它在第一次需要的时候被构造.
  3 其他scope的bean,都是在第一次需要使用的时候创建

* 什么时候销毁

  单例bean：始终存在于application contex中，只有当application终结的时候，才会被销毁

  Prototype：和其他scope相比,Spring并没有管理prototype实例完整的生命周期,在实例化,配置,组装对象交给应用后,spring不再管理.只要bean本身不持有对另一个资源（如数据库连接或会话对象）的引用，只要删除了对该对象的所有引用或对象超出范围，就会立即收集垃圾.
   Request: 每次客户端请求都会创建一个新的bean实例,一旦这个请求结束,实例就会离开scope,被垃圾回收.

  Session：用户结束回话。bean实例会被GC

  

 



Spring模块
spring aop：
通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring框架中。所以，可以很容易地使 Spring框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中

spring orm：
Spring框架插入了若干个ORM框架，从而提供了ORM对象的关系工具，其中包括了Hibernate、JDO和 IBatis SQL Map等，所有这些都遵从Spring的通用事物和DAO异常层次结构。

spring web：
Web上下文模块建立在应用程序上下文模块之上，为基于web的应用程序提供了上下文。所以Spring框架支持与Struts集成，web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

sping webMvc
MVC框架是一个全功能的构建Web应用程序的MVC实现。通过策略接口，MVC框架变成为高度可配置的。MVC容纳了大量视图技术，其中包括JSP、POI等，模型来有JavaBean来构成，存放于m当中，而视图是一个街口，负责实现模型，控制器表示逻辑代码，由c的事情。Spring框架的功能可以用在任何J2EE服务器当中，大多数功能也适用于不受管理的环境。Spring的核心要点就是支持不绑定到特定J2EE服务的可重用业务和数据的访问的对象，毫无疑问这样的对象可以在不同的J2EE环境，独立应用程序和测试环境之间重用。


spring dao：
JDBC、DAO的抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理，和不同数据库供应商所抛出的错误信息。异常层次结构简化了错误处理，并且极大的降低了需要编写的代码数量，比如打开和关闭链接

spring context：
Spring上下文是一个配置文件，向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI、EJB、电子邮件、国际化、校验和调度功能。

Spring core：
核心容器提供Spring框架的基本功能。Spring以bean的方式组织和管理Java应用中的各个组件及其关系。Spring使用BeanFactory来产生和管理Bean，它是工厂模式的实现。BeanFactory使用控制反转(IoC)模式将应用的配置和依赖性规范与实际的应用程序代码分开。







spring事物传播模式：
1、required：业务方法运行时，如果有事物加入这个事物,否者新建事物
2、not_support：声明方法不需要事物，容器默认不开启事物，如果方法在一个事物中被调用，该事物挂起，调用结束，原事物恢复
3、requirenew：不管是否存在事物，都会发起一个新事物，如果方法已经运行在事物中，则原事物挂起创建新事物。
4、mandatory(强制的):只能在一个已存在的事物中执行，不能发起自己的事物，如果在没有事物的环境下别调用，抛异常
5、support：在事物范围内调用，成为该事物的一部分。在事物外被调用，则在没有事物的环境下被调用。
6、never：不能再事物中被调用，否者报错
7、nested：如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。

事务嵌套的问题应该从事务传播方式考虑。



事物的隔离级别
spring(源码中默认是-1，完全取决于数据库)本身没有事物，所有的事物操作是基于数据库的
没有隔离级别可能发生这么几种情况：
脏读：一个事物读取到另一个事物未提交的数据
不可重复读：一个事物内多次查询返回不同的数据结果(不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据)
幻读：一个事物读取到别的事物插入的数据，导致前后不一致

解决上面的情况，使用不同的隔离级别。

1、Serializable ：最严格的级别，事务串行执行，资源消耗最大；
2、REPEATABLE READ ：保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。避免了“脏读取”和“不可重复读取”的情况，但是带来了更多的性能损失。
3、READ COMMITTED :大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读取”。该级别适用于大多数系统。
4、Read Uncommitted ：保证了读取过程中不会读取到非法数据。


@Transactional失效
1、如果方法不是public会失效、private、static、final都失效
2、如果异常是check默认也不会滚，如果想check异常回归，注解上面写明异常类型（Spring事务管理只对出现运行期异常进行回滚）
3、数据库引擎要支持事物，如mysql 注意如果是myisam事物是不起作用的
4、是否开启了对注解的解析
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
5、spring 是否扫描到这个包



场景：如果某一个没有事务的方法调用了存在事务注解的方法，那么事务会起作用吗？

不会。原因spring 事务是通过sping aop的方式，也就是通过代理类队目标类进行增强，没有加注解，代理类中没有对该方法增强，所有事务不起作用。

解决方法：

1、在当前类的方法中通过AopContext.currentProxy()获取当前类的代理对象，再调用的代理对象的方法(其实是增强后的方法)，事务就会生效了。

2、可以把自己的实例注入进来，内部方法调用改为直接调用注入的实例。因为Spring其实是为你创建了一个代理，那我们调用代理就好了。其效果是一样的。








核心机制分析

设计模式：
   工厂模式：private static final Protocol protocol =
			ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
			Dubbo 里有很多这种代码。这也是一种工厂模式，只是实现类的获取采用了 jdkspi 的机
			制。这么实现的优点是可扩展性强，想要扩展实现，只需要在 classpath 下增加个文件就可
			以了，代码零侵入。另外，像上面的 Adaptive 实现，可以做到调用时动态决定调用哪个实
			现，但是由于这种实现采用了动态代理，会造成代码调试比较麻烦，需要分析出实际调用的
			实现类。
   观察者模式：cusumer在订阅服务时，实现notifyListener，开启listener注册中心会每 5 秒定时检查是否有服
				务更新，如果有更新，向该服务的提供者发送一个 notify 消息， provider 接受到 notify 消息
				后，即运行 NotifyListener 的 notify 方法，执行监听器方法。
   动态代理模式：
			   Dubbo 扩展 jdkspi 的类 ExtensionLoader 的 Adaptive 实现是典型的动态代理实现。 Dubbo
			需要灵活地控制实现类，即在调用阶段动态地根据参数决定调用哪个实现类，所以采用先生
			成代理类的方法，能够做到灵活的调用。生成代理类的代码是 ExtensionLoader 的
			createAdaptiveExtensionClassCode 方法。代理类的主要逻辑是，获取 URL 参数中指定参数的
			值作为获取实现类的 key

bean加载
扩展点机制
动态代理
远程调用

