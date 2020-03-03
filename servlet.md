## 什么是Servlet

Servlet 是基于 Java 技术的 web 组件，容器托管的，用于生成动态内容。像其他基于 Java 的组件技术一样，
Servlet 也是基于平台无关的 Java 类格式，被编译为平台无关的字节码，可以被基于 Java 技术的 web server
动态加载并运行。容器，有时候也叫做 servlet 引擎，是 web server 为支持 servlet 功能扩展的部分。客户端
通过 Servlet 容器实现的请求/应答模型与 Servlet 交互。

## 什么是Servlet容器

Servlet 容器是 web server 或 application server 的一部分，提供基于请求/响应发送模型的网络服务，解码基
于 MIME 的请求，并且格式化基于 MIME 的响应。Servlet 容器也包含了管理 Servlet 生命周期。
Servlet 容器可以嵌入到宿主的 web server 中，或者通过 Web Server 的本地扩展 API 单独作为附加组件安装。
Servelt 容器也可能内嵌或安装到包含 web 功能的 application server 中。

> 也就是说一个java web服务器必须要支持http的解析，还要实现对于servlet的加载和管理。

## Servlet接口

GenericServlet 和 HttpServlet，开发者只需要继承 HttpServlet 去实现自己的 Servlet 即可。

###实例数量

servlet 容器对于每一个 Servlet 声明必须且只能产生一个实例。不过，如果 Servlet 实现了 SingleThreadModel 接口，servlet 容器可以选择实例化多个实例以便处理高负荷请求或者串行化请求到一个特定实例。

### Single Thread Model

SingleThreadModel 接口的作用是保证一个特定servlet 实例的service 方法在一个时刻仅能被一个线程执行，
一定要注意，此保证仅适用于每一个 servlet 实例，因此容器可以选择池化这些对象。有些对象可以在同一
时刻被多个 servlet 实例访问，因此容器可以选择池化这些对象。

### Servlet生命周期

#### 加载和实例化

Servlet 容器负责加载和实例化 Servlet。加载和实例化可以发生在容器启动时，或者延迟初始化直到容器决
定有请求需要处理时。当 Servlet 引擎启动后，servlet 容器必须定位所需要的 Servlet 类。

> 准确应该是容器加载Servlet的时机

#### 初始化

一旦一个 Servlet 对象实例化完毕，容器接下来必须在处理客户端请求之前初始化该 Servlet 实例。初始化
的目的是以便 Servlet 能读取持久化配置数据，初始化一些代价高的资源（比如 JDBC API 连接），或者执
行一些一次性的动作。

#### 请求处理

Servlet 完成初始化后，Servlet 容器就可以使用它处理客户端请求了。客户端请求由 ServletRequest 类型的
request 对象表示。Servlet 封装响应并返回给请求的客户端，该响应由 ServletResponse 类型的 response 对象
表示。这两个对象（request 和 response）是由容器通过参数传递到 Servlet 接口的 service 方法的。

**由 HttpServlet 中实现的 service 方法转发到相应的协议相关的处理方法上doGet，doPost**

**异步处理**

> 这里spring中有inteceptor和filter方式，应该使用filter方式。因为在 Servlet 中等待是一个低效的操作，因为这是阻塞操作，从而白白占用一个线程或其他一些受限资源。

####终止服务

Servlet 容器没必要保持装载的 Servlet 持续任何特定的一段时间。一个 Servlet 实例可能会在 servlet 容器内
保持活跃（active）持续一段时间（以毫秒为单位），Servlet 容器的寿命可能是几天，几个月，或几年，或
者是任何之间的时间。

### 请求

HTTP 方法是 POST和内容类型是 application/x-www-form-urlencoded。servlet 必须可以通过 request 对象的输入流得到 POST 数据。

> 有的前端不懂http协议。

#### 请求路径元素

- Context Path：与 ServletContext 相关联的路径前缀是这个 servlet 的一部分。如果这个上下文是基于 Web
  服务器的 URL 命名空间基础上的“默认”上下文，那么这个路径将是一个空字符串。否则，如果上下文不是
  基于服务器的命名空间，那么这个路径以/字符开始，但不以/字符结束。
- Servlet Path：路径部分直接与激活请求的映射对应。这个路径以“/”字符开头，如果请求与“/ *”或“”模式
  匹配，在这种情况下，它是一个空字符串。
- PathInfo：请求路径的一部分，不属于 Context Path 或 Servlet Path。如果没有额外的路径，它要么是 null，要么是以'/'开头的字符串。

requestURI = contextPath + servletPath + pathInfo 

#### Request对象的生命周期

**每个 request 对象只在 servlet 的 service 方法的作用域内，或过滤器的 doFilter 方法的作用域内有效，除非该组件启用了异步处理并且调用了 request 对象的 startAsync 方法。**

## ServletContext

ServletContext（Servlet 上下文）接口定义了 servlet 运行在的 Web 应用的视图。容器供应商负责提供 Servlet
容器的 ServletContext 接口的实现。Servlet 可以使用 ServletContext 对象记录事件，获取 URL 引用的资源，
存取当前上下文的其他 Servlet 可以访问的属性。

**ServletContainerInitializer**这个类会在servlet容器启动阶段被回调，可以在onStartup方法里做一些servlet、filter、listener的注册等操作

## Response

和Request差不多

## 过滤器

过滤器是一种代码重用的技术，它可以改变 HTTP 请求的内容，响应，及 header 信息。过滤器通常不产生
响应或像 servlet 那样对请求作出响应，而是修改或调整到资源的请求，修改或调整来自资源的响应。

### 生命周期

在 Web 应用部署之后，在请求导致容器访问 Web 资源之前，容器必须找到过滤器列表并按照如上所述的应
用到 Web 资源。容器必须确保它为过滤器列表中的每一个都实例化了一个适当类的过滤器,并调用其
init(FilterConfig config)方法。过滤器可能会抛出一个异常,以表明它不能正常运转。如果异常的类型是
UnavailableException，容器可以检查异常的 isPermanent 属性并可以选择稍候重试过滤器。

当容器接收到传入的请求时，它将获取列表中的第一个过滤器并调用 doFilter 方法，传入 ServletRequest 和
ServletResponse，和一个它将使用的 FilterChain 对象的引用。

## 注解与热拔插

web 应用中，使用注解的类仅当它们位于 WEB-INF/classes 目录中，或它们被打包到位于应用的
WEB-INF/lib 中的 jar 文件中时它们的注解才将被处理。

### 热拔插

**web fragment 是 web.xml 的部分或全部，可以在一个类库或框架jar 包的 META-INF 目录指定和包括。在 WEB-INF/lib 目录中的普通的老的 jar 文件即使没有web-fragment.xml 也可能被认为是一个 fragment，任何在它中指定的注解都将按照定义在 8.2.3 节的规则处理，容器将会取出并按照如下定义的规则进行配置。**

如果框架打包成 jar 文件，且有部署描述符的形式的元数据信息，那么 web-fragment.xml 描述符必须在该 jar
包的 META-INF/目录中。

如果框架想使用 META-INF/web-fragment.xml，以这样一种方式，它扩充了 web 应用的 web.xml，框架必须
被绑定到 Web 应用的 WEB-INF/lib 目录中。为了使框架中的任何其他类型的资源（例如，类文件）对 web
应用可用，把框架放置在 web 应用的 classloader 委托链的任意位置即可。换句话说，只有绑定到 web 应用
的 WEB-INF/lib 目录中的 JAR 文件，但不是那些在类装载委托链中更高的，需要扫描其 web-fragment.xml。