#### Tcomcat总体架构

![å¾ 1.Tomcat çæ"ä½ç"æ](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/image001.gif)



​        多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了。所以整个 Tomcat 的生命周期由 Server 控制。

​       Server通过一个数组管理者service，Service的生命周期通过Lifecycle 接口控制，service是由多个connector、一个container组成。

​      connector：负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。

https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/



