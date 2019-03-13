#### tomcat运行

* Tomcat是运行在JVM中的一个进程。它定义为【中间件】，[顾名思义](https://www.baidu.com/s?wd=%E9%A1%BE%E5%90%8D%E6%80%9D%E4%B9%89&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，是一个在Java项目与JVM之间的中间容器。
* Web项目的本质，是一大堆的资源文件和方法。Web项目没有入口方法(main方法)，，意味着Web项目中的方法不会自动运行起来。
* Web项目部署进Tomcat的webapp中的目的是很明确的，那就是希望Tomcat去调用
  写好的方法去为客户端返回需要的资源和数据。
*  Tomcat可以运行起来，并调用写好的方法。那么，Tomcat一定有一个main方法。
* 对于Tomcat而言，它并不知道我们会有什么样的方法，这些都只是在项目被部署进webapp下后才确定的，由此分析，必然用到了Java的反射来实现类的动态加载、实例化、获取方法、调用方法。但是我们部署到Tomcat的中的Web项目必须是按照规定好的接口来进行编写，以便进行调用

https://blog.csdn.net/qq_32951553/article/details/79686367



####Tomcat原理总结

1. Tomcat需要main方法启动。
2. Tomcat需要监听本机上的某个端口。
3. Tomcat需要抓取此端口上来自客户端的链接并获得请求调用的方法与参数。
4. Tomcat需要根据请求调用的方法，动态地加载方法所在的类，完成累的实例化并通过该实例获得需要的方法最终将请求传入方法执行。
5. 将结果返回给客户端（jsp/html页面、json/xml字符串）
6. 
#### Tcomcat总体架构

![å¾ 1.Tomcat çæ"ä½ç"æ](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/image001.gif)



​        多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了。所以整个 Tomcat 的生命周期由 Server 控制。

​       Server通过一个数组管理者service，Service的生命周期通过Lifecycle 接口控制，service是由多个connector、一个container组成。

​      connector：负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。

https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/



#### servlet

- 当第一次调用某个servlet容器时，需要在入其servlet类，并调用其init方法
- 针对每一个request请求，创建一个javax.servlet.ServletRequest、javax.servlet.ServletResponse实例
- 调用servlet的service方法，将servletRequest、ServletResponse作为参数载入
- 当关闭servlet类时，调用其destory方法，并卸载servlet类

 

tomcat将错误信息存储在一个properties文件中，便于读取和编辑。tomcat将properties文件划分到不同的包中，每个文件都是通过StringManager类来处理的，StringManager类被包内所有的对象共享，所以StringManager是单例类



**连接器Connector**

HttpConnetcors实例维护了一个HttpProcessor的实例池，避免重复创建，