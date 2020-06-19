title: Tomcat
author: Sunkz
tags:
  - tomcat

photos:

- https://s1.ax1x.com/2020/05/23/YxSJUI.png

categories: []
date: 2020-04-02 19:38:00

---
### 顶层设计

<img src="https://s1.ax1x.com/2020/04/01/G30Si9.png" style="zoom:50%;" />

### Service中的核心组件

##### 连接器(Connector) 

- 监听网络端口

- 接收网络连接(同步/异步 阻塞/非阻塞IO)请求

  > 处理连接的时候可以做到非阻塞,但是真正处理请求执行业务时还是同步阻塞. (当然处理业务也可以利用Java非阻塞或异步).
  >
  > Node.js Netty则是在框架层面直接做到连接和处理业务整个过程的非阻塞???
  >
  > Node.js 只有一个主线程,一个事件循环,没办法利用CPU多核.Vert.x可以利用多线程,多事件循环??? Spring WebFlux..???

- 读取请求字节流

- 根据具体的应用层协议(HTTP/HTTPS/AJP)解析字节流,生成统一的Tomcat Request对象

- 将Tomcat Request对象转成标准的ServletRequest

- 调用Servlet容器,得到ServletResponse

- 将ServletResponse转换成TomcatResponse对象

- 将Tomcat Response转成网络字节流

- 将响应流写回浏览器

<img src="https://s1.ax1x.com/2020/04/01/G3XEgs.png" alt="G3XEgs.png" style="zoom:50%;" />

##### 容器(Container) 

- Tomcat容器: ServletContext 

  > ServletContext持有所有Servlet实例,Servlet 可以通过全局的ServletContext来共享数据，这些数据包括 Web应用的初始化参数、Web应用目录下的文件资源等.
  
- Spring容器 : IoC

  > Spring的ContextLoaderListener监听到Tomcat的启动事件,开始初始化Spring根容器(IoC容器),并将其存储到ServletContext中,便于以后来获取.
  
- SpringMVC容器 : DispatcherServlet

  > Tomcat启动时扫描Servlet时,若检测到DispatcherServlet未被实例化,还会调用DispatcherServlet的init方法,DispatcherServlet创建自己的SpringMVC容器,持有SpringMVC相关的Bean.同时，Spring MVC还会通过ServletContext拿到Spring根容器，并将Spring根容器设为SpringMVC容器的父容器.Spring MVC容器可以访问父容器中的Bean，但是父容器不能访问子容器的Bea.

<img src="https://s1.ax1x.com/2020/04/02/GJkfij.png" style="zoom: 45%;" />

<img src="https://s1.ax1x.com/2020/04/02/GYJFOO.png" style="zoom: 67%;" />