---

# First Chapter

### RESTEASY ,从学会使用到了解原理

* [https://www.cnblogs.com/langtianya/p/7624647.html](https://www.cnblogs.com/langtianya/p/7624647.html)

#### 一、Rest简介及Resteasy产生背景

##### 了解Rest是什么

* REST并非标准，而是一种开发 Web 应用的架构风格，可以将其理解为一种设计模式。

###### 基于 REST 的 Web 服务遵循一些基本的设计原则：

* 系统中的每一个对象或是资源都可以通过一个唯一的 URI 来进行寻址，URI 的结构应该简单、可预测且易于理解，比如定义目录结构式的 URI。
* 使用 HTTP 方法，建立创建、检索、更新和删除（CRUD：Create, Retrieve, Update and Delete）操作与 HTTP 方法之间的一对一映射：
* 创建资源，应该使用 POST方法, URI : xxx/book\(在服务器端新建图书信息,需提供该图书所有信息\)
* 检索某个资源，应该使用 GET 方法, URI:xxx/book/{ID} （从服务器端获得某图书信息）
* 更改资源状态或对其进行更新，应该使用 PUT 方法,URI:xxx/book/{ID}（在服务器端更新某已存在的图书信息，需提供更新的内容）
* 删除某个资源，应该使用 DELETE 方法, URI :xxx/book/{ID} （从服务器端删除某图书信息）
* URI 所访问的每个资源都可以使用不同的形式加以表示（比如 XML 或者 JSON），具体的表现形式取决于访问资源的客户端，客户端与服务提供者使用一种内容协商的机制来选择合适的数据格式，最小化彼此之间的数据耦合。

##### 了解JAX-RS是什么

* JSR-311（JAX-RS：JavaAPI for RESTful Web Services）旨在定义一个统一的规范，使得 Java 程序员可以使用一套固定的接口来开发 REST 应用，避免了依赖于第三方框架。
* JAX-RS是一套用java实现REST服务的规范，提供了一些标注将一个资源类，一个POJOJava类，封装为Web资源. 这些标注包括以下：
* @Path：标注资源类或方法的相对路径。
* @GET，@PUT，@POST，@DELETE：标注方法是用的HTTP请求的类型
* @Produces：标注返回的MIME媒体类型。
* @Consumes：标注可接受请求的MIME媒体类型。
* @PathParam，@QueryParam，@HeaderParam，@CookieParam，@MatrixParam，@FormParam：分别标注方法的参数来自于HTTP请求的不同位置，例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自于HTTP请求的Cookie。

##### Resteasy简介

* RESTEasy是JBoss的一个开源项目，提供一套完整的框架帮助开发人员构建RESTful Web Service和RESTful Java应用程序。通过Http协议对外提供基于Java API的 RestFul Web Service。
* 作为JAX-RS的标准实现,RestEasy还具有以下亮点特性：
* 1）不需要配置文件，只要把JARs文件放到类路径里面，添加 @Path等标注就可以了
* 3）HTTP 请求由Seam来提供，不需要一个额外的Servlet

#### 手把手教你使用Resteasy

#### 揭秘Resteasy的实现原理

* ![](http://www.primeton.com/uploads/image/20160919/20160919112901_58002.png)

* 本人研究Resteasy实现原理的方法是：通过上面这个Demo来调试阅读Resteasy的源码进而理解其实现原理。

* 要发布restful的service要解决以下几个问题：
* 1\) 谁来接受来自客户端的请求，并进行分发交给对应的对象的方法去处理。
* 2\) 负责处理客户端请求的对象由谁来负责产生（上面Demo中的TestRest对象）
* 3\) 如何解析Java类上面的JAX-RS注解，使客户端过来的请求可以找到对应的对象的方法去执行。
*
* ![](http://www.primeton.com/uploads/image/20160919/20160919112939_62256.png)
* Demo中要把TestRest发布成Rest服务首先在web.xml文件中做了以下配置：
* ResteasyBootstrap作为监听器是拉起Resteasy服务的入口，在服务启动时主要做了以下动作：

* 1）通过ListenerBootstrap组件读取在web.xml文件中的一些系统配置信息，创建ResteasyDeployment对象，并将这些配置信息初始化到该对象中，其中就包括将”resteasy.resources”中配置的资源类的路径初始化到其成员变量resourceClasses中;
* 2）通过调用ResteasyDeployment的start\(\)方法，并根据相关配置信息初始化Resteasy的核心组件ResteasyProviderFactory ，Dispatcher，Registry.
* 3）最关键的部分是调用registration\(\)，在该方法中会遍历之前在web.xml中配置的资源并将其注册到Registry中, 以Demo中的例子来看会遍历resourceClasses中配置好的TestRest资源路径,并加载该类然后通过调用registry.addPerRequestResource\(clazz\)注册到Registry中; 详见以下代码片段：
![](http://www.primeton.com/uploads/image/20160919/20160919112954_90937.png)

* 在addPerRequestResource\(\)中做了两个主要的事情:其中一个是会使用相应的ResourceFactory来包装资源类TestRest,见以下代码片段：![](http://www.primeton.com/uploads/image/20160919/20160919113016_56262.png)

* 阅读POJOResourceFactory的源码可以了解到其作用就是包含了资源类的所有元信息，因此它可以利用ResteasyProviderFactory提供的注入器在需要时通过createResource\(\)来创建资源类TestRest的对象；
* 第二个主要的事情是Registry可以通过资源类中的元信息来解析上面的JAX-RS注解,并将该注解的路径和对应的方法生成的invoker对象注册到Registry中,在Demo中就是把”/path1/subpath/{id}”和 test\(\)方法的invoker对象注册到Registry中。
![](http://www.primeton.com/uploads/image/20160919/20160919113024_40804.png)

* 在web.xml文件中另一个配置是配置了HttpServletDispatcher,该类是HttpServlet的实现是所有请求的入口,通过其service\(\)方法最终将请求交给之前启动服务时已经初始化好的Dispatcher对象来处理. 以Demo为例,当请求”[http://localhost:8080/resteasydemo/path1/subpath/123”过来时,Dispatcher对象会调用其成员变量Registry对象来解析该请求中的路径”/path1/subpath/123”](http://localhost:8080/resteasydemo/path1/subpath/123”过来时,Dispatcher对象会调用其成员变量Registry对象来解析该请求中的路径”/path1/subpath/123”), 然后匹配到相应的invoker来执行客户端请求\(详见以下代码段\)，并将结果返回,页面会显示”Hello 123”;

* ![](http://www.primeton.com/uploads/image/20160919/20160919113042_25328.png)
* ![](http://www.primeton.com/uploads/image/20160919/20160919113044_37840.png)

#### 总结

* 谁来接受来自客户端的请求，并进行分发交给对应的对象的方法去处理。
* HttpServletDispatcher,（接受并分发客户端http请求）
* 负责处理客户端请求的对象由谁来负责产生。
* ResourceFactory （在服务器启动时通过web.xml读取class的配置信息然后通过反射机制产生）
* 如何解析Java类上面的注解，使客户端过来的请求可以找到对应的方法去执行。
* Registry（服务器启动时加载用户自定义Rest资源时，会解析上面的注解，并将注解相对路径和该类中执行的方法建立对应关系注册到Registry中，当客户端请求过来时会根据请求中的相对路径去Registry中查找对应的invoker对象，然后执行并将处理结果返回）
* Resteasy就是通过以上几个核心组件的相互配合，最终将一个JavaBean发布成Rest服务，这种基于服务注册的实现方式，使得Resteasy具有较好的可扩展性，例如它能很好的和Spring进行整合将SpringBean发布成Rest服务，它是如何做到的呢？首先扩展了Resteasy的ResourceFactory实现了一个SpringResourceFactory（用来从Spring容器中获得对象），然后在服务启动时当Spring容器初始化好以后，通过扩展Spring的BeanFactoryPostProcessor，将Spring容器中初始化好的SpringBean以及对应的SpringResourceFactory注册到Resteasy的Registry中.这样客户端请求过来后，当请求路径在Registry中匹配到相应的SpringBean时就可以调用该SpringBean的ResourceFactory的createResource方法，该方法可以从Spring容器中获得对象来处理请求。

##### Resteasy发布Rest服务的两种方式：

* 一种是通过listener \(ResteasyBootstrap\)方式在server启动时通过该listener的contextInitialized\(\)初始化Resteasy核心组件及Rest资源。
* 第二种是如果没有在web.xml中配置ResteasyBootstrap监听器，则在HttpServletDispatcher，第一次请求过来时通过servlet的init方法初始化Resteasy核心组件及Rest资源。
无论哪种方式原理都是一样的，只是初始化的时机不同。


