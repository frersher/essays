dubbo

当单一节点的处理能力无法满足日益增长的计算、存储任务的时候，且硬件的提升得不偿失，应用程序不能进一步优化的时候，考虑分布式系统。

**SPI(类似于策略模式)**

​    是jdk内置的服务提供发现机制，说白了就是寻找借口的实现类；具体点：在Jar包的“src/META-INF/services/”目录下建立一个文件，文件名是接口全限定名，内容可以多行，每一行是具体实现类的全限定名。

​      java的spi运行流程是运用java.util.ServiceLoader这个类的load方法去在src/META-INF/services/寻找对应的全路径接口名称的文件，然后在文件中找到对应的实现方法并注入实现，然后你可以运用了

​     dubbo并没有直接使用java原生的spi，而是对其进行增强，dubbo的spi逻辑封装在ExtensionLoader类中，Dubbo SPI 所需的配置文件需放置在 META-INF/dubbo 路径下，dubbo的spi是通过键值对的方式进行配置，这样可以按需加载



​	

CAP理论
对于分布式数据存储，最多只能同时满足一致性、可用性、分区容错性中的两者。
一致性：对于每一次读操作，要么都读到最新写入的数据，要么错误。
可用性：对于每一次请求，都能得到一个及时的、非错误的响应，但是不保证请求的结果是基于最新写入的数据
分区容错性：指的是节点之间的网络问题，即使一些消息对包或者延迟，整个系统能继续提供服务（提供一致性或者可用性）。

**dubbo模块**

* common:公共逻辑模块
* remoting：远程通信模块，dubbo协议实现
* rpc：远程调用模块，抽象各种协议，以及动态代理，只包含一对一调用，不关心集群管理。
* config：配置模块，对外的API，用户通过Config使用dubbo，dubbo对外隐藏细节
* registry：注册中心基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。
* contain：容器，以简单的main方式加载spring启动
* cluster：集群，将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。



**注册中心**
基于dubbo协议给出默认一个注册中心实现SimpleRegistryService，只是简单地实现不支持集群。SimpleRegitryService在被订阅的时候，同时会refer引用调用方暴露的NitifyListener服务，当注册数据变动的时候会自动推送。

生产者发布服务
1. 指定了哪种的注册中心，是基于 dubbo 协议的，指定了注册中心的地址以及
端口号

2. 发布xxx 服务，服务的实现为 xxxImpl
每个<dubbo:service/>在 spring 内部都会生成一个 ServiceBean 实例，
ServiceBean 的实例化过程中调用 export 方法来暴露服务

3. 遍历 registryUrls 向注册中心注册服务
给每个 registryUrl 添加属性 key 为 export， value 为上面的发布服务
url 得到如下 registryUrl

4、由发布的服务实例，服务接口以及 registryUrl 为参数，通过代理工厂
proxyFactory 获取 Invoker 对象，Invoker 对象是 dubbo 的核心模型，
其他对象都向它靠拢或者转换成它。

5、通过 Protocol 对象暴露服务 protocol.export(invoker)
通过 DubboProtocol 暴露服务的监听(不是此节内容)
通过 RegistryProtocol 将服务地址发布到注册中心，并订阅此服务

RegistryProtocol.export(Invoker) 暴露服务

1.调 DubboProtocol 暴露服务的监听
2.获取注册中心 getRegistry(Invoker)
3.获取发布 url 就是 registryUrl 的 export 参数的值
4.DubboRegistry.register(registryProviderUrl)
通过注册器向注册中心注册服务
5、构建订阅服务 overrideProviderUrl
6、构建 OverrideListener 它实现与 NotifyLisener, 当注册中心的订阅
的 url 发生变化时回调重新 export


消费者引用服务
1. 指定了哪种的注册中心，是基于 dubbo 协议的，指定了注册中心的地址以及
端口号
2. 引用远程 xxx 服务
每个<dubbo:reference/>标签spring 加载的时候都会生成一个Referenc
eBean。
3.遍历 registryUrls 集合，使用 Protocol.refer(interface, regist
ryUrl)的到可执行对象 invoker
4.如果注册中心有多个的话， 通过集群策略 Cluser.join()将多个 invoke
r 伪装成一个可执行 invoker， 这里默认使用 available 策略
5.利用代理工厂生成代理对象 proxyFactory.getProxy(invoker)

Protocol.refer 过程流程
1. 根据传入的 registryUrl 是用来选择 RegistryProcol 它的协议属性是
registry, 下面要选择使用哪种注册中心所以要根据 REGISTRY_KEY 属
性重新设置 registrUrl
dubbo://127.0.0.1:9098/com.alibaba.dubbo.registry.Regis
tryService?application=demo-consumer&dubbo=2.0.0&pid=45
24&refer=application%3Ddemo-consumer%26dubbo%3D2.0.0%26
interface%3Dcom.alibaba.dubbo.demo.DemoService%26method
s%3DsayHello%26pid%3D4524%26side%3Dconsumer%26timestamp
%3D1415881461048&timestamp=1415881461113
2. 根据 registrUrl 利用 RegistryFactory 获取注册器（过程跟暴露服务
那边一样）， 这里是 dubbo 协议得到的是注册器是 DubboRegistry
引用并订阅注册中心服务，
3. 构建引用服务的 subscribeUrl
consumer://10.5.24.221/com.alibaba.dubbo.demo.DemoServi
ce?application=demo-consumer&category=consumers&check=f
alse&dubbo=2.0.0&interface=com.alibaba.dubbo.demo.DemoS
ervice&methods=sayHello&pid=8536&side=consumer&timestam
p=1415945205031
并通过注册器向注册中心注册消费方， 主要这里的 category 是 consume
rs
4.构建目录服务 RegistryDirectory,
构建订阅消费者订阅 url，这里主要 category=providers 去注册中心寻
找注册的服务提供者

向注册中心订阅消费方，注册中心根据消费者传入的 url 找到匹配的服务提
供者 url (注意：这里服务提供者没有设置 category，注册中心对于没有
设置的默认取 providers 值)

然后注册中心回调服务消费者暴露的回调接口来对服务提供者的服务进行引
用 refer 生成对应的可执行对象 invoker。服务提供者与服务的消费建立
连接

5. 通过 Cluster 合并 directory 中的 invokers， 返回可执行对象
invoker

6. ProxyFactory.getProxy(invoker) 创建代理对象返回给业务方使用


Zookeeper注册中心
Zookeeper 对数据存储类似 linux 的目录结构，

dubbo的容错和负载均衡
https://www.cnblogs.com/juncaoit/p/7691411.html





#### 如何实现RPC服务的调用

客户端、服务端要完成一次RPC调用，必须先建立连接，双方需要根据某一种约定的协议进行网络通信，这就是通信协议，可以通信后，服务端接受到请求后，需要一某种方式进行处理，然后将结果序列化返回，这是一个RPC调用过程



#### 判断服务节点是否存活

Zookeeper判断节点是否存活机制就是注册中心摘除机制，服务消费者以注册中心为准，当服务节点变更时，注册中心会把变更通知给消费者，消费者会拉取最新的数据节点信息。

在网络颤抖时服务消费者会频繁收到服务变更的通知，于是不断请求注册中心拉去最新节点信息，如果是成百上千个服务消费者同时拉取数据，可能导致注册中心带宽给占满。

**需要一种保护机制**

* 心跳开关保护机制

  注册中心设置一个开关，开关打开的时候，注册中心不会通知所有服务消费者有服务节点变更，比如只给10%的消费者返回，这样注册中心的请求量就减少到1/10。但是这样会导致消费者同步最新节点信息延迟，所以这种开关可以作为一种紧急措施，在网络抖动的时候打开。

* 服务节点摘除保护机制

  服务提供者每隔一段时间汇报心跳给注册中心以表示自己存活，如果隔了固定时间之后没有汇报心跳，注册中心认为该节点已经dead，于是从服务的可用节点中移除。如果遇到网络情况，大量的节点汇报心跳失败，然后被移除，剩下的节点无法承受现有的调用，引起‘雪崩’。 对于这种情况需要根据实际的业务情况，设定一个阈值 通常是20%，即使遇到网络等特殊情况，最多可以移除20%的服务节点 



无论心跳开关还是节点摘除都是节点信息不断发生变化，所以注册中心被称为动态节点中心，其实可以换一个思路，服务消费者并不严格以注册中心服务信息为准，而是更多的以实际调用信息判断提供者节点是否可用，所以考虑一种静态的注册中心。




* 静态注册中心

  心跳机制是当服务提供者出现异常时，注册中心将异常节点移除；可以考虑这种心跳机制在服务消费者端做。服务端调用某一服务节点失败超过一定次数，可以将这个节点标记为异常节点，并每隔一段时间发起探活，如果探活成功，重新标记为可用，重新发起调用。这个时候服务提供者不需要翔注册中心汇报心跳信息 注册中心的数据不在动态变化，所以称为静态注册中心



#### 如何使用负载均衡算法

* 随机算法
* 轮询算法
* 加权轮询算法
* hash一致性算法



#### 如何使用服务路由

服务路由：服务消费者在发起调用时必须按照某种规则选择服务节点，从而满足某种特定的需求。



#### 服务的引用

Dubbo 服务引用的时机有两个

1、Spring 容器调用 ReferenceBean 的 afterPropertiesSet 方法时引用服务

2、在 ReferenceBean 对应的服务被注入到其他类中时引用。这两个引用服务的时机区别在于

第一个是饿汉式的，第二个是懒汉式的，默认情况下，Dubbo 使用懒汉式引用服务。如果需要使用饿汉式，可通过配置 <dubbo:reference> 的 init 属性开启

```
下面我们按照 Dubbo 默认配置进行分析，整个分析过程从 ReferenceBean 的 getObject 方法开始。当我们的服务被注入到其他类中时，Spring 会第一时间调用 getObject 方法，并由该方法执行服务引用逻辑。按照惯例，在进行具体工作之前，需先进行配置检查与收集工作。接着根据收集到的信息决定服务用的方式，有三种，第一种是引用本地 (JVM) 服务，第二是通过直联方式引用远程服务，第三是通过注册中心引用远程服务。不管是哪种引用方式，最后都会得到一个 Invoker 实例。如果有多个注册中心，多个服务提供者，这个时候会得到一组 Invoker 实例，此时需要通过集群管理类 Cluster 将多个 Invoker 合并成一个实例。合并后的 Invoker 实例已经具备调用本地或远程服务的能力了，但并不能将此实例暴露给用户使用，这会对用户业务代码造成侵入。此时框架还需要通过代理工厂类 (ProxyFactory) 为服务接口生成代理类，并让代理类去调用 Invoker 逻辑。避免了 Dubbo 框架代码对业务代码的侵入，同时也让框架更容易使用。
```



##### 服务目录

```
服务目录中存储了一些和服务提供者有关的信息，通过服务目录，服务消费者可获取到服务提供者的信息，比如 ip、端口、服务协议等。通过这些信息，服务消费者就可通过 Netty 等客户端进行远程调用。在一个服务集群中，服务提供者数量并不是一成不变的，如果集群中新增了一台机器，相应地在服务目录中就要新增一条服务提供者记录。或者，如果服务提供者的配置修改了，服务目录中的记录也要做相应的更新。如果这样说，服务目录和注册中心的功能不就雷同了吗？确实如此，这里这么说是为了方便大家理解。实际上服务目录在获取注册中心的服务配置信息后，会为每条配置信息生成一个 Invoker 对象，并把这个 Invoker 对象存储起来，这个 Invoker 才是服务目录最终持有的对象。Invoker 有什么用呢？看名字就知道了，这是一个具有远程调用功能的对象。讲到这大家应该知道了什么是服务目录了，它可以看做是 Invoker 集合，且这个集合中的元素会随注册中心的变化而进行动态调整。
```

